## ReadMe
evmap相关概念；

## event\_signal\_map 
用于signal no到该signal一系列event的映射；
```cpp
struct event_signal_map {
	// 一个evmap_io*或者evmap_signal*的数组；没有元素则为NULL
	void **entries;
	int nentries; //entries数组的大小，以signalNo为下标；
};

struct evmap_signal {
	struct event_dlist {
		struct event *lh_first; //first element
	}events;
};
```

event\_dlist的扩展过程
```cpp
LIST_HEAD (event_dlist, event); 

struct event_dlist {
	struct event *lh_first; //first element
}

#define LIST_HEAD(name, type)						\
struct name {								\
	struct type *lh_first;  /* first element */			\
	}
```

## signal\_map init clear
evmap\_signal\_initmap\_
```cpp
void evmap_signal_initmap_(struct event_signal_map *ctx)
{
	ctx->nentries = 0;
	ctx->entries = NULL;
}
```

evmap\_signal\_clear\_
```cpp
void evmap_signal_clear_(struct event_signal_map *ctx)
{
	if (ctx->entries != NULL) {
		int i;
		for (i = 0; i < ctx->nentries; ++i) { //释放二级指针
			if (ctx->entries[i] != NULL)
				mm_free(ctx->entries[i]);
		}
		mm_free(ctx->entries);
		ctx->entries = NULL;
	}
	ctx->nentries = 0;
}
```

## signal\_map add del
evmap\_signal\_add\_
```cpp
int evmap_signal_add_(struct event_base *base, int sig, struct event *ev)
{
	const struct eventop *evsel = base->evsigsel;
	struct event_signal_map *map = &base->sigmap;
	struct evmap_signal *ctx = NULL;

	/* 如果sig比当前map容量大，那么扩展map */
	if (sig >= map->nentries) {
		if (evmap_make_space(
			map, sig, sizeof(struct evmap_signal *)) == -1) //扩展有可能扩出不止一个槽
			return (-1);
	}
	GET_SIGNAL_SLOT_AND_CTOR(ctx, map, sig, evmap_signal, evmap_signal_init,
	    base->evsigsel->fdinfo_len);

	if (LIST_EMPTY(&ctx->events)) {
		if (evsel->add(base, ev->ev_fd, 0, EV_SIGNAL, NULL)
		    == -1)
			return (-1);
	}

	LIST_INSERT_HEAD(&ctx->events, ev, ev_signal_next);

	return (1);
}
```

evmap\_signal\_del\_
```cpp
int evmap_signal_del_(struct event_base *base, int sig, struct event *ev)
{
	const struct eventop *evsel = base->evsigsel;
	struct event_signal_map *map = &base->sigmap;
	struct evmap_signal *ctx;

	if (sig >= map->nentries)
		return (-1);

	GET_SIGNAL_SLOT(ctx, map, sig, evmap_signal);

	LIST_REMOVE(ev, ev_signal_next);

	if (LIST_FIRST(&ctx->events) == NULL) {
		if (evsel->del(base, ev->ev_fd, 0, EV_SIGNAL, NULL) == -1)
			return (-1);
	}

	return (1);
}

```

## signal\_map active
evmap\_signal\_active\_
```cpp
void evmap_signal_active_(struct event_base *base, evutil_socket_t sig, int ncalls)
{
	struct event_signal_map *map = &base->sigmap;
	struct evmap_signal *ctx;
	struct event *ev;

	if (sig < 0 || sig >= map->nentries)
		return;
	GET_SIGNAL_SLOT(ctx, map, sig, evmap_signal);

	if (!ctx)
		return;
	LIST_FOREACH(ev, &ctx->events, ev_signal_next)
		event_active_nolock_(ev, EV_SIGNAL, ncalls);
}
```


## event\_io\_map
io\_map的定义如下：可使用hash table，亦可使用map
参见 [hash table实现](./evmapEvheap.md)
```cpp
#ifdef EVMAP_USE_HT
	#define HT_NO_CACHE_HASH_VALUES
	#include "ht-internal.h"
	struct event_map_entry;
	HT_HEAD(event_io_map, event_map_entry);
#else
	//如果EVMAP_USE_HT没有被定义那么，io_map同于signal_map，只是key为fd
	#define event_io_map event_signal_map
#endif
```

