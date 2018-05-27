[toc]

## ReadMe
查看event\_base相关的信息，包含event\_base相定义、event\_base相的操作函数event\_base相\_xx系列函数。

## 多线程与event_base

event_base跨线程add/remove event导致崩溃或者死循环。
> 据查:libvent 1.4.x是非线程安全的，要跨线程执行event_add，会有问题。
> 因此传统做法是通过pipe来通知宿主线程执行event_add操作。
>> libevent 2.0.x通过线程锁做到了线程安全，可以跨线程执行event_add。
>> 需要在创建event_base之前调用evthread_use_pthreads()，需要添加event_pthread 库，函数定义在event/thread.h
>>> 编译的时候需要带上 -levent_pthreads
>


---
让libevent支持多线程（当然也可以自己注册自己的锁、条件变量？？？）
```cpp
/** Sets up Libevent for use with Pthreads locking and thread ID functions.
	构建libevent时带上pthreads参数；
	编译app时带上 -levent_pthreads
	在event_base_new()之前调用

    @return 0 on success, -1 on failure. */
EVENT2_EXPORT_SYMBOL
int evthread_use_pthreads(void);
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
	//set by event_base_loopcontinue调用
	//set by event_active_nolock_添加比当前base->event_running_priority更小的event
	int event_continue; //立即启动一个新的事件循环

	//当前运行事件的优先级；运行哪个fd的回调就会它的值设为fd，没有运行则为-1；
	int event_running_priority;  

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
	struct event_callback *current_event; //当前正在执行的callback，否则为NULL

#ifdef _WIN32
	struct event_iocp_port *iocp;  // iocp支持结构，如果iocp使能；
#endif

	/** Flags that this base was configured with */
	enum event_base_config_flag flags;

	struct timeval max_dispatch_time; //轮询fd上回调时至多运行多长时间
	int max_dispatch_callbacks; //轮询fd上回调时至多运行个数
	//fd大于该值的要进行上面的time,numer的检查；fd越小越优先执行；
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

## API

event_base_dispatch()
```cpp
/**
   Event dispatching分派 loop圈、环

	一直会运行event_base直到
		没有pending、active
		event_base_loopbreak()
		event_base_loopexit()

  @param base the event_base structure returned by event_base_new() or
	 event_base_new_with_config()
  @return 0 if successful, -1 if an error occurred, or 1 if we exited because
	 no events were pending or active.
  @see event_base_loop()
 */
EVENT2_EXPORT_SYMBOL
int event_base_dispatch(struct event_base *);
```

event_base_loop()
```cpp
/**
  等待事件变成active,并运行它们的回调；
	比event_base_dispatch()更灵活；
  默认一直运行直到没有pending/active的事件，或者event_base_loopbreak/event_base_loopexit被调用；
	可以设置flags参数来更改这一默认行为；
	EVLOOP_ONCE： 阻塞直到有一个活跃的event，然后执行完活跃事件的回调就退出。
	EVLOOP_NONBLOCK : 不阻塞，检查哪个事件准备好，调用优先级最高的那一个，然后退出。

  @param eb the event_base structure returned by event_base_new() or
	 event_base_new_with_config()
  @param flags any combination of EVLOOP_ONCE | EVLOOP_NONBLOCK
  @return 0 if successful, -1 if an error occurred, or 1 if we exited because
	 no events were pending or active.
  @see event_base_loopexit(), event_base_dispatch(), EVLOOP_ONCE,
	 EVLOOP_NONBLOCK
  */
EVENT2_EXPORT_SYMBOL
int event_base_loop(struct event_base *, int)
```

event_base_loopbreak()
```cpp
/**
  Abort the active event_base_loop() immediately.

  event_base_loop() will abort the loop after the next event is completed;
  event_base_loopbreak() is typically invoked from this event's callback.
  This behavior is analogous to the "break;" statement.

  Subsequent invocations of event_base_loop() will proceed normally.

  @param eb the event_base structure returned by event_init()
  @return 0 if successful, or -1 if an error occurred
  @see event_base_loopexit()
 */
EVENT2_EXPORT_SYMBOL
int event_base_loopbreak(struct event_base *);
```

event_base_loopexit()
```cpp
/**
  Exit the event loop after the specified time

  The next event_base_loop() iteration after the given timer expires will
  complete normally (handling all queued events) then exit without
  blocking for events again.

  Subsequent invocations of event_base_loop() will proceed normally.

  @param eb the event_base structure returned by event_init()
  @param tv the amount of time after which the loop should terminate,
	or NULL to exit after running all currently active events.
  @return 0 if successful, or -1 if an error occurred
  @see event_base_loopbreak()
 */
EVENT2_EXPORT_SYMBOL
int event_base_loopexit(struct event_base *, const struct timeval *)
```

