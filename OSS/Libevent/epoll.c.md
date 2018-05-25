[toc]

## ReadMe
linux platform用的是epoll
MacOS platform用的是kqueue

## eventop定义
eventop的定义如下
```cpp
// Structure to define the backend of a given event_base.
struct eventop {
	//The name of this backend. 
	const char *name;

	//创建一个结构体（被用于backend运行过程中），并返回它；
		//失败，返回NULL
		//成功，返回的指针会被设置于event_base.evbase
	void *(*init)(struct event_base *);

	//对给定的fd/signal使能、消除事件
		//events是我们想触发的事件；包括EV_READ, EV_WRITE, EV_SIGNAL, and EV_ET.
		//old参数是之前我们在此fd/signal上设置的事件；
		//fdinfo参数是一个结构体关联到这个fd上的evmap，第一次监控fd时，evmap的大小为0；
	//返回
		//0 成功
		//-1 失败
	int (*add)(struct event_base *, evutil_socket_t fd, short old, short events, void *fdinfo);
	int (*del)(struct event_base *, evutil_socket_t fd, short old, short events, void *fdinfo);

	//是实现事件分发的关键实现；
		//监控到哪个之前添加的事件准备好了，并对激活的事件调用event_active；
	//返回：
		//0，成功
		//-1，失败
	int (*dispatch)(struct event_base *, struct timeval *);

	//从event_base中释放backend相关的数据
	void (*dealloc)(struct event_base *);

	// Flag: 如果fork之后需要reinitialize evetn_base，那么就置1；
	int need_reinit;

	// Bit-array 这个backend能提供的对事件的特性；
	enum event_method_feature features;

	//为每个fd记录激活的事件；
		//每个fd的evmap入口；
		//被当作上述add/del的最后一个函数参数；
	size_t fdinfo_len;
};
```

## epollops定义
epollops的定义如下
```cpp
static const struct eventop epollops_changelist = {
	"epoll (with changelist)",
	epoll_init,
	event_changelist_add_,
	event_changelist_del_,
	epoll_dispatch,
	epoll_dealloc,
	1, /* need reinit */
	EV_FEATURE_ET|EV_FEATURE_O1| EARLY_CLOSE_IF_HAVE_RDHUP,
	EVENT_CHANGELIST_FDINFO_SIZE
};

const struct eventop epollops = {
	"epoll",
	epoll_init,
	epoll_nochangelist_add,
	epoll_nochangelist_del,
	epoll_dispatch,
	epoll_dealloc,
	1, /* need reinit */
	EV_FEATURE_ET|EV_FEATURE_O1|EV_FEATURE_EARLY_CLOSE,
	0
};
```

## epoll\_init
### 这是epoll_init返回的类型；
```cpp
struct epollop {
	struct epoll_event *events; //epoll的事件
	int nevents; //events的数量；
	int epfd;  //epoll的句柄
#ifdef USING_TIMERFD
	int timerfd;  //timerfd的值，否则为-1；
#endif
};
```

### epoll_init的调用
```cpp
struct event_base* event_base_new(void)
{
	...
	if (cfg) {
		base = event_base_new_with_config(cfg);
		...
	}
	return base;
}

struct event_base* event_base_new_with_config(const struct event_config *cfg)
{
	...
	if ((base = mm_calloc(1, sizeof(struct event_base))) == NULL) 
	...
		
	for (i = 0; eventops[i] && !base->evbase; i++) {
		if (cfg != NULL) {
			/* 从配置中过滤掉不应该的backend. determine if this backend should be avoided */
			if (event_config_is_avoided_method(cfg, eventops[i]->name))
				continue;
			if ((eventops[i]->features & cfg->require_features) != cfg->require_features)
				continue;
		}

		/* 从环境变量中过滤掉不该使用的backend. also obey the environment variables */
		if (should_check_environment && event_is_method_disabled(eventops[i]->name))
			continue;

		base->evsel = eventops[i];

		base->evbase = base->evsel->init(base);
	}

	if (base->evbase == NULL) {
		...
		return NULL;
	}

	...

	return (base);
}

static const struct eventop *eventops[] = {
#ifdef EVENT__HAVE_EVENT_PORTS
	&evportops,
#endif
#ifdef EVENT__HAVE_WORKING_KQUEUE
	&kqops,
#endif
#ifdef EVENT__HAVE_EPOLL
	&epollops,
#endif
#ifdef EVENT__HAVE_DEVPOLL
	&devpollops,
#endif
#ifdef EVENT__HAVE_POLL
	&pollops,
#endif
#ifdef EVENT__HAVE_SELECT
	&selectops,
#endif
#ifdef _WIN32
	&win32ops,
#endif
	NULL
}
```

