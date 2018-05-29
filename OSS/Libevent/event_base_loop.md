[toc]

## ReadMe
说明base\_loop的运行逻辑，可通过此来了解libevent的机制；

## base\_loop logic
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

## io event add 
### epoll下读事件添加的堆栈
epoll下event\_add会触发如下调用
```cpp
#0  epoll_apply_one_change (base=0x6121a0, epollop=0x612440, ch=0x7fffffffe350) at epoll.c:353
#1  0x00007ffff7baef30 in epoll_nochangelist_add (base=0x6121a0, fd=0, old=0, events=2, p=0x612bd0) at epoll.c:392
#2  0x00007ffff7ba4a8e in evmap_io_add_ (base=0x6121a0, fd=0, ev=0x612b30) at evmap.c:330
#3  0x00007ffff7ba0080 in event_add_nolock_ (ev=0x612b30, tv=0x0, tv_is_absolute=0) at event.c:2600
#4  0x00007ffff7b9f80a in event_add (ev=0x612b30, tv=0x0) at event.c:2445
#5  0x00000000004072da in main (argc=1, argv=0x7fffffffe668) at eventTst.cpp:149
```


## io event active
### epoll下读事件激活的堆栈
首先来看一个EV\_READ事件激活的堆栈吧
```cpp
#0  event_queue_insert_active (base=0x6121a0, evcb=0x612b30) at event.c:3328
#1  0x00007ffff7ba12c4 in event_callback_activate_nolock_ (base=0x6121a0, evcb=0x612b30) at event.c:2973
#2  0x00007ffff7ba1009 in event_active_nolock_ (ev=0x612b30, res=2, ncalls=1) at event.c:2912
#3  0x00007ffff7ba4e87 in evmap_io_active_ (base=0x6121a0, fd=0, events=34) at evmap.c:428
#4  0x00007ffff7baf2f5 in epoll_dispatch (base=0x6121a0, tv=0x0) at epoll.c:500
#5  0x00007ffff7b9dace in event_base_loop (base=0x6121a0, flags=0) at event.c:1947
#6  0x00007ffff7b9d466 in event_base_dispatch (event_base=0x6121a0) at event.c:1772
#7  0x0000000000406fbe in run (arg=0x6121a0) at eventTst.cpp:43
```

### event\_active\_nolock\_
各种backend监听到fd有事件之后，调用此将event转成callback并加到激活队列；
```cpp
void
event_active_nolock_(struct event *ev, int res, short ncalls)
{
	struct event_base *base;

	event_debug(("event_active: %p (fd "EV_SOCK_FMT"), res %d, callback %p",
		ev, EV_SOCK_ARG(ev->ev_fd), (int)res, ev->ev_callback));

	base = ev->ev_base;
	EVENT_BASE_ASSERT_LOCKED(base);

	if (ev->ev_flags & EVLIST_FINALIZING) {
		/* XXXX debug */
		return;
	}

	/* 获取到以后将要传入到event callback的‘激活事件类型’参数 */
	switch ((ev->ev_flags & (EVLIST_ACTIVE|EVLIST_ACTIVE_LATER))) {
	default:
	case EVLIST_ACTIVE|EVLIST_ACTIVE_LATER:
		EVUTIL_ASSERT(0);
		break;
	case EVLIST_ACTIVE:
		/* We get different kinds of events, add them together */
		ev->ev_res |= res;
		return;
	case EVLIST_ACTIVE_LATER:
		ev->ev_res |= res;
		break;
	case 0:
		ev->ev_res = res;
		break;
	}

	/* 如果管理event的base此时正在处理fd比当前fd高的事件，那么置打断标志 */
	if (ev->ev_pri < base->event_running_priority)
		base->event_continue = 1;

	/* 信号事件，设置调用callback的次数 */
	if (ev->ev_events & EV_SIGNAL) {
#ifndef EVENT__DISABLE_THREAD_SUPPORT
		if (base->current_event == event_to_event_callback(ev) &&
		    !EVBASE_IN_THREAD(base)) {
			++base->current_event_waiters;
			EVTHREAD_COND_WAIT(base->current_event_cond, base->th_base_lock);
		}
#endif
		ev->ev_ncalls = ncalls;
		ev->ev_pncalls = NULL;
	}

	/* 将ev转成cb，添加到base的激活队列中去；*/
	event_callback_activate_nolock_(base, event_to_event_callback(ev));
}
```

