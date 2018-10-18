[TOC]

## ReadMe
libevent的入门介绍，或者引言文章。



## install and use

分为event1和event2两个版本，**一般用作学习使用event1，工程则使用event2**.

基于“**事件**”的**异步**通信模式；
事件，网络io、定时器。
异步，函数编写时间、调用时间不是同一时间；（某一条件满足时由kernel触发调用，借助callback实现）



## libevent frame

frame demo

```cpp
/* Step1. create event_base. */
event_base_new();

/* Step2. create event. */
event_new();
bufferevent_socket_new();
    
/* Step3. add event to event_base. */
event_add();

/* Step4. loop waiting for event ready. */
event_base_dispatch();
event_base_loop();

/* Step5. release event_base. */
event_base_free();
```

event_get_supported_methods(); 获取当前系统支持哪些多路IO

event_base_get_method(); 查看当前使用的多路IO

event_reinit();  fork后父进程创建的base可在子进程中生效；





### event_base



### event

分为常规event、还buffer的event



#### normal event

api as follow.

```cpp
struct event *ev = event_new(*base, fd, what, cb, arg);
	//what, ev_read/ev_write/ev_persist/ev_signal/ev_et.
		//ev_timeout已经被废弃，因为可以直接在event_add()中进行指定。
	//cb, typedef void (*cb)(fd, what, void *arg).

int event_add(*ev, timeval *tv);
	//tv, 该ev的超时时间（以秒和微秒进行限定）；
int event_del(*ev);

int event_free(*ev);
```



-------

**事件的非未决、未决状态**

事件非未决：事件没有资格被处理。

> ev被event_new()创建出来，但未被event_add()到base中。

事件未决：事件有资格被处理，但尚未被处理。

> ev已经被event_add()到了base中，但其对应事件未被触发。
> 对于一个非持久事件的状态变更：**nonPending --> pending --> activing  --> processed --> nonPending.**





#### buffer event

主要应用于网络socket，所以**网络通信中的事件首选bufferevent**.而event则不仅可以用于网络socket，还可以用于管道之类的fd。



bufferevent有两个缓冲区（队列实现），read buffer, write buffer.

1. buffer中的数据只能读一次，读了数据就没了。
2. 数据先进先出。
3. read buffer.
   1. 如果buffer中有数据，那么对应的用户注册的读缓冲userReadCB会被触发；
   2. 用户应该在userReadCB中使用bufferevent_read() api来从read buffer中进行数据的读取，而非recv/read()因为fd已经被封装在bufferevent内部了，不对外暴露了；
4. write buffer
   1. 如果buffer中有数据，那么其中的数据会被自动发送给对端，当发送成功之后用户注册的userWriteCB会被触发，这个cb有点鸡肋哈（只能通知写数据完成）；
   2. 应该使用bufferevent_write()往write buffer中写数据，而非send/write()；
5. others..



--------

**API as follow.**

