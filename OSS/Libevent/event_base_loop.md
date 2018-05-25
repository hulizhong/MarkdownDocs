[toc]

## ReadMe
说明base\_loop的运行逻辑，可通过此来了解libevent的机制；

## base\_loop的逻辑
```cpp
int event_base_dispatch(){
	return (event_base_loop(event_base, 0)); //flag=0那么没有pending事件就会退出、或者出错退出；
}

//阻塞直到执行了一个事件，然后退出；
#define EVLOOP_ONCE	0x01
//检测当前是否已有ready的事件，选取一个优先级最高的执行并退出；（不会阻塞，没有ready的就退出了）
#define EVLOOP_NONBLOCK	0x02
//当没有pending的事件也不退出；直到调用loopexit/loopbreak之类的api手动停止loop
#define EVLOOP_NO_EXIT_ON_EMPTY 0x04

int event_base_loop(struct event_base *base, int flags)
{
	const struct eventop *evsel = base->evsel;
	struct timeval tv;
	struct timeval *tv_p;
	int res, done, retval = 0;

	/* Grab the lock.  We will release it inside evsel.dispatch, and again
	 * as we invoke user callbacks. */
	EVBASE_ACQUIRE_LOCK(base, th_base_lock);

	if (base->running_loop) {
		event_warnx("%s: reentrant invocation.  Only one event_base_loop"
		    " can run on each event_base at once.", __func__);
		EVBASE_RELEASE_LOCK(base, th_base_lock);
		return -1;
	}

	base->running_loop = 1;

	clear_time_cache(base);

	if (base->sig.ev_signal_added && base->sig.ev_n_signals_added)
		evsig_set_base_(base);

	done = 0;

#ifndef EVENT__DISABLE_THREAD_SUPPORT
	base->th_owner_id = EVTHREAD_GET_ID();
#endif

	base->event_gotterm = base->event_break = 0;

	while (!done) {
		base->event_continue = 0;
		base->n_deferreds_queued = 0;

		/* Terminate the loop if we have been asked to */
		if (base->event_gotterm) {
			break;
		}

		if (base->event_break) {
			break;
		}

		tv_p = &tv;
		if (!N_ACTIVE_CALLBACKS(base) && !(flags & EVLOOP_NONBLOCK)) {
			timeout_next(base, &tv_p);
		} else {
			/*
			 * if we have active events, we just poll new events
			 * without waiting.
			 */
			evutil_timerclear(&tv);
		}

		/* If we have no events, we just exit */
		if (0==(flags&EVLOOP_NO_EXIT_ON_EMPTY) &&
		    !event_haveevents(base) && !N_ACTIVE_CALLBACKS(base)) {
			event_debug(("%s: no events registered.", __func__));
			retval = 1;
			goto done;
		}

		event_queue_make_later_events_active(base);

		clear_time_cache(base);

		res = evsel->dispatch(base, tv_p);

		if (res == -1) {
			event_debug(("%s: dispatch returned unsuccessfully.",
				__func__));
			retval = -1;
			goto done;
		}

		update_time_cache(base);

		timeout_process(base);

		if (N_ACTIVE_CALLBACKS(base)) {
			int n = event_process_active(base);
			if ((flags & EVLOOP_ONCE)
			    && N_ACTIVE_CALLBACKS(base) == 0
			    && n != 0)
				done = 1;
		} else if (flags & EVLOOP_NONBLOCK)
			done = 1;
	}
	event_debug(("%s: asked to terminate loop.", __func__));

done:
	clear_time_cache(base);
	base->running_loop = 0;

	EVBASE_RELEASE_LOCK(base, th_base_lock);

	return (retval);
}
```