### event\_callback\_activate\_nolock\_
添加一个callback到base的活动队列中；
```cpp
int
event_callback_activate_nolock_(struct event_base *base,
    struct event_callback *evcb)
{
	int r = 1;

	if (evcb->evcb_flags & EVLIST_FINALIZING)
		return 0;

	/* cb当前是否已经添加过，如果是active_later那么从later队列删除 */
	switch (evcb->evcb_flags & (EVLIST_ACTIVE|EVLIST_ACTIVE_LATER)) {
	default:
		EVUTIL_ASSERT(0);
	case EVLIST_ACTIVE_LATER:
		event_queue_remove_active_later(base, evcb);
		r = 0;
		break;
	case EVLIST_ACTIVE:
		return 0;
	case 0:
		break;
	}

	event_queue_insert_active(base, evcb);

	if (EVBASE_NEED_NOTIFY(base))
		evthread_notify_base(base);

	return r;
}
```

### event\_queue\_insert\_active
将evcb置成evlist_active状态，并添加到base->activequeues队列中；
```cpp
static void
event_queue_insert_active(struct event_base *base, struct event_callback *evcb)
{
	EVENT_BASE_ASSERT_LOCKED(base);

	if (evcb->evcb_flags & EVLIST_ACTIVE) {
		/* Double insertion is possible for active events */
		return;
	}

	INCR_EVENT_COUNT(base, evcb->evcb_flags);

	evcb->evcb_flags |= EVLIST_ACTIVE;

	base->event_count_active++;
	MAX_EVENT_COUNT(base->event_count_active_max, base->event_count_active);
	EVUTIL_ASSERT(evcb->evcb_pri < base->nactivequeues);
	TAILQ_INSERT_TAIL(&base->activequeues[evcb->evcb_pri],
	    evcb, evcb_active_next);
}
```


## process active event
### base\_loop处理激活事件的堆栈
如下是base loop处理就绪事件的调用堆栈；
调用时机：是在base\_loop中的process\_active中进行的；
```cpp
#0  cb (fd=0, events=2, arg=0x612b10) at eventTst.cpp:78 用户的callback,从标准输入读取数据
#1  0x00007ffff7b9ce58 in event_process_active_single_queue (base=0x6121a0, activeq=0x6125f0, max_to_process=2147483647, endtime=0x0) at event.c:1646
#2  0x00007ffff7b9d3de in event_process_active (base=0x6121a0) at event.c:1738
#3  0x00007ffff7b9db3a in event_base_loop (base=0x6121a0, flags=0) at event.c:1961
#4  0x00007ffff7b9d466 in event_base_dispatch (event_base=0x6121a0) at event.c:1772
#5  0x0000000000406fbe in run (arg=0x6121a0) at eventTst.cpp:43 //thread function
```


### event\_process\_active
激活base中的优先级队列；
低优先级的有可能会饿死高优先级；
```cpp
static int
event_process_active(struct event_base *base)
{
	/* Caller must hold th_base_lock */
	struct evcallback_list *activeq = NULL;
	int i, c = 0;
	const struct timeval *endtime;
	struct timeval tv;
	const int maxcb = base->max_dispatch_callbacks;
	const int limit_after_prio = base->limit_callbacks_after_prio;
	if (base->max_dispatch_time.tv_sec >= 0) {
		update_time_cache(base);
		gettime(base, &tv);
		evutil_timeradd(&base->max_dispatch_time, &tv, &tv);
		endtime = &tv;
	} else {
		endtime = NULL;
	}

	//轮询激活事件；
	for (i = 0; i < base->nactivequeues; ++i) {
		//i=event优先级，从高到低轮询同一优先级的所有回调；
		if (TAILQ_FIRST(&base->activequeues[i]) != NULL) {
			base->event_running_priority = i;
			activeq = &base->activequeues[i];
			if (i < limit_after_prio) //高优先级的至多运行int_max个；
				c = event_process_active_single_queue(base, activeq,
				    INT_MAX, NULL);
			else
				c = event_process_active_single_queue(base, activeq,
				    maxcb, endtime); //低优先级的至多运行maxcb个，或者时间到了；
			if (c < 0) {
				goto done;
			} else if (c > 0)
				break; /* Processed a real event; do not
					* consider lower-priority events */
			/* If we get here, all of the events we processed
			 * were internal.  Continue. */
		}
	}

done:
	base->event_running_priority = -1;

	return c;
}
```

