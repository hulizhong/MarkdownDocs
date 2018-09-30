[TOC]

## ReadMe



## About Epoll

### VS libevent

libevent是对epoll等C10K问题解决方案的更上一层的封装；

- epoll的api没有CallBack的概念；而libevent已经支持CallBack了。
- epoll只是libevent的一种backend；libevent能支持更多的backend，如win下的IOCP。



### ET, LT

- edge-triggered 边沿触发
	- 仅当状态发生变化时才会通知。
		- 就绪之后，如果不处理fd，使fd变为非就绪状态，那么在这之前是不会再有事件通知；
			- 一直循环read()直到错误eagain产生，才算变为非就绪状态，这时才挂起、等待。
			- read()返回的成功数，小于申请读取的字节数。
			- 一直循环write()直到错误eagain产生，才算变为非就绪状态，这时才挂起、等待。
			- write()返回的成功数，小于申请写入的字节数。
	- 高速，错误率比较大。
	- 只支持no_block socket (非阻塞socket)。
		- 以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。
			- epoll返回active events之后，轮询这些fd读、写时不会因数据未准备好而阻塞住；

- level-triggered 水平触发
	- 类似于原来的select/poll操作,只要还有没有处理的事件就会一直通知。
	- 默认模式，错误率比较小。
	- 支持blocksocket和no_blocksocket。
		- 但我觉得：最好也用no_blocksocket，道理同et模式。


## API
### epoll_cteate

创建一个epoll句柄，如下：

```cpp
#include <sys/epoll.h>
//epoll本身也占个文件描述符号，创建成功后可在 /proc/进程id/fd/ 查看；

int epoll_create(int size);
	//@param size：此epfd能监听的socket fd数量；
		//从Linux 2.6.8该参数被忽略，但一定要>0
int epoll_create1(int flags);
	//@param flags 
		//=0，那么除了省略size之外，同于epoll_wait()
		//!=0, 目前只能=EPOLL_CLOEXEC，同于open的fd_cloexec；
			//close on exec, not on-fork, 
			// 如果对描述符设置了FD_CLOEXEC，使用execl执行的程序里，此描述符被关闭，不能再使用它，
			// 但是在使用fork调用的子进程中，此描述符并不关闭，仍可使用。
```



### epoll_ctl

对ep句柄进行操作，如下：

```cpp
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

//@param op 
	EPOLL_CTL_ADD //把fd注册到epfd上，并把event与fd关联上；
	EPOLL_CTL_MOD //更改关联到fd上的event; 
	EPOLL_CTL_DEL //把fd从epfd中移开，event被忽略并可设为NULL；
	
//@param event 
	typedef union epoll_data {
		void        *ptr;  //Notice.这是个泛型指针，可以指向包含callback的结构体；
		int          fd;
		uint32_t     u32;
		uint64_t     u64;
	} epoll_data_t;

	struct epoll_event {
		uint32_t     events;      /* Epoll events */
		epoll_data_t data;        /* User data variable */
	};

//@param event 
	EPOLLIN //可读
	EPOLLOUT //可写
	EPOLLRDHUP (since Linux 2.6.17) //监听对端断开连接（不能再向对端写数据了？）
		//半连接（在ET模式下发送探测对端连接关闭时有用？）
	EPOLLPRI //带外数据（紧急数据到达，可读）
	EPOLLERR //fd上发生错误事件（epoll_wait()一直会等待此类事件，不是必须设置的事件）
	EPOLLHUP //fd挂起；（epoll_wait()一直会等待此类事件，不是必须设置的事件）
	EPOLLET //fd为ET模式（边沿触发），默认为LT模式。
	EPOLLONESHOT (since Linux 2.6.2) //fd只捕获一次事件；（必须调用epoll_ctl(EPOLL_CTL_MOD)来重新安装事件）

//return
	0 //正常
	-1 //错误，并会设置errno
```



### epoll_wait

等待事件发生，如下：

```cpp
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
	//@param events 
		//epoll_wait()返回的可用的事件集；
	//@param maxevents 
		//epoll_wait()最多返回这么多events数量；
	//@param timeout 
		//等待的最小milliseconds数；
		//-1，一直等待，直到有事件发生；

	//return
		>0; //所等待事件发生的fd数量；
		=0; //指定时间内，所有fd没有事件发生；（即超时）
		=-1; //发生错误，并设置errno值
	//errno如下：
		EINTR //在事件发生、超时前被信号中断；
		EFAULT //参数events所指向的空间没有写权限；
		EBADF //epfd是非法的文件描述符；
		EINVAL //epfd不是epoll实例；或者maxevents<=0
```



### epoll_pwait

