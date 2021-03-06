[TOC]

## base\_free deadlock.
场景如下：
> A线程evthread_use_pthreads, base_new然后跑event_base_loop
> B线程event_add, event_del, base_loopbreak, base_free
>
> > 有时候就会base_free卡住，此时base_loop也未退出来；

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

## cant stop base\_dispatch
base\_loopbreak cant stop base\_dispatch
线程函数A
```cpp
void threadCB(event_base *base)
{
	event_base_dispatch(base); 
}
```

主线程B
```cpp
struct event_base *base = event_base_new();
boost::thread(threadCB, base);
sleep(4); //do some work.

int res = event_base_loopbreak(base);
while (res != 0) {
	res = event_base_loopbreak(base); //一直循环在此处
}
```

解决：在主线程B处添加多线程声明（创建base之前）
```cpp
#ifdef WIN32             
	evthread_use_windows_threads();  
#else                    
	evthread_use_pthreads();
#endif
```

### loopbreak VS loopexit
loopbreak，立即退出；
如果当前正在运行active events的cb，那么运行完当前的cb就会停止；（不是一系列哈）
如果当前没有运行active events的cb，那么直接退出；
```cpp
int event_base_loopbreak(struct event_base *);
```

loopexit，给定时间退出；（如果无时间值则立即退出）
如果当前正在运行active events的cb，那么运行完所有的cb才会停止；
立即退出：（如果当前没有跑active events的cb，那么下一轮cb调完才会停止）
```cpp
int event_base_loopexit(struct event_base *, const struct timeval *);
```


## 给libevnet注册log callback

写日志的callback
```cpp
/*
log handler的形式：
	typedef void (*event_log_cb)(int severity, const char *msg);
在自己提供的这个callback里面不能调用libevent提供的任务api，否则会产生未定义行为；
*/
static void eventLogCb(int severity, const char *msg)
{                               
	const char *s;              
	switch (severity) {         
		case _EVENT_LOG_DEBUG: s = "debug"; break;
		case _EVENT_LOG_MSG:   s = "msg";   break;
		case _EVENT_LOG_WARN:  s = "warn";  break;
		case _EVENT_LOG_ERR:   s = "error"; break;
		default:               s = "?";     break; /* never reached */
	}                           
	printf("[%s] %s\n", s, msg);
}  
```

注册callback
```cpp
if (mBase == NULL) 
{                  
	/*
	打开日志开关，并将日志送往默认的log handler；
	这是个全局开关，必须先于base_new, use_ptrheads函数调用；
	参数只能是常量：
		EVENT_DBG_ALL 开启debugging日志；
		EVENT_DBG_NONE 关闭debugging日志； 
	*/
	event_enable_debug_logging(EVENT_DBG_ALL);
	/*
	设置日志输出回调；
		如果参数为NULL，那么cb设为默认的log handler.
	*/
	event_set_log_callback(eventLogCb);

	evthread_use_pthreads();
	mBase = event_base_new();
}
```

补充：
用户自定义的log handler callback将会被设置到全局变量log\_fn
默认的log handler为event\_log
```cpp
static void event_log(int severity, const char *msg)
{
	if (log_fn)
		log_fn(severity, msg);
	else {
		const char *severity_str;
		switch (severity) {
		case EVENT_LOG_DEBUG:
			severity_str = "debug";
			break;
		case EVENT_LOG_MSG:
			severity_str = "msg";
			break;
		case EVENT_LOG_WARN:
			severity_str = "warn";
			break;
		case EVENT_LOG_ERR:
			severity_str = "err";
			break;
		default:
			severity_str = "???";
			break;
		}
		(void)fprintf(stderr, "[%s] %s\n", severity_str, msg);
	}
}
```

## epoll使用changelist
在linux平台下epoll默认使用nochangelist；如何让其使用changelist呢？
```cpp
struct event_config *evcfg = event_config_new();
event_config_set_flag(evcfg, EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST);
mBase = event_base_new_with_config(evcfg);
```



## event_base_new failed.

server code demo as follow.

```cpp
#ifdef WIN32
	evthread_use_windows_threads();
#else
	evthread_use_pthreads();
#endif

if (event_base_new() == NULL) {
    printf("init event_base failed.\n");
}
```

由于是server端程序，所以<font color=red>在daemon模式</font>下运行会得到如下日志

> init event_base failed.

<font color=red>为了得到更详尽的日志，在前台模式下运行程序</font>，得到如下日志

> Use windows threads lock in libevent
> [warn] evsig_init_: socketpair: 提供了一个无效的参数。
> [warn] event_base_new_with_config: Unable to make base notifiable.
> ...

跟踪libevent代码，

```cpp
int evsig_init_(struct event_base *base) {
    int evutil_make_internal_pipe_(evutil_socket_t fd[2]);
}

int evutil_make_internal_pipe_(evutil_socket_t fd[2]) {
    //call pipe2() or pipe() when switch on;
    //socketpair(AF_INET) or unix socket.
}
```

**综上所述**

很有可能因为网络问题造成windows下socketpaire失败，导致evsig_init_()失败，最终引起event_base_new()失败；--应该查看网络状况！



## https server longconnection.

Version. 2.1.5-beta
用libcurl作为客户端 对 libevent http server进行长连接请求； ---可以正常工作；
用libcurl作为客户端 对 libevent https server进行长连接请求；---第二个https请求就会被reset掉；



Solution 1.  去掉libcurl上传大于1024 byte就100-continue的机制；

Solution 2.  将以下代码write enable放到函数的最后去；（一般都是处理完数据之后才enable write.）

```cpp
static void 
evhttp_send_continue(struct evhttp_connection *evcon,
			struct evhttp_request *req)
{
	bufferevent_enable(evcon->bufev, EV_WRITE); //move the enable write to last.
	evbuffer_add_printf(bufferevent_get_output(evcon->bufev),
			"HTTP/%d.%d 100 Continue\r\n\r\n",
			req->major, req->minor);
	evcon->cb = evhttp_send_continue_done;
	evcon->cb_arg = NULL;
	bufferevent_setcb(evcon->bufev,
	    evhttp_read_cb,
	    evhttp_write_cb,
	    evhttp_error_cb,
	    evcon);
}
```



<font color=red>疑问：虽然提前enable了，但为什么会reset掉client的https connection呢？？</font>？





## 待续