### event\_process\_active\_single\_queue
帮助event_process_active调用单个queue上的所有事件的回调，这些事件都是同一优先级的；
调用此函数时，调用者必须持有锁，函数离开时会解锁；
返回成功调用cb的个数；-1如果被signal, event_break打断；
```cpp
static int
event_process_active_single_queue(struct event_base *base,
    struct evcallback_list *activeq,
    int max_to_process, const struct timeval *endtime)
{
	struct event_callback *evcb;
	int count = 0;

	EVUTIL_ASSERT(activeq != NULL);

	/* foreach每个callback */
	for (evcb = TAILQ_FIRST(activeq); evcb; evcb = TAILQ_FIRST(activeq)) {
		struct event *ev=NULL;
		/* 从active队列删除此evcb */
		if (evcb->evcb_flags & EVLIST_INIT) { //event用event_assign初始化过了
			ev = event_callback_to_event(evcb);

			if (ev->ev_events & EV_PERSIST || ev->ev_flags & EVLIST_FINALIZING)
				event_queue_remove_active(base, evcb);
			else
				event_del_nolock_(ev, EVENT_DEL_NOBLOCK); //非ev_persist要删除事件，保证下次不能激活了；
			event_debug((
			    "event_process_active: event: %p, %s%s%scall %p",
			    ev,
			    ev->ev_res & EV_READ ? "EV_READ " : " ",
			    ev->ev_res & EV_WRITE ? "EV_WRITE " : " ",
			    ev->ev_res & EV_CLOSED ? "EV_CLOSED " : " ",
			    ev->ev_callback));
		} else {
			event_queue_remove_active(base, evcb);
			event_debug(("event_process_active: event_callback %p, "
				"closure %d, call %p",
				evcb, evcb->evcb_closure, evcb->evcb_cb_union.evcb_callback));
		}

		if (!(evcb->evcb_flags & EVLIST_INTERNAL))
			++count;


		base->current_event = evcb;
#ifndef EVENT__DISABLE_THREAD_SUPPORT
		base->current_event_waiters = 0;
#endif

		/* 按照callback的closure标志，调用不同的cb */
		switch (evcb->evcb_closure) {
		case EV_CLOSURE_EVENT_SIGNAL:
			EVUTIL_ASSERT(ev != NULL);
			event_signal_closure(base, ev);
			break;
		case EV_CLOSURE_EVENT_PERSIST:
			EVUTIL_ASSERT(ev != NULL);
			event_persist_closure(base, ev);
			break;
		case EV_CLOSURE_EVENT: { //这是咱们平时用的最多的普通事件
			void (*evcb_callback)(evutil_socket_t, short, void *);
			EVUTIL_ASSERT(ev != NULL);
			evcb_callback = *ev->ev_callback;
			EVBASE_RELEASE_LOCK(base, th_base_lock);  //调用user callback之前释放base锁；
			evcb_callback(ev->ev_fd, ev->ev_res, ev->ev_arg);  //调用用户设置的cb；
		}
		break;
		case EV_CLOSURE_CB_SELF: {
			void (*evcb_selfcb)(struct event_callback *, void *) = evcb->evcb_cb_union.evcb_selfcb;
			EVBASE_RELEASE_LOCK(base, th_base_lock);
			evcb_selfcb(evcb, evcb->evcb_arg);
		}
		break;
		case EV_CLOSURE_EVENT_FINALIZE:
		case EV_CLOSURE_EVENT_FINALIZE_FREE: {
			void (*evcb_evfinalize)(struct event *, void *);
			int evcb_closure = evcb->evcb_closure;
			EVUTIL_ASSERT(ev != NULL);
			base->current_event = NULL;
			evcb_evfinalize = ev->ev_evcallback.evcb_cb_union.evcb_evfinalize;
			EVUTIL_ASSERT((evcb->evcb_flags & EVLIST_FINALIZING));
			EVBASE_RELEASE_LOCK(base, th_base_lock);
			evcb_evfinalize(ev, ev->ev_arg);
			event_debug_note_teardown_(ev);
			if (evcb_closure == EV_CLOSURE_EVENT_FINALIZE_FREE)
				mm_free(ev);
		}
		break;
		case EV_CLOSURE_CB_FINALIZE: {
			void (*evcb_cbfinalize)(struct event_callback *, void *) = evcb->evcb_cb_union.evcb_cbfinalize;
			base->current_event = NULL;
			EVUTIL_ASSERT((evcb->evcb_flags & EVLIST_FINALIZING));
			EVBASE_RELEASE_LOCK(base, th_base_lock);
			evcb_cbfinalize(evcb, evcb->evcb_arg);
		}
		break;
		default:
			EVUTIL_ASSERT(0);
		}

		EVBASE_ACQUIRE_LOCK(base, th_base_lock);
		base->current_event = NULL;
#ifndef EVENT__DISABLE_THREAD_SUPPORT
		if (base->current_event_waiters) {
			base->current_event_waiters = 0;
			EVTHREAD_COND_BROADCAST(base->current_event_cond);
		}
#endif

		/* 各种退出foreach的条件 */
		if (base->event_break) //如果调用了_loopbreak()
			return -1;
		if (count >= max_to_process) //如果执行cb个数达到了上限；
			return count;
		if (count && endtime) { //如果执行时间超过了endtime
			struct timeval now;
			update_time_cache(base);
			gettime(base, &now);
			if (evutil_timercmp(&now, endtime, >=))
				return count;
		}
		if (base->event_continue) //如果调用了_loopcontinue()，或者backend激活了一个更低优先级的事件；
			break;
	}
	return count;
}
```



