[toc]

## Event Type

如下是event的类型
```cpp
// 一个定时器时间到了。
// 不需要传此标记给event_new/assign()来获得一个超时事件；
#define EV_TIMEOUT	0x01

// 等待fd可读
#define EV_READ		0x02

// 等待fd可写
#define EV_WRITE	0x04

// 等待一个POSIX signal触发
#define EV_SIGNAL	0x08

// 持续性的事件（事件激活之后监控不会被删除）
// 用于定时器：当激活之后，其超时时间重置为0；
#define EV_PERSIST	0x10

// 使用ET模式，如果backend的io复用模式支持
#define EV_ET		0x20

// 使线程A中的evnet_del不会阻塞等待线程B中的event callback完成；
// 在多线程中应该配合event_finalize()/evetn_free_finalize()使用，来提供安全性；
// 这是个实验性的API，争取在2.1版本变得稳定；
#define EV_FINALIZE     0x40

// 探测连接关闭事件；
// 不是所有的backends都支持这个特性，可用EV_FEATURE_EARLY_CLOSE来检验下；
#define EV_CLOSED	0x80
```

## event STD

如下（event的状态迁移图）
![](img/eventStateTransitionDiagram.jpg)


## API event 

### 事件初始化
- event_new()

	```cpp
	/**
	  创建一个事件

		fd, events决定触发什么事件；
		callback, callback_arg决定事件激活之后该做些什么；

		events
			events包含ev_read, ev_write时，检测fd是否可读、写；
			events包含ev_signal，那么fd就是等待的信号值；
			如果events没有包含上面这些标志，那么此event只能被手动event_active()、超时触发；（此时fd应该为-1）

			ev_persist标志，可使event_add()一直有效，直到event_del()被调用。
			ev_et标志，混搭着ev_read/write使用，但必须backends后端epoll,select支持才行。
				告诉backends的IO复用技术使用edge-triggered模式。
			ev_timeout标志，在此处没有影响。

			可以设置多个事件在同一fd上，但他们必须都在ET模式下，或者非ET模式下；

		当事件event触发时，会调用cb(fd, 触发的事件events, 之前设置回调时一并设置的参数arg)
		触发的events
			ev_read/write/signal
			ev_timeout代表超时；
			ev_et代表ev_triggered事件发生；

	  @param base the event base to which the event should be attached.
	  @param fd the file descriptor or signal to be monitored, or -1.
	  @param events desired events to monitor: bitfield of EV_READ, EV_WRITE,
		  EV_SIGNAL, EV_PERSIST, EV_ET.
	  @param callback callback function to be invoked when the event occurs
	  @param callback_arg an argument to be passed to the callback function

	  @return a newly allocated struct event that must later be freed with
		event_free().
	  @see event_free(), event_add(), event_del(), event_assign()
	 */
	typedef void (*event_callback_fn)(evutil_socket_t, short, void *);
	EVENT2_EXPORT_SYMBOL
	struct event *event_new(struct event_base *, evutil_socket_t, short, event_callback_fn, void *);
	```

- event_set()

	```cpp
	/**
	准备一个新的event.
	新代码中（2.0）不建议使用它了，而是用event_assign(), event_new()替代它；
	  @deprecated event_set() is not recommended for new code, because it requires
		 a subsequent后来的 call to event_base_set() to be safe under most circumstances环境.
		 Use event_assign() or event_new() instead.
	 */
	EVENT2_EXPORT_SYMBOL
	void event_set(struct event *, evutil_socket_t, short, void (*)(evutil_socket_t, short, void *), void *);
	```

- event_assign()

	```cpp
	/**
	  准备一个新的event, 它已经被分配好内存；
		分配的内存可能在堆上，这样做之后可能因结构体大小与后来的版本不兼容。	
			event_get_struct_event_size()可知道一个event占用的大小；
		当事件处于active,pending状态时，调用此函数是不安全的。
		参数同于event_new()

	  @param ev an event struct to be modified
	  @param base the event base to which ev should be attached.
	  @param fd the file descriptor to be monitored
	  @param events desired events to monitor; can be EV_READ and/or EV_WRITE
	  @param callback callback function to be invoked when the event occurs
	  @param callback_arg an argument to be passed to the callback function

	  @return 0 if success, or -1 on invalid arguments.

	  @see event_new(), event_add(), event_del(), event_base_once(),
		event_get_struct_event_size()
	  */
	EVENT2_EXPORT_SYMBOL
	int event_assign(struct event *, struct event_base *, evutil_socket_t, short, event_callback_fn, void *);
	```



### 事件注册
- event_add()

	```cpp
	/**
	  增加一个事件event到pending集合中去；

		事件ev必须由event_assign()赋值, event_new()生成，或者之前加的事件超时了；
		事件ev如果是个定时器，那么会覆盖之前的定时器；（定时器只能有一个？）

	  @param ev an event struct initialized via event_assign() or event_new()
	  @param timeout the maximum amount of time to wait for the event, or NULL
			 to wait forever
	  @return 0 if successful, or -1 if an error occurred
	  @see event_del(), event_assign(), event_new()
	  */
	EVENT2_EXPORT_SYMBOL
	int event_add(struct event *ev, const struct timeval *timeout);

	#include <sys/time.h>
	struct timeval
	{
		time_t tv_sec; //秒 [long int] = 1000ms
		suseconds_t tv_usec; //微秒 [long int] = 0.001ms
	};
	```

### 其它 