```cpp
struct bufferevent *bev = bufferevent_socket_new(*base, fd, options);
	//options. bev的选项。
		//bev_opt_close_on_free 释放bev时关闭fd。
		//bev_opt_threadsafe 自动为bev分配锁，以便在多线程中安全的使用。
		//bev_opt_defer_callbacks 延迟所有的callback.
		//bev_opt_unlock_callbacks 当设置了threadsafe之后，bev调用用户设定的callback时都会进行加锁，但这个选项会让bev调用user设置的cb时不进行锁定。
	//默认创建出来的bev是，writebuff是enable的，readbuffer是disable的。 --rabin.
void bufferevent_free(*bev);

void bufferevent_setcb(*bev, readcb, writecb, eventcb, void *cbarg);
	//readcb, typedef (*bufferevent_data_cb)(*bev, void *ctx);
		//设置给读缓冲的cb，内部使用bufferevent_read()进行数据的读取；
	//wrtiecb, typedef (*bufferevent_data_cb)(*bev, void *ctx);
		//设置给写缓冲的cb，一般不用置NULL.
	//eventcb, typedef void (*bufferevent_event_cb)(*bev, what, void *ctx)
		//事件回调，处理一些异常、额外的场景；（不能指定只关心的事件---rabin.）
		//what
			//bev_event_reading, 读取操作时发生了某事件，具体看其它标志
			//bev_event_writing, 写入操作时发生了某事件，具体看其它标志
			//bev_event_error, 发生错误，详情调用evutil_socket_error()
			//bev_event_timeout, 发生超时
			//bev_event_eof, 遇到文件结束符
			//bev_event_connected, 连接已建立

size_t bufferevent_read(*bev, void* data, size_t sz);
	//从bev中的readbuff中读取sz长的data
int bufferevent_write(*bev, const void* data, size_t sz);
	//往bev中的writebuff中写入sz长的data

void bufferevent_enable(*bev, what);
	//启用缓冲区的what事件
	//what, ev_read/ev_write
void bufferevent_disable(*bev, what);
	//禁用缓冲区的what事件
	//what, ev_read/ev_write.
short bufferevent_get_enabled(*bev);
	//获取当前bev的禁用状态（借助&进行查看）

int bufferevent_socket_connect(*bev, *sockaddr, addrlen);
	//==connect().
struct evconnlistener *listener = evconnlistener_new(*base, cb, void* arg, flags, backlog, fd);
	//创建监听器，==listen() + accept().
	//cb, void(*evconnlistener_cb)(*listener, fd, *sockaddr, addrlen, void* arg);
		//接收到新连接之后，用户需要做的操作；
struct evconnlistener *listener = evconnlistener_new_bind(*base, cb, void *arg, flags, backlog, *sockaddr, addrlen);
	//创建监听器，==socket() + bind() + listen() + accept().
	//flags, 可识别的标志
		//lev_opt_close_on_free, 释放listener时关闭fd.
		//lev_opt_reuseable, 端口复用
void evconnlistener_free(*listener);
	//释放监听器

```







------------

**bufferevent tcp server side**

as follow.

```cpp
void readCb(*bev, *arg)
{
    /* Step1, read data from readbuffer. */
    char buf[128] = {0};
    bufferevent_read(bev, buff, sizeof(buf));
    /* Step2. process data.*/
    /* Step3. send the result data to writebuffer. */
    const char* sndData = "hello...ack.";
    bufferevent_write(bev, p, strlen(sndData)+1);
}

void writeCb(*bev, *arg)
{
    std::cout<< "already send data." << std::endl;
}

void eventCb(*bev, what, *arg)
{
    if (what & bev_event_eof) {
        cout << "connection closed." << endl;
        bufferevent_free(bev);
    }
    else if (what & bev_event_error) {
        cout << "error occour." << endl;
        bufferevent_free(bev);
    }
}

void listenerCb(*listener, cfd, *sockaddr, addrlen, *arg) 
{
    *base = (event_base*)arg;
    /* Step1, new the bufferevent with cfd, and set the buffevent callback. */
    bufferevent *bev = bufferevent_socket_new(base, cfd, bev_opt_close_free);
    bufferevent_set_cb(bev, readCb, writeCb, eventCb, NULL);
    
    /* Step2, enable the readbuffer. */
    bufferevent_enable(bev, ev_read);
}

int main() 
{
    sockaddr_in addr;
    /* Step1. init base, listener. */
    event_base *base = event_base_new();
	evconnlistener *listener;
    listener = evconnlistener_new_bind(base, listenerCb, (void*)base,
                                       lev_opt_close_on_free|lev_opt_reuseable,
                                       128, &addr, sizeof(addr));
    /* Step2. loop the base. */
    event_base_dispatch(base);
    
    /* Step3. release the base, listener. */
    evconnlistener_free(listener);
    event_base_free(base);
}

```



**bufferevent tcp client side**

as follow

```cpp
int main()
{
    /* Step1. init the base. */
    event_base *base = event_base_new();
    fd = socket();
    
    /* Step2, init the bufferevent and set callback on it. */
    bufferevent *bev = bufferevent_socket_new(base, fd, bev_opt_close_on_free);
    sockaddr_in addr;
    bufferevent_socket_connect(bev, &addr, sizeof(addr));
    bufferevent_set_cb(bev, readCb, writeCb, eventCb, NULL);
    //bufferevent_enable(bev, ev_read);
    
    /* Step3, run the base loop. */
    event_base_dispatch(base);
    
    /* Step4, release the base. */
    event_base_free(base);
    return 0;
}
```



## tcp server

server can use.

```cpp
evconnlistener_new_bind();
	//listener系统函数创建tcp服务；
```



client can use.

```cpp
bufferevent_socket_connect();
	//bufferevent系列网络IO集成了connect()；
```







## Others