## io event del
### epoll下读事件删除的堆栈
注意：删除事件的时机，是在event_process_active_signle_queue
```cpp
(gdb) bt
#0  epoll_apply_one_change (base=0x6121a0, epollop=0x612440, ch=0x7ffff5de6b30) at epoll.c:272
#1  0x00007ffff7baefb9 in epoll_nochangelist_del (base=0x6121a0, fd=0, old=2, events=2, p=0x612bd0)
    at epoll.c:410
#2  0x00007ffff7ba4d77 in evmap_io_del_ (base=0x6121a0, fd=0, ev=0x612b30) at evmap.c:396
#3  0x00007ffff7ba0a79 in event_del_nolock_ (ev=0x612b30, blocking=0) at event.c:2827
#4  0x00007ffff7b9cbb9 in event_process_active_single_queue (base=0x6121a0, activeq=0x6125f0, 
    max_to_process=2147483647, endtime=0x0) at event.c:1608
#5  0x00007ffff7b9d3de in event_process_active (base=0x6121a0) at event.c:1738
#6  0x00007ffff7b9db3a in event_base_loop (base=0x6121a0, flags=0) at event.c:1961
#7  0x00007ffff7b9d466 in event_base_dispatch (event_base=0x6121a0) at event.c:1772
#8  0x00000000004070de in run (arg=0x6121a0) at eventTst.cpp:43
```