用于fd到该fd一系列event的映射；
```cpp
struct event_io_map {
	//一个evmap_io*或者evmap_signal*的数组；没有元素则为NULL
		//申请entries[fd]时长度为(sizeof(struct evmap_io) + evsel->fdinfo_len)
		//这个多余的空间用于changelist的时候，是fd对应的更改事件在base.changelist中的位置；
	void **entries;
	int nentries; //entries数组的大小；以fd为下标；
};

/** An entry for an evmap_io list: notes all the events that want to read or
	write on a given fd, and the number of each.
  */
struct evmap_io {
	//struct event_dlist events; 扩展如下：
	struct event_dlist {
		struct event *lh_first; //first element
	}events;
	ev_uint16_t nread;
	ev_uint16_t nwrite;
	ev_uint16_t nclose;
};
```

## io\_map add del
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
	/* 获取fd的事件列表 (evmap_io)ctx = io[fd] */
	GET_IO_SLOT_AND_CTOR(ctx, io, fd, evmap_io, evmap_io_init,
						 evsel->fdinfo_len);
	/*
	(gdb) macro exp GET_IO_SLOT_AND_CTOR(ctx, io, fd, evmap_io, evmap_io_init,evsel->fdinfo_len);
	expands to: do {
		if ((io)->entries[fd] == ((void *)0)) {
			多余申请的空间，类型为event_changelist_fdinfo
			用来存储，该fd对应的event_change在base.changelist中的索引；
			(io)->entries[fd] = event_mm_calloc_((1), (sizeof(struct evmap_io)+evsel->fdinfo_len));
			if (__builtin_expect(!!((io)->entries[fd] == ((void *)0)),0))
				return (-1); 
			(evmap_io_init)((struct evmap_io *)(io)->entries[fd]);
		}
		(ctx) = (struct evmap_io *)((io)->entries[fd]);
	} while (0);
	*/

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

## io\_map active
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


## changelist
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
	int n_changes; //当前数组中的有效元素个数；
	int changes_size; //当前数组的容量；
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

event\_changelist\_get\_or\_construct
返回fd/signalNo对应的event\_change，它是changelist中的一个元素；
```cpp
static struct event_change *
event_changelist_get_or_construct(struct event_changelist *changelist,
    evutil_socket_t fd,
    short old_events,
    struct event_changelist_fdinfo *fdinfo)
{
	struct event_change *change;

	if (fdinfo->idxplus1 == 0) {
		/* 没有fd的event_change */
		int idx;
		EVUTIL_ASSERT(changelist->n_changes <= changelist->changes_size);

		/* 如果变化的数量，达到了change list的容量，那么扩容（小于64，或者两倍） */
		if (changelist->n_changes == changelist->changes_size) {
			if (event_changelist_grow(changelist) < 0)
				return NULL;
		}

		idx = changelist->n_changes++; //以当前变化数量为新event_change的索引；
		change = &changelist->changes[idx];
		fdinfo->idxplus1 = idx + 1; //fd、signalNo的变化信息在changelist中的存储位置；

		memset(change, 0, sizeof(struct event_change));
		change->fd = fd;
		change->old_events = old_events;
	} else {
		/* 有fd的event_change，直接获取 */
		change = &changelist->changes[fdinfo->idxplus1 - 1];
		EVUTIL_ASSERT(change->fd == fd);
	}
	return change;
}
```

## event\_changelist\_del\_
```cpp
int
event_changelist_del_(struct event_base *base, evutil_socket_t fd, short old, short events,
    void *p)
{
	struct event_changelist *changelist = &base->changelist;
	struct event_changelist_fdinfo *fdinfo = p;
	struct event_change *change;

	event_changelist_check(base);
	change = event_changelist_get_or_construct(changelist, fd, old, fdinfo);
	event_changelist_check(base);
	if (!change)
		return -1;

	/* A delete on an event set that doesn't contain the event to be
	   deleted produces a no-op.  This effectively emoves any previous
	   uncommitted add, rather than replacing it: on those platforms where
	   "add, delete, dispatch" is not the same as "no-op, dispatch", we
	   want the no-op behavior.

	   If we have a no-op item, we could remove it it from the list
	   entirely, but really there's not much point: skipping the no-op
	   change when we do the dispatch later is far cheaper than rejuggling
	   the array now.

	   As this stands, it also lets through deletions of events that are
	   not currently set.
	 */

	if (events & (EV_READ|EV_SIGNAL)) {
		if (!(change->old_events & (EV_READ | EV_SIGNAL)))
			change->read_change = 0;
		else
			change->read_change = EV_CHANGE_DEL;
	}
	if (events & EV_WRITE) {
		if (!(change->old_events & EV_WRITE))
			change->write_change = 0;
		else
			change->write_change = EV_CHANGE_DEL;
	}
	if (events & EV_CLOSED) {
		if (!(change->old_events & EV_CLOSED))
			change->close_change = 0;
		else
			change->close_change = EV_CHANGE_DEL;
	}

	event_changelist_check(base);
	return (0);
}
```
