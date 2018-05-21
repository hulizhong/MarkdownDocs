[toc]

## ReadMe

问题：同一个time event可以多次event\_add吗？
> 可以，但只有最后一次设置是生效的；


问题：同一个io event可以多次event\_add吗？
> 可以，但只激活一次，有可能后续的add都没成功？可以查看下event\_add的返回！
>> 有待进一步检查？？？？
>> 返回是0正常的，但不知道libevent内部怎么处理的？有待验证？？？
>> 


问题：同一个fd可以多次event\_add( read event)事件吗？
> 可以，这样一个fd就会有多个read event，且每个read event的cb可以不一样；
> 当fd有数据可读时，这些所有的read event都会被激活，其所注册的不同的cb也会跑起来；
>> 但因为libevent是单线程，所以各个cb都会有先后顺序，多个cb抢着处理相同的fd是有问题的；


## API

```cpp
#define evtimer_set(ev, cb, arg)	event_set((ev), -1, 0, (cb), (arg))

//以下这些别名，只能都是one-shot的超时事件。
#define evtimer_assign(ev, b, cb, arg) \
	event_assign((ev), (b), -1, 0, (cb), (arg))
#define evtimer_new(b, cb, arg)	       event_new((b), -1, 0, (cb), (arg))
#define evtimer_add(ev, tv)		event_add((ev), (tv))
#define evtimer_del(ev)			event_del(ev)
#define evtimer_pending(ev, tv)		event_pending((ev), EV_TIMEOUT, (tv))
#define evtimer_initialized(ev)		event_initialized(ev)
```

- 注意：
	- 老式的timeout\_\*宏都被新式的evtimer\_\*宏所替代。
	- new一个timer的时候：
		- events标志为0（只能激活一次）；其实event_new的时候可以传ev_persist得到持久的timer。
		- fd为-1；（不能为具体的socket fd）
	- 最终其实生效的是event\_add的时候第二个参数。


## 超时监听原理

多路IO复用函数都是有一个超时值的，如epoll\_wait();
如果libevent要监听多个超时事件，那么只需要把最小的超时值作为IO复用函数的超时值；

- 调用事件监听时，只传了一个事件、且是最小超时值的那一个；


而如何管理这些众多的time event，libevent提供了以下两种管理方法；

- 小根堆
- 通用超时（common timeout）