### event\_del\_nolock\_
帮助event_del实现删除事件的功能；
调用者应该持有base.th_base_lock的锁；
第二个参数必须是：EVENT_DEL_{BLOCK, NOBLOCK, AUTOBLOCK,EVEN_IF_FINALIZING}之一；
```cpp
int
event_del_nolock_(struct event *ev, int blocking)
{
	struct event_base *base;
	int res = 0, notify = 0;

	event_debug(("event_del: %p (fd "EV_SOCK_FMT"), callback %p",
		ev, EV_SOCK_ARG(ev->ev_fd), ev->ev_callback));

	/* An event without a base has not been added */
	if (ev->ev_base == NULL)
		return (-1);

	EVENT_BASE_ASSERT_LOCKED(ev->ev_base);

	if (blocking != EVENT_DEL_EVEN_IF_FINALIZING) {
		if (ev->ev_flags & EVLIST_FINALIZING) {
			/* XXXX Debug */
			return 0;
		}
	}

	/* If the main thread is currently executing this event's callback,
	 * and we are not the main thread, then we want to wait until the
	 * callback is done before we start removing the event.  That way,
	 * when this function returns, it will be safe to free the
	 * user-supplied argument. */
	base = ev->ev_base;
#ifndef EVENT__DISABLE_THREAD_SUPPORT
	if (blocking != EVENT_DEL_NOBLOCK &&
	    base->current_event == event_to_event_callback(ev) &&
	    !EVBASE_IN_THREAD(base) &&
	    (blocking == EVENT_DEL_BLOCK || !(ev->ev_events & EV_FINALIZE))) {
		++base->current_event_waiters;
		EVTHREAD_COND_WAIT(base->current_event_cond, base->th_base_lock);
	}
#endif

	EVUTIL_ASSERT(!(ev->ev_flags & ~EVLIST_ALL));

	/* See if we are just active executing this event in a loop */
	if (ev->ev_events & EV_SIGNAL) {
		if (ev->ev_ncalls && ev->ev_pncalls) {
			/* Abort loop */
			*ev->ev_pncalls = 0;
		}
	}

	if (ev->ev_flags & EVLIST_TIMEOUT) {
		/* NOTE: We never need to notify the main thread because of a
		 * deleted timeout event: all that could happen if we don't is
		 * that the dispatch loop might wake up too early.  But the
		 * point of notifying the main thread _is_ to wake up the
		 * dispatch loop early anyway, so we wouldn't gain anything by
		 * doing it.
		 */
		event_queue_remove_timeout(base, ev);
	}

	if (ev->ev_flags & EVLIST_ACTIVE)
		event_queue_remove_active(base, event_to_event_callback(ev));
	else if (ev->ev_flags & EVLIST_ACTIVE_LATER)
		event_queue_remove_active_later(base, event_to_event_callback(ev));

	if (ev->ev_flags & EVLIST_INSERTED) {
		event_queue_remove_inserted(base, ev);
		if (ev->ev_events & (EV_READ|EV_WRITE|EV_CLOSED))
			res = evmap_io_del_(base, ev->ev_fd, ev);
		else
			res = evmap_signal_del_(base, (int)ev->ev_fd, ev);
		if (res == 1) {
			/* evmap says we need to notify the main thread. */
			notify = 1;
			res = 0;
		}
	}

	/* if we are not in the right thread, we need to wake up the loop */
	if (res != -1 && notify && EVBASE_NEED_NOTIFY(base))
		evthread_notify_base(base);

	event_debug_note_del_(ev);

	return (res);
}
```


## base notify
### base内部添加read notify event
base new的时候就创建一个base的通知事件；
用来干什么呢？
> 用来解决多线程下线程的（添加、删除、激活），loop的（停止）被阻塞在backend io.dispatch()上；
>> 比如，在a线程添加fd的read事件，但b线程还卡在io.dispatch()，那么这次add就不能及时反应在backend io上；
>> 比如，在a线程对b线程的base loop发起了停止，但B线程还卡在io.dispatch()，那么不能立即退出；
>> 但是激活呢？--不应该吧，因为已经跳过了io.dispatch() rwhy
> 

```cpp
#0  epoll_apply_one_change (base=0x6121a0, epollop=0x612440, ch=0x7fffffffe220) at epoll.c:292
#1  0x00007ffff7baef30 in epoll_nochangelist_add (base=0x6121a0, fd=11, old=0, events=2, p=0x6127a0)
    at epoll.c:392
#2  0x00007ffff7ba4a8e in evmap_io_add_ (base=0x6121a0, fd=11, ev=0x6123a0) at evmap.c:330
#3  0x00007ffff7ba0080 in event_add_nolock_ (ev=0x6123a0, tv=0x0, tv_is_absolute=0) at event.c:2600
#4  0x00007ffff7ba2f96 in evthread_make_base_notifiable_nolock_ (base=0x6121a0) at event.c:3625
#5  0x00007ffff7ba2e22 in evthread_make_base_notifiable (base=0x6121a0) at event.c:3574
#6  0x00007ffff7b9ac8c in event_base_new_with_config (cfg=0x612160) at event.c:684
#7  0x00007ffff7b9a55c in event_base_new () at event.c:485
#8  0x0000000000407327 in main (argc=1, argv=0x7fffffffe5d8) at eventTst.cpp:101
```

