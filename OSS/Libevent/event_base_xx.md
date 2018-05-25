[toc]

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



## API

- event_base_dispatch()

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

- event_base_loop()

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

- event_base_loopbreak()

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

- event_base_loopexit()

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