### epoll_init的逻辑
```cpp
static void* epoll_init(struct event_base *base)
{
	int epfd = -1;
	struct epollop *epollop;

#ifdef EVENT__HAVE_EPOLL_CREATE1
	/* First, try the shiny new epoll_create1 interface, if we have it. */
	epfd = epoll_create1(EPOLL_CLOEXEC);
#endif
	if (epfd == -1) {
		if ((epfd = epoll_create(32000)) == -1) {
			...
			return (NULL);
		}
		evutil_make_socket_closeonexec(epfd);
	}

	if (!(epollop = mm_calloc(1, sizeof(struct epollop)))) {
		close(epfd);
		return (NULL);
	}

	epollop->epfd = epfd;

	/* Initialize fields */
	//INITIAL_NEVENT=32最初只有32个epoll事件可以注册，后续应该是可以增多的吧？
	epollop->events = mm_calloc(INITIAL_NEVENT, sizeof(struct epoll_event));
	if (epollop->events == NULL) {
		...
		return (NULL);
	}
	epollop->nevents = INITIAL_NEVENT;

	if ((base->flags & EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST) != 0 ||
	    ((base->flags & EVENT_BASE_FLAG_IGNORE_ENV) == 0 &&
		evutil_getenv_("EVENT_EPOLL_USE_CHANGELIST") != NULL)) {

		base->evsel = &epollops_changelist;
	}

#ifdef USING_TIMERFD
	/*
		epoll只提供了毫秒级的定时器；
		如果想要更精确的定时器，那么需要用到clock_monotonic_coarse定时器，但是这里我们用timerfd来提供更细粒度的控制；
			我怎么觉得用了timerfd后，还是提供的是之前epoll接口毫秒级？？？？
	*/
	if ((base->flags & EVENT_BASE_FLAG_PRECISE_TIMER) &&
	    base->monotonic_timer.monotonic_clock == CLOCK_MONOTONIC) {
		int fd;
		fd = epollop->timerfd = timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK|TFD_CLOEXEC);
		if (epollop->timerfd >= 0) {
			struct epoll_event epev;
			memset(&epev, 0, sizeof(epev));
			epev.data.fd = epollop->timerfd;
			epev.events = EPOLLIN; //添加timerfd的读事件；
			if (epoll_ctl(epollop->epfd, EPOLL_CTL_ADD, fd, &epev) < 0) {
				...
				epollop->timerfd = -1;
			}
		} else {
			...
			epollop->timerfd = -1;
		}
	} else {
		epollop->timerfd = -1;
	}
#endif

	evsig_init_(base);

	return (epollop);
}
```

## event\_changelist\_add_
## event\_changelist\_del_
## epoll\_nochangelist\_add,
## epoll\_nochangelist\_del,

## epoll\_dispatch
```cpp
static int epoll_dispatch(struct event_base *base, struct timeval *tv)
{
	struct epollop *epollop = base->evbase;
	struct epoll_event *events = epollop->events;
	int i, res;
	long timeout = -1;

#ifdef USING_TIMERFD
	if (epollop->timerfd >= 0) {
		struct itimerspec is;
		is.it_interval.tv_sec = 0;
		is.it_interval.tv_nsec = 0;
		if (tv == NULL) {
			...
		} else {
			...
			is.it_value.tv_sec = tv->tv_sec;
			is.it_value.tv_nsec = tv->tv_usec * 1000;
		}
		/* 这是什么意思？？？TODO: we could avoid unnecessary syscalls here by only
		   calling timerfd_settime when the top timeout changes, or
		   when we're called with a different timeval.
		*/
		if (timerfd_settime(epollop->timerfd, 0, &is, NULL) < 0) {
			event_warn("timerfd_settime");
		}
	} else
#endif
	if (tv != NULL) {
		timeout = evutil_tv_to_msec_(tv);
		if (timeout < 0 || timeout > MAX_EPOLL_TIMEOUT_MSEC) {
			/* Linux kernels can wait forever if the timeout is
			 * too big; see comment on MAX_EPOLL_TIMEOUT_MSEC. */
			timeout = MAX_EPOLL_TIMEOUT_MSEC;
		}
	}

	epoll_apply_changes(base);
	event_changelist_remove_all_(&base->changelist, base);

	EVBASE_RELEASE_LOCK(base, th_base_lock); //释放base，要不调event_add时加不上event了；

	res = epoll_wait(epollop->epfd, events, epollop->nevents, timeout);

	EVBASE_ACQUIRE_LOCK(base, th_base_lock); //锁住base，再处理event callback.

	if (res == -1) {
		...
	}

	event_debug(("%s: epoll_wait reports %d", __func__, res));
	EVUTIL_ASSERT(res <= epollop->nevents);  //断言返回的事件数要小于容器的事件数；

	for (i = 0; i < res; i++) {
		int what = events[i].events;
		short ev = 0;
#ifdef USING_TIMERFD
		if (events[i].data.fd == epollop->timerfd)
			continue;  //超时的不要管，那么超时回调什么时候调用rwhy. --只是限于timerfd，而-1的timer, signal是不走这里的；
#endif

		if (what & (EPOLLHUP|EPOLLERR)) { //把EPOLLxx的标记转为libevent的EV_xx标记；
			ev = EV_READ | EV_WRITE;
		} else {
			if (what & EPOLLIN)
				ev |= EV_READ;
			if (what & EPOLLOUT)
				ev |= EV_WRITE;
			if (what & EPOLLRDHUP)
				ev |= EV_CLOSED;
		}

		if (!ev)
			continue;

		evmap_io_active_(base, events[i].data.fd, ev | EV_ET);  //激活事件入队列，但调用事件callback，留作外层base_loop调用；
	}

	if (res == epollop->nevents && epollop->nevents < MAX_NEVENT) {
		//如果epoll事件容器用完，且小于最大配置，那么按2倍扩容；
		int new_nevents = epollop->nevents * 2;
		struct epoll_event *new_events = mm_realloc(epollop->events,
		    new_nevents * sizeof(struct epoll_event));
		...
	}

	return (0);
}
```

## epoll\_dealloc