### init notify
evthread\_make\_base\_notifiable\_nolock_
添加一个持久的内部读事件，用于唤醒主线程（即是epoll\_wait之类backend wait吧）；
注意这个读事件的优先级为0，是最高的；
```cpp
static int
evthread_make_base_notifiable_nolock_(struct event_base *base)
{
	void (*cb)(evutil_socket_t, short, void *);
	int (*notify)(struct event_base *);

	if (base->th_notify_fn != NULL) {
		/* The base is already notifiable: we're doing fine. */
		return 0;
	}

#if defined(EVENT__HAVE_WORKING_KQUEUE)
	if (base->evsel == &kqops && event_kq_add_notify_event_(base) == 0) {
		base->th_notify_fn = event_kq_notify_base_;
		/* No need to add an event here; the backend can wake
		 * itself up just fine. */
		return 0;
	}
#endif

#ifdef EVENT__HAVE_EVENTFD
	/* linux会走这个分支，即支持eventfd； */
	base->th_notify_fd[0] = evutil_eventfd_(0,
	    EVUTIL_EFD_CLOEXEC|EVUTIL_EFD_NONBLOCK); //创建eventfd，并设为cloExec, nonblock特性；
	if (base->th_notify_fd[0] >= 0) {
		base->th_notify_fd[1] = -1; //eventfd只需要一个fd，同步双方都对该efd进行读写；
		notify = evthread_notify_base_eventfd; //base的通知函数，往efd写数据。
		cb = evthread_notify_drain_eventfd;    //base通知的回调函数，从efd读数据；
	} else
#endif
	if (evutil_make_internal_pipe_(base->th_notify_fd) == 0) {
		notify = evthread_notify_base_default;
		cb = evthread_notify_drain_default;
	} else {
		return -1;
	}

	base->th_notify_fn = notify;  //base的通知函数；

	/* prepare an event that we can use for wakeup */
	/* 初始化持久读wakeup event；fd,cb */
	event_assign(&base->th_notify, base, base->th_notify_fd[0],
				 EV_READ|EV_PERSIST, cb, base);

	/* we need to mark this as internal event */
	base->th_notify.ev_flags |= EVLIST_INTERNAL;
	event_priority_set(&base->th_notify, 0); //这样的event优先级最高，越低越高；

	return event_add_nolock_(&base->th_notify, NULL, 0);
}
```

### notify with eventfd
evthread_notify_base_eventfd
往efd写数据，通知事件；
```cpp
static int evthread_notify_base_eventfd(struct event_base *base)
{
	ev_uint64_t msg = 1;
	int r;
	do {
		r = write(base->th_notify_fd[0], (void*) &msg, sizeof(msg));
	} while (r < 0 && errno == EAGAIN);

	return (r < 0) ? -1 : 0;
}
```

evthread_notify_drain_eventfd
从efd读数据，通知到达的回调；
通知回调不需要做什么！
因为通知函数，使notify fd可读，打断了backend io.dispatch，达到了效果；
这里只需要把notify fd上的数据读干净，保证下次通知能正常到达；
```cpp
static void evthread_notify_drain_eventfd(evutil_socket_t fd, short what, void *arg)
{
	ev_uint64_t msg;
	ev_ssize_t r;
	struct event_base *base = arg;

	r = read(fd, (void*) &msg, sizeof(msg));
	if (r<0 && errno != EAGAIN) {
		event_sock_warn(fd, "Error reading from eventfd");
	}
	EVBASE_ACQUIRE_LOCK(base, th_base_lock);
	base->is_notify_pending = 0; //接收完，把base的notify_pending重置；
	EVBASE_RELEASE_LOCK(base, th_base_lock);
}
```

