[toc]

## ReadMe

libevent提供了一种机制
> 当一个文件描述符发生了已定义的事件（网络可读可写、超时、信号）时主动调用回调函数；


libevent主旨
> 旨在替换事件驱动型网络服务中的事件循环；
> 一个应用仅仅需要调用event_dispatch()和动态地在无需改变事件循环的情况下添加/删除事件。？？？


## Demo

```cpp
//创建主通知链base
base = event_base_new();

//创建要监听的事件
listener_event = event_new();
event_add(); //并将其加入到主通知链中；
event_free(); //释放由event_new申请的结构体

//主循环
event_base_dispatch();
```


```cpp
struct event_base* base = event_base_new();

//发生读事件后，从socket中取出数据
//struct event* event_new(struct event_base *base, evutil_socket_t fd, short events, void (*cb)(evutil_socket_t, short, void *), void *arg)
struct event* read_ev = (struct event*)malloc(sizeof(struct event));
event_set(read_ev, sock, EV_READ|EV_PERSIST, on_read, read_ev);

//event_assign(struct event *ev, struct event_base *base, evutil_socket_t fd, short events, void (*callback)(evutil_socket_t, short, void *), void *arg)
event_base_set(base, read_ev);
event_add(read_ev, NULL);

event_base_dispatch(base);

event_del(read_ev);
event_base_free(base);
```

