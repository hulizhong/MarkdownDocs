[toc]

## ReadMe
### VS libevent

libevent是对epoll等C10K问题解决方案的更上一层的封装；

- epoll的api没有CallBack的概念；而libevent已经支持CallBack了。
- epoll只是libevent的一种backend；libevent能支持更多的backend，如win下的IOCP。



### epoll的ET, LT模式
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
- epoll_cteate()

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


- epoll_ctl()

	```cpp
	int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

	//@param op 
		EPOLL_CTL_ADD //把fd注册到epfd上，并把event与fd关联上；
		EPOLL_CTL_MOD //更改关联到fd上的event; 
		EPOLL_CTL_DEL //把fd从epfd中移开，event被忽略并可设为NULL；
		
	//@param event 
		typedef union epoll_data {
			void        *ptr;
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
		EPOLLRDHUP (since Linux 2.6.17) //监听对端断开连接
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


- epoll_wait()

	```cpp
	int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
		//@param events 
			//epoll_wait()返回的可用的事件集；
		//@param maxevents 
			//epoll_wait()最多返回这么多events数量；
		//@param timeout 
			//等待的最小milliseconds数；
			//-1，一直等待，直到有事件发生；
		//@param sigmask 
		//return
			>0; //有事件发生的fd数量；
			=0; //指定时间内，所有fd没有事件发生；（即超时）
			=-1; //发生错误，并设置errno值
		//errno如下：
			EINTR //在事件发生、超时前被信号中断；
			EFAULT //参数events所指向的空间没有写权限；
			EBADF //epfd是非法的文件描述符；
			EINVAL //epfd不是epoll实例；或者maxevents<=0

	int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);
		//在epoll_pwait的时候，避免一些信号的干扰；
		//用法，如下：（进入epoll_pwait之前设置信号掩码，epoll_pwait之后恢复老的信息掩码）
			sigset_t origmask;
			sigprocmask(SIG_SETMASK, &sigmask, &origmask);
			ready = epoll_wait(epfd, &events, maxevents, timeout);
			sigprocmask(SIG_SETMASK, &origmask, NULL);
	```


## demo

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