### notify with pipe
evthread_notify_base_default

evthread_notify_drain_default


### notify use
evthread_notify_base
告诉正在运行event_loop的线程：让其停止backend的io dispatch函数，并处理激活的callback.
```cpp
/** Tell the thread currently running the event_loop for base (if any) that it
 * needs to stop waiting in its dispatch function (if it is) and process all
 * active callbacks. */
static int
evthread_notify_base(struct event_base *base)
{
	EVENT_BASE_ASSERT_LOCKED(base);
	if (!base->th_notify_fn)
		return -1;
	if (base->is_notify_pending)
		return 0;
	base->is_notify_pending = 1;
	return base->th_notify_fn(base);
}
```

什么时候发通知？？
> 1、有用到了多线程  
> 2、base已经loop起来了  
> 3、跑base loop的线程id!=当前调用线程id  

外加以下调用时机
> event\_base\_loopbreak
> event\_base\_loopcontinue
> event\_add\_nolock\_
> event\_del\_nolock\_
> event\_callback\_active\_nolock\_
> event\_callback\_active\_later\_nolock\_
> event\_base\_del\_virtual_ 
> 

所有的调用都需要用以下宏来进行判断；
```cpp
(gdb) macro expand EVBASE_NEED_NOTIFY(event_base)
expands to: (evthread_id_fn_ != ((void *)0) && (event_base)->running_loop && (event_base)->th_owner_id
!= evthread_id_fn_())
(gdb) p evthread_id_fn_
$1 = (long unsigned int (*)(void)) 0x7ffff7976f27 <evthread_posix_get_id>

//extern unsigned long (*evthread_id_fn_)(void);
```

## priority 
### base priority 
event_base_priority_init

### event priority 
event_priority_set
如下面设置base的notify event的优先级为0
```cpp
event_priority_set(&base->th_notify, 0); //这样便设置成这个事件的callback优先级
```

如果没有调用上述函数进行设置那么event_new的时候默认为
```cpp
int event_assign(...) {
	...
	if (base != NULL) {
		/* by default, we put new events into the middle priority */
		ev->ev_pri = base->nactivequeues / 2;
	}
}
```

### priority原理
在base众多字段中和事件优先级有关系的是如下：
> base.activequeues
> base.nactivequeues
> base.event\_continue
> base.limit\_callbacks\_after\_prio

其中最重要的是：event\_continue
它可以在激活事件、手动调用loopcontinue进行触发；
在循环调用callback list时进行判断是否开启，并退出，进入下一次base loop；（开启新一次的事件分发）

其次重要的是base.limit\_callbacks\_after\_prio
它可以保证，高优先级的先运行完；


看看base.event\_continue有谁在设置
怎么我觉得下面的2处设置没用，---rwhy
因为跑到调用callback的时候，2已经执行完了，再也不能去打断callback队列执行了（它俩是一个线程内的同步序列）；rwhy
```cpp
//1、每次循环都会重置如下开关
base_loop() {
	while (!done) {
		base->event_continue = 0;
		...
	}
}

//2、激活事件时，只要当前激活的事件优先级高于base正在运行的优先级，那么置开关；
//base没有跑callback时置-1
void event_active_nolock_(struct event *ev, int res, short ncalls) {
	if (ev->ev_pri < base->event_running_priority)
		base->event_continue = 1;
}

//3、
int event_base_loopcontinue(struct event_base *event_base) {   
	event_base->event_continue = 1;  
}
```

看看base.event\_continue有谁在用，让执行callback的循环退出，进入下次base\_loop。
```cpp
event_process_active(struct event_base *base) {
	//从高到低优先级执行cb list.
	for (i = 0; i < base->nactivequeues; ++i) {
		if (TAILQ_FIRST(&base->activequeues[i]) != NULL) {
			base->event_running_priority = i;
			activeq = &base->activequeues[i];
				//调用i优先级对应callback list
				c = event_process_active_single_queue(base, activeq, maxcb, endtime) {
					//每执行完一个cb，就要检测
					if (base->event_continue)
						break;
				}
		}
	}

done:
	base->event_running_priority = -1;
}
```

