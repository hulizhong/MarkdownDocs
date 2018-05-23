[toc]

## base\_free死锁
场景如下：
> 
> A线程evthread_use_pthreads, base_new然后跑event_base_loop
> B线程event_add, event_del, base_loopbreak, base_free
>> 有时候就会base_free卡住，此时base_loop也未退出来；

查看event_base里有什么相关的结构
```cpp
struct event_base {
#ifndef EVENT__DISABLE_THREAD_SUPPORT
	/* threading support */
	/** A lock to prevent conflicting accesses to this event_base */
	void *th_base_lock;
#endif

	/* Notify main thread to wake up break, etc. */
	/** True(==1) if the base already has a pending notify, and we don't need
	 * to add any more. */
	int is_notify_pending;
	/** A function used to wake up the main thread from another thread. */
	int (*th_notify_fn)(struct event_base *base);
};
```

查看base_loopbreak逻辑
```cpp
int event_base_loopbreak(struct event_base *event_base)
{
	...
	EVBASE_ACQUIRE_LOCK(event_base, th_base_lock); //获得base.th_base_lock锁
	//step. 置loop break标志；
	event_base->event_break = 1;

	//step. 向base发送通知
	if (EVBASE_NEED_NOTIFY(event_base)) {
		r = evthread_notify_base(event_base);
	} else {
		r = (0);
	}
	EVBASE_RELEASE_LOCK(event_base, th_base_lock); //释放base.th_base_lock锁
	return r;
}
```

查看base_loop逻辑
```cpp
int event_base_loop(struct event_base *base, int flags)
{
	const struct eventop *evsel = base->evsel; //得到backend MulitIO

	/* Grab the lock.  We will release it inside evsel.dispatch, and again
	 * as we invoke user callbacks. */
	EVBASE_ACQUIRE_LOCK(base, th_base_lock);

	...

	//gottern是loopexit置的标志，break是loopbreak置的标志；
	base->event_gotterm = base->event_break = 0;

	while (!done) {
		...

		/* Terminate the loop if we have been asked to */
		if (base->event_gotterm) {
			break;
		}

		if (base->event_break) {
			break;
		}

		....

		//A线程，会卡在这里
		res = evsel->dispatch(base, tv_p); //这里会调用backend io的wait函数，如epoll_wait
		
		...

	}
	event_debug(("%s: asked to terminate loop.", __func__));

done:
	...

	EVBASE_RELEASE_LOCK(base, th_base_lock);

	return (retval);
}

//对应MacOS上的kq_dispatch
static int kq_dispatch(struct event_base *base, struct timeval *tv)
{
	...

	EVBASE_RELEASE_LOCK(base, th_base_lock);

	//A线程，会卡在这里，kevent调用相当于epoll_wait()
	res = kevent(kqop->kq, changes, n_changes,
	    events, kqop->events_size, ts_p);

	EVBASE_ACQUIRE_LOCK(base, th_base_lock);

	....

	event_debug(("%s: kevent reports %d", __func__, res));

	for (i = 0; i < res; i++) {
		...
		if (events[i].filter == EVFILT_SIGNAL) {
			evmap_signal_active_(base, events[i].ident, 1);
		} else {
			evmap_io_active_(base, events[i].ident, which | EV_ET);
		}
	}

	return (0);
}
```

查看base_free的逻辑
```cpp
//static void event_base_free_(struct event_base *base, int run_finalizers)
static void event_base_free_(*base, 1)
{
	/* XXXX grab the lock? If there is contention when one thread frees
	 * the base, then the contending thread will be very sad soon. */

	....

	//B线程，会卡在这里；
	if (base->evsel != NULL && base->evsel->dealloc != NULL)
		base->evsel->dealloc(base);

	.....
}

//对应MacOS上的kq_dealloc
static void kq_dealloc(struct event_base *base)
{
	struct kqop *kqop = base->evbase;
	evsig_dealloc_(base); //删除信号相关的
	kqop_free(kqop);  //删除kqueue相关的
}

static void kqop_free(struct kqop *kqop)
{
	if (kqop->changes)
		mm_free(kqop->changes);
	if (kqop->events)
		mm_free(kqop->events);
	if (kqop->kq >= 0 && kqop->pid == getpid())
		close(kqop->kq); //B线程，会卡在这里
	memset(kqop, 0, sizeof(struct kqop));
	mm_free(kqop);
}
```

综上所述：
> 线程死锁，是因为在A线程还在backend io.wait()，而B线程就试图对backend io.close()
> 所以会引起死锁
>> 解决办法如下：
>> event_base_free()只在event_loop线程调用；
>

## 待续

