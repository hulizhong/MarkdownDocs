[TOC]



## ReadMe

select, poll相关知识讨论；



## About Select





## Select API



### fd_set

socket fd的容器；

```cpp
void FD_ZERO(fd_set *set);
void FD_SET(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_CLR(int fd, fd_set *set);
```



### select

```cpp
/* According to POSIX.1-2001 */
#include <sys/select.h>

/* According to earlier standards */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
	//所有的指针参数同时为入参、出参；timeout的值（表示还剩余多少时间）；
	//nfds, 为三个set中值最高的fd，+1；
	//返回：fd数量（其上的等待事件已准备好）；0超时；-1失败；
```

注意：nfds最大为1024，用`bitset`来置位的，如果nfds>1024，那么返回值将是有问题的！！
FD_SETSIZE fd的最大值。

### pselect

````cpp
#include <sys/select.h>
int pselect(int nfds, fd_set *readfds, fd_set *writefds,
            fd_set *exceptfds, const struct timespec *timeout,
            const sigset_t *sigmask);
````





## About Poll





## Poll API



### pollfd

用于一个socket fd、其上关心的事件、返回的事件

```cpp
#include <poll.h>
struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    	//POLLIN, POLLOUT
    short revents;    /* returned events */
    	//POLLERR, POLLHUP, POLLNVAL
    	//POLLPRI(紧急数据可读), POLLRDHUP(tcp连接对端关闭)
};

```



### poll

等待指定事件的发生

```cpp
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```



### ppoll

poll()在阻塞等待的时候，能够被信号中断，所以就有了ppoll()屏蔽信号处理。

```cpp
int ppoll(struct pollfd *fds, nfds_t nfds, 
          const struct timespec *timeout_ts, const sigset_t *sigmask);
	//sigmask为屏蔽的信号；
```



demo如下

```cpp
sigset_t origmask;
int timeout;

timeout = (timeout_ts == NULL) ? -1 :
          (timeout_ts.tv_sec * 1000 + timeout_ts.tv_nsec / 1000000);
sigprocmask(SIG_SETMASK, &sigmask, &origmask);
ready = poll(&fds, nfds, timeout);
sigprocmask(SIG_SETMASK, &origmask, NULL);
```





## Select VS Poll

select(); 和 poll(); 函数实现差不多的功能，但是还是有如下的区别：

1 select(); 要监听的信号集受到内核的限制，而poll(); 要监听的信号集是可以是自己任意设置的一个数组，不受内核限制。

2 select(); 所有的指针参数均为入参、出参；所以每次select调用完都需要重新设置参数（三个fdset，一个timeout）。而poll没有这个困扰，因为其pollfd结构。