```cpp
int epoll_pwait(int epfd, struct epoll_event *events,
                int maxevents, int timeout, const sigset_t *sigmask);
	//在epoll_pwait的时候，避免一些信号的干扰；
	//用法，如下：（进入epoll_pwait之前设置信号掩码，epoll_pwait之后恢复老的信息掩码）
		
sigset_t origmask;
sigprocmask(SIG_SETMASK, &sigmask, &origmask);
ready = epoll_wait(epfd, &events, maxevents, timeout);
sigprocmask(SIG_SETMASK, &origmask, NULL);
```



## Demo

```cpp
#define MAX_EVENTS 10
struct epoll_event ev, events[MAX_EVENTS];
int listen_sock, conn_sock, nfds, epollfd;

/* Set up listening socket, 'listen_sock' (socket(), bind(), listen()) */

epollfd = epoll_create(10);
if (epollfd == -1) {
	perror("epoll_create");
	exit(EXIT_FAILURE);
}

ev.events = EPOLLIN;
ev.data.fd = listen_sock;
if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev) == -1) {
	perror("epoll_ctl: listen_sock");
	exit(EXIT_FAILURE);
}

for (;;) {
	nfds = epoll_wait(epollfd, events, MAX_EVENTS, -1);
	if (nfds == -1) {
		perror("epoll_pwait");
		exit(EXIT_FAILURE);
	}

	for (n = 0; n < nfds; ++n) {
		if (events[n].data.fd == listen_sock) {
			conn_sock = accept(listen_sock,(struct sockaddr *) &local, &addrlen);
			if (conn_sock == -1) {
				perror("accept");
				exit(EXIT_FAILURE);
			}
			setnonblocking(conn_sock);
			ev.events = EPOLLIN | EPOLLET;
			ev.data.fd = conn_sock;
			if (epoll_ctl(epollfd, EPOLL_CTL_ADD, conn_sock, &ev) == -1) {
				perror("epoll_ctl: conn_sock");
				exit(EXIT_FAILURE);
			}
		} else {
			do_use_fd(events[n].data.fd);
		}
	}
}
```





## EPoll Reactor

epoll_creat创建的是红黑树

et 边沿触发，buff中未被读尽的数据不会触发监听事件；
	高速模式，只支持non-block fd.

```cpp
ev = EPOLLIN | EPOLLET;
epoll_wait(..);
readn(fd, 10); //will block until read 10 bytes. 导致epoll_wait不能触发；
```



lt 水平触发：默认模式，buff中未被读尽的数据会触发监听事件；
	 低速、支持block, non-block fd. ----丰阻塞fd一般都会对应着忙轮询；
	适合只读取一部分数据就能满足需求；（如读用户自己封装的协议，从网络中读取协议头，再按协议头读协议体这种场景）



优点：高效，突破1024；

缺点：不能跨平台（linux）；



突破1024

```bash
/proc/sys/fs/file-max
	#818735，当前linux所能打开的最大文件数，受硬件限制；
ulimit -n
	#1024，当前用户下，进程能打开的最大文件数；
/etc/security/limits.conf
	#*	soft	nofile	1024  这个就是ulimit -n的默认值
	#*	hard	nofile	10000 这个值是ulimit -n能修改的最大值

```



------

epoll反应堆模型
核心关键字：epoll、ET模式、非阻塞+轮询、void *ptr即泛型指针（用于回调）

原来的模型

```cpp
socket();
bind();
listen();
epfd = epoll_create(); //create red-black tree.
epoll_ctrl(epfd, EPOLL_CTL_ADD, lfd);
while (true) {
    nfds = epoll_wait(epfd, events);
    for (int n=0; n<nfds; n++) {
        if (events[n].data.fd == lfd) {
        	cfd = accept(lfd);
        	epoll_ctrl(epfd, EPOLL_CTL_ADD, cfd);
        }
        else if (events[n].data.fd == cfd) {
            read();
            process(data);
            write(); //对端关闭、对端滑动窗口size=0、本端buff满了，这些都会使write失败；
        }
    }
}
```

反应堆：不仅要监听cfd的读事件，也要监听cfd的写事件；