## event_base定义
```cpp
struct event_base {
	const struct eventop *evsel; //backend的各种回调函数
	void *evbase;  //跑backend所需要的数据

	/** List of changes to tell backend about at next dispatch.  Only used
	 * by the O(1) backends. */
	struct event_changelist changelist;

	/** Function pointers used to describe the backend that this event_base
	 * uses for signals */
	const struct eventop *evsigsel;
	/** Data to implement the common signal handelr code. */
	struct evsig_info sig;

	/** Number of virtual events */
	int virtual_event_count;
	/** Maximum number of virtual events active */
	int virtual_event_count_max;
	int event_count; //当前监控的数量（即add的）
	int event_count_max;  //最大监控事件数量，超过会怎么样？
	int event_count_active; //当前base里面激活的事件数量
	int event_count_active_max; //最大激活事件数量，超过会怎么样？

	int event_gotterm; //base_loopexit的flag
	int event_break;   //base_loopbreak的flag.（立即退出）
	/** Set if we should start a new instance of the loop immediately. */
	int event_continue;

	int event_running_priority;  //当前运行事件的优先级

	int running_loop; //flag, 已经用base_loop把这个base跑起来了；防止重复调用；

	/** Set to the number of deferred_cbs we've made 'active' in the
	 * loop.  This is a hack to prevent starvation; it would be smarter
	 * to just use event_config_set_max_dispatch_interval's max_callbacks
	 * feature */
	int n_deferreds_queued;

	//管理激活（就绪）事件
	struct evcallback_list *activequeues; //当前激活事件队列，数组索引是cb的优先级；
	int nactivequeues; //上面数组的长度
	struct evcallback_list active_later_queue; //下一次处理事件时变为active的cb list；

	/* common timeout logic */

	/** An array of common_timeout_list* for all of the common timeout
	 * values we know. */
	struct common_timeout_list **common_timeout_queues;
	int n_common_timeouts; //common_timeout_queues中实体的个数
	int n_common_timeouts_allocated; //common_timeout_queues的总大小

	struct event_io_map io; //从fd到事件的映射

	struct event_signal_map sigmap;  //从信号到事件的映射；

	struct min_heap timeheap; //超时事件的优先级队列

	/** Stored timeval: used to avoid calling gettimeofday/clock_gettime
	 * too often. */
	struct timeval tv_cache;

	struct evutil_monotonic_timer monotonic_timer;

	/** Difference between internal time (maybe from clock_gettime) and
	 * gettimeofday. */
	struct timeval tv_clock_diff;
	/** Second in which we last updated tv_clock_diff, in monotonic time. */
	time_t last_updated_clock_diff;

#ifndef EVENT__DISABLE_THREAD_SUPPORT
	/* threading support */
	unsigned long th_owner_id; //运行base_loop的线程ID
	void *th_base_lock;  //锁，锁住这个event_base
	/** A condition that gets signalled when we're done processing an
	 * event with waiters on it. */
	void *current_event_cond;
	/** Number of threads blocking on current_event_cond. */
	int current_event_waiters;
#endif
	/** The event whose callback is executing right now */
	struct event_callback *current_event;

#ifdef _WIN32
	struct event_iocp_port *iocp;  // iocp支持结构，如果iocp使能；
#endif

	/** Flags that this base was configured with */
	enum event_base_config_flag flags;

	struct timeval max_dispatch_time;
	int max_dispatch_callbacks;
	int limit_callbacks_after_prio;

	/* Notify main thread to wake up break, etc. */
	int is_notify_pending; //=1，如果当前base已经有未决的通知；我们不需要再向base发送通知了；
	evutil_socket_t th_notify_fd[2];  //一对socketpair，被某些函数用于唤醒主线程；
	struct event th_notify;  //一个event，被某些函数用于唤醒主线程；
	int (*th_notify_fn)(struct event_base *base); //一个函数，被用于在另外一个线程唤醒主线程；

	/** Saved seed for weak random number generator. Some backends use
	 * this to produce fairness among sockets. Protected by th_base_lock. */
	struct evutil_weakrand_state weakrand_seed;

	/** List of event_onces that have not yet fired. */
	LIST_HEAD(once_event_list, event_once) once_events;

};
```
