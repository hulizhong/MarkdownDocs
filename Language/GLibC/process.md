[toc]

## ReadMe

## About API
大概有如下跟进程有关的API
[fork](#fork)
[exec系列](#exec*)
[system](#system)


### fork
```cpp
#include <unistd.h>
pid_t fork(void);
```

#### 写时复制
子进程会复制父进程的一些数据（堆、栈、静态数据），但会采用写时复制（即只有写操作时，复制才会发生）。


#### Be Careful
如果下面三个分支，不用{}限定的话，很有可能A跑完跑B，B跑完跑C；
因为父、子进程都是从04行开始执行的；
```cpp
01 void fun()
02 {
03 	res = fork();
04 	if (res == 0) {
05 		//A
06 	}
07 	else if(res > 0) {
08 		//B
09 	}
10 	else {
11 		//C
12 	}
13 }
```

### exec*
exec这个系列函数，替换当前的进程映象。

- 参数
	- path可执行文件的文件名（完整路径）。
	- file可执行文件的文件名（可在PATH中搜索到）。
	- arg可变参数；
	- argv参数数组；传被传递给path/file的main函数。
	- Envp设置新程序的环境变量。（默认为全局变量environ指定）
- 返回
	- 正常情况是不返回的。除非是出错了-1，并设置errno。

```cpp
extern char **environ;
int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg, ..., char * const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[], char *const envp[]);
```


#### Be Careful
代码执行路径
Exec*()调用之后的代码是不会再执行了，因为从此处调用进程就已被替换。



### system
开启子进程执行command，此时会把父进程hold on
```cpp
int system(const char *command);
```

如何获取其返回值？



## 僵尸进程
什么是进程的僵尸态？
> 子进程退出之后、父进程未读取其退出状态之前。
> 因为子进程结束运行时，内核不会立即释放该进程的进程表表项，还等待着其父进程来回收；
>

```cpp
//返回的pid_t为子进程的id，退出状态存于status.
pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
```

以上函数都是阻塞的；怎么使其调用高效呢？
> 在事件发生的情况下执行非阻塞调用才能提高程序的效率。
>> 所以：可以注册sigchild信号回调函数，在其中调用wait/waitpid()
>

## 守护进程

## 进程间通信
InterProcess Communication是一组编程接口，能让同一OS内多个线程相互传递、交换信息；
可使异步环境下各进程相互合作、等待，按一定的速度执行；（进程同步）

进程间通信 VS 同步？
> 把‘同步’和‘通讯’的概念分得那么清楚，能通讯就一定是一种同步机制，这是显而易见的。
> 线程之间的叫同步，进程之间的叫通信。

IPC有大概有如下这些：
[管道](#IPC管道)
[信号量](#IPC信号量)
[共享内存](#IPC共享内存)
[消息队列](#IPC消息队列)
[文件句柄](#IPC文件句柄)
[socket](#IPCSocket)
[文件锁](#IPC文件锁)
[高级IPC](#IPC高级IPC)
完了之后，可以查看各种[IPC总结](#IPC总结)


### IPC管道
#### 匿名管道
匿名管道只能用于相关联进程（如父子）间的通信。
```cpp
//单方向的数据传输。
int pipe2(int pipefd[2], int flags);

//全双工管道。
int socketpair(int domain, int type, int protocol, int sv[2]);
```

#### 命名管道
可以突破匿名管道的限制：可用于非关联进程间通信，因为有个全局唯一变量pathname。
```cpp
int mkfifo(const char *pathname, mode_t mode);
```

### IPC信号量
- PV操作
	- P 如果SV>0，那么-1；如果SV=0那么挂起进程执行；
	- V 如果有其它进程因SV而挂起，则唤醒之；否则SV+1；

```cpp
//创建、获取信号量；
int semget(key_t key, int nsems, int semflg);

//对信号量进行PV操作；
int semop(int semid, struct sembuf *sops, unsigned nsops);
int semtimedop(int semid, struct sembuf *sops, unsigned nsops, struct timespec *timeout);

//直接对信号量进行控制；
int semctl(int semid, int semnum, int cmd, ...);
```


### IPC共享内存
速度最快的IPC
> 因为它不涉及进程间的任何数据传输。但带来的问题是：
> 需要其它借助手段来同步进程对共享内存的访问，否则会产生竞争条件。

如下是SystemV(five)的方法
```cpp
//创建、获取共享内存；
int shmget(key_t key, size_t size, int shmflg);

//共享内存使用前、后需要关联到进程地址空间、从进程地址空间中分离开。
void *shmat(int shmid, const void *shmaddr, int shmflg);
int shmdt(const void *shmaddr);

int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

如下是共享内存posix方法
MAP_ANONYMOUS匿名内存映射可用于相关进程间通信。
打开同一文件可实现，无相关进程间通信。
```cpp
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);

int shm_open(const char *name, int oflag, mode_t mode);
int shm_unlink(const char *name);
```


### IPC消息队列
进程间传递二进制块数据的一种简单有效的方式。
> 每个块都有一个特定的类型。
> 接收方可根据类型来有选择的进行数据接收，而非FIFO那样先进先出。

```cpp
int msgget(key_t key, int msgflg);

int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);

int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```

### IPC文件句柄
进程间传递文件描述符
如何让接收方创建出来的fd具有同发送方的fd指向同一文件表项呢（即同一文件句柄）？

这个真的可以吗？？？ -rwhy

#### 父子进程
理所当然父进程中的fd，会复制到子进程中。

#### 不相关进程


### IPC文件锁
此函数只能锁定整个文件，无法锁定文件的某一区域。
dup()或fork()时fd锁状态不会被继承。
```cpp
#include <sys/file.h>
int flock(int fd, int operation);
```

- 锁的类型operation
	- LOCK\_SH 加共享锁。多个进程可同时对同一个文件作共享锁定。
	- LOCK\_EX 加互斥锁。一个文件同时只有一个互斥锁定。
	- LOCK\_NB 非阻塞（锁不上时，立即返回）。通常配合加锁使用。
	- LOCK\_UN 解锁。


### IPCSocket
socket不仅可实现同机器内多进程的通信，亦可实现跨机器的多进程间通信。


### IPC高级IPC


### IPC总结
#### ipcs
如下，可用ipcs来查看SystemV的三种IPC，分别是共享内存、信号量、消息队列；
```bash
root@think# ipcs

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      

------ Semaphore Arrays --------
key        semid      owner      perms      nsems     

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages  
```