```cpp
int lfd = socket();
fcntl(lfd, F_SETFL, O_NONBLOCK); //set non block.
bind();
listen();
epfd = epoll_create(MAX_EVENTS+1); //create red-black tree.
struct epoll_event epv; epv.events = EPOLLIN; //epoll_event with in.
epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, EPOLLIN, &epv);
while (true) {
    nfds = epoll_wait(epfd, events, MAX_EVENTS+1, 1000ms);
    for (int n=0; n<nfds; n++) {
        if (events[n].data.fd == lfd) {
        	cfd = accept(lfd);
        	epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, EPOLLIN);
        }
        else if (events[n].data.fd == cfd) {
            if (events[n].events & EPOLLIN) {
                read();
                process(data);                
                epoll_ctl(epfd, EPOLL_CTL_DEL, cfd, NULL);
                epoll_event epv; epv.events = EPOLLOUT; //epoll_event with out.
                epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &epv);
            }
            else if (events[n].events & EPOLLOUT) { 
            	write();                 
                epoll_ctl(epfd, EPOLL_CTL_DEL, cfd, NULL);
                epoll_event epv; epv.events = EPOLLIN; //epoll_event with in.
                epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &epv);
            }
        }
    }
}
```

反应堆（进化）：加了回调机制（这些callback在适当的时机由kernel调用，同于信号处理函数、线程函数）；

```cpp
struct myevent_s
{
    int fd;
    int events;  //fd上监听的事件；
    void *arg;
    void (*callback)(int fd, int events, void* arg);
    int status;  //该fd是否在epoll fd中进行监听；1监听、0未监听；
    char buf[BUFLEN];  //BUFLEN=128
    int len;
};
void eventset(struct myevent_s *ev, int fd, void(*cb)(int, int, void*), void* arg)
{/*Init myevent_s, contain the callback set.*/
    ev->fd = fd;
    ev->callback = cb;
    ev->arg = arg;
    ev->events = 0;
    //...
}
int epfd;
struct myevent_s g_events[MAX_EVENTS+1]; //1024
struct epoll_event events[MAX_EVENTS+1];

void eventadd(int epfd, int events, struct myevent_s *ev)
{/*add events into epfd tree. the epv.data.ptr was myevent_s.*/
    ev->events = events;
    struct epoll_event epv;
    epv.events = ev->events;
    epv.data.fd = ev->fd; //Was not need do. 
    epv.data.ptr = ev;
    if (ev->status == 0) {
        epoll_ctl(epfd, EPOLL_CTL_ADD, ev->fd, &epv);
        ev->status = 1;
    }
}
void eventdel(int epfd, struct myevent_s *ev)
{/*del ev from epfd tree.*/
    if (ev->status == 1) {
        epoll_ctl(epfd, EPOLL_CTL_DEL, ev->fd, NULL); //epoll_ctl_del第三参数可忽略；
        ev->status = 0;
    }
}

void acceptconn(int lfd, int events, void *arg)
{/*accept event callback. when it called? ---the lfd can read, called by kernel.*/
    int cfd = accept(lfd);
    for (int i=0; i<MAX_EVENTS; i++) {
        if (g_events[i].status == 0) break; //find no use position.
    }
    //make sure i!=MAX_EVENTS.
    fcntl(cfd, F_SETFL, O_NONBLOCK);
    eventset(&g_events[i], cfd, recvdata, &g_events[i]);
    eventadd(epfd, EPOLLIN, &g_event[i]);
}
void recvdata(int cfd, int events, void *arg)
{/*read event callback.*/
    struct myevent_s *ev = (struct myevent_s*)arg;
    ev->len = read(cfd, ev->buf, BUFLEN); //should use recv();
    process(ev->buf);
    eventdel(epfd, ev);
    eventset(ev, cfd, senddata, ev);
    eventadd(epfd, EPOLLOUT, ev);
}
void senddata(int cfd, int events, void *arg)
{/*write event callback.*/
    struct myevent_s *ev = (struct myevent_s*)arg;
    write(cfd, ev->buf, ev->len); //should use send();
    eventdel(epfd, ev);
    eventset(ev, cfd, recvdata, ev);
    eventadd(epfd, EPOLLIN, ev);
}

int lfd = socket();
fcntl(lfd, F_SETFL, O_NONBLOCK); //set non block.
bind();
listen();
epfd = epoll_create(MAX_EVENTS+1); //create red-black tree.
eventset(&g_events[MAX_EVENTS], lfd, acceptconn, &g_events[MAX_EVENTS]);
eventadd(epfd, EPOLLIN, &g_events[MAX_EVENTS]);
while (true) {
    nfds = epoll_wait(epfd, events, MAX_EVENTS+1, 1000ms);
    for (int n=0; n<nfds; n++) {
        //用epoll_event.data.ptr指向包裹着callback的结构体；
        struct myevent_s *ev = (struct myevent_s*)events[n].data.ptr;
        if ((events[n].events & EPOLLIN) && (ev->events & EPOLLIN)) {
            ev->callback(ev->fd, events[n].events, ev->arg);
        }
        if ((events[n].events & EPOLLOUT) && (ev->events & EPOLLOUT)) {
             ev->callback(ev->fd, events[n].events, ev->arg);
        }
    }
}
```

