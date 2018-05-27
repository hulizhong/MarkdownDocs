## ReadMe
evmap相关概念；

## evmpa\_io\_active\_
把一组事件激活（等待在fd上的、注册到base上的）
```cpp
void evmap_io_active_(struct event_base *base, evutil_socket_t fd, short events)
{
	struct event_io_map *io = &base->io;
	struct evmap_io *ctx;
	struct event *ev;

#ifndef EVMAP_USE_HT
	if (fd < 0 || fd >= io->nentries)
		return;
#endif
	GET_IO_SLOT(ctx, io, fd, evmap_io); //(cvmap_io)ctx = io[fd]，即获得之前注册到fd上的所有事件

	if (NULL == ctx)
		return;
	LIST_FOREACH(ev, &ctx->events, ev_io_next) { //遍历ctx上的所有事件；
		if (ev->ev_events & events) //如果ev的事件符合传入参数events事件，那么激活该事件1次；
			event_active_nolock_(ev, ev->ev_events & events, 1);
	}
}
```


## evmap\_io\_add\_
将fd上的ev事件，添加至backend的等候事件中，并且添加到base.io中；
添加到backend中时，epoll有以下两种方案：
> changelist，这个效率更高
> nochangelist

而kqueue只有changelist
```cpp
//返回
	//-1，失败
	//0，成功，但后台backend忽略此次backend
	//1，成功，添加到backend成功；
int evmap_io_add_(struct event_base *base, evutil_socket_t fd, struct event *ev)
{
	const struct eventop *evsel = base->evsel;
	struct event_io_map *io = &base->io;
	struct evmap_io *ctx = NULL;
	int nread, nwrite, nclose, retval = 0;
	short res = 0, old = 0;
	struct event *old_ev;

	EVUTIL_ASSERT(fd == ev->ev_fd);

	if (fd < 0)
		return 0;

#ifndef EVMAP_USE_HT
	if (fd >= io->nentries) {
		if (evmap_make_space(io, fd, sizeof(struct evmap_io *)) == -1)
			return (-1);
	}
#endif
	/* 获取fd的事件列表，--rwhy没有会分配？？ */
	GET_IO_SLOT_AND_CTOR(ctx, io, fd, evmap_io, evmap_io_init,
						 evsel->fdinfo_len); //(evmap_io)ctx = io[fd]

	nread = ctx->nread;
	nwrite = ctx->nwrite;
	nclose = ctx->nclose;

	/* 获得fd上老的注册事件 */
	if (nread) //如果此fd之前的nread!=0，那么之前就注册过读事件
		old |= EV_READ;
	if (nwrite)
		old |= EV_WRITE;
	if (nclose)
		old |= EV_CLOSED;

	/* 获得fd上此次注册事件，并更改注册对应类型的事件数 */
	if (ev->ev_events & EV_READ) {
		if (++nread == 1)
			res |= EV_READ;
	}
	if (ev->ev_events & EV_WRITE) {
		if (++nwrite == 1)
			res |= EV_WRITE;
	}
	if (ev->ev_events & EV_CLOSED) {
		if (++nclose == 1)
			res |= EV_CLOSED;
	}
	if (EVUTIL_UNLIKELY(nread > 0xffff || nwrite > 0xffff || nclose > 0xffff)) {
		event_warnx("Too many events reading or writing on fd %d",
		    (int)fd);
		return -1;
	}
	if (EVENT_DEBUG_MODE_IS_ON() &&
	    (old_ev = LIST_FIRST(&ctx->events)) &&
	    (old_ev->ev_events&EV_ET) != (ev->ev_events&EV_ET)) {
		event_warnx("Tried to mix edge-triggered and non-edge-triggered"
		    " events on fd %d", (int)fd);
		return -1;
	}

	/* 如果ev是read/write/close事件类型，那么把ev添加到backend中去 */
	if (res) {
		void *extra = ((char*)ctx) + sizeof(struct evmap_io); //注意这是之前结构体外申请的多余空间；
		/* XXX(niels): we cannot mix edge-triggered and
		 * level-triggered, we should probably assert on
		 * this. */
		if (evsel->add(base, ev->ev_fd,
			old, (ev->ev_events & EV_ET) | res, extra) == -1)
			return (-1);
		retval = 1;
	}

	/* 把事件ev添加到base.io中去 */
	ctx->nread = (ev_uint16_t) nread;
	ctx->nwrite = (ev_uint16_t) nwrite;
	ctx->nclose = (ev_uint16_t) nclose;
	LIST_INSERT_HEAD(&ctx->events, ev, ev_io_next);

	return (retval);
}
```

## changelist knowledge
只会在backend支持changelist才会使用这个特性；
changelist的好处是：
> 1. 避免上层应用程序在dispatch之前对同一个事件进行多次的add, del。  --rwhy这个可以吗？？？   
> 2. 一个fd上多个改变通过一次系统调用就可以完成，比如epoll可以同时add和delete。  

rwhy有效防护的是？
同一个fd的读事件多次add,del？？ 
还是对同一个event的read/write更改？？？


从上次eventop.dispatch()到现在的change列表；
```cpp
struct event_changelist {
	struct event_change *changes; //更改事件列表
	int n_changes;
	int changes_size;
};
```

代表一个更改事件
```cpp
/** Represents a */
struct event_change {
	evutil_socket_t fd; //更改事件的fd/signalNo
	/* The events that were enabled on the fd before any of these changes
	   were made.  May include EV_READ or EV_WRITE. */
	short old_events;  

	ev_uint8_t read_change; //fd上读事件更改标志，信号可置ev_change_signal.
	ev_uint8_t write_change;
	ev_uint8_t close_change;
};

//上述read_change, write_change可置的标志；
#define EV_CHANGE_ADD     0x01  //增加事件
#define EV_CHANGE_DEL     0x02  //删除事件，与ev_change_add相反
#define EV_CHANGE_SIGNAL  EV_SIGNAL  //这是一个signal事件，而非fd事件
#define EV_CHANGE_PERSIST EV_PERSIST //持久事件，但现在没用到；
#define EV_CHANGE_ET      EV_ET      //ET模式
```

## event\_changelist\_add_
```cpp
int
event_changelist_add_(struct event_base *base, evutil_socket_t fd, short old, short events,
    void *p)
{
	struct event_changelist *changelist = &base->changelist;
	struct event_changelist_fdinfo *fdinfo = p;
	struct event_change *change;

	event_changelist_check(base);

	/* 有就返回，没有就创建初始化 */
	change = event_changelist_get_or_construct(changelist, fd, old, fdinfo);
	if (!change)
		return -1;

	/* An add replaces any previous delete, but doesn't result in a no-op,
	 * since the delete might fail (because the fd had been closed since
	 * the last add, for instance. */

	if (events & (EV_READ|EV_SIGNAL)) {
		change->read_change = EV_CHANGE_ADD |
		    (events & (EV_ET|EV_PERSIST|EV_SIGNAL));
	}
	if (events & EV_WRITE) {
		change->write_change = EV_CHANGE_ADD |
		    (events & (EV_ET|EV_PERSIST|EV_SIGNAL));
	}
	if (events & EV_CLOSED) {
		change->close_change = EV_CHANGE_ADD |
		    (events & (EV_ET|EV_PERSIST|EV_SIGNAL));
	}

	event_changelist_check(base);
	return (0);
}
```

