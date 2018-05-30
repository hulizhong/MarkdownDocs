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
APUE书中：[管道](#IPC管道)
APUE书中：[共享内存](#IPC共享内存)
APUE书中：[消息队列](#IPC消息队列)
APUE书中：[信号量](#IPC信号量)
APUE书中：[socket](#IPCSocket)
APUE书中：[高级IPC](#IPC高级IPC)
[文件锁（记录锁）](#IPC文件锁（记录锁）)
[文件句柄](#IPC文件句柄)
完了之后，可以查看各种[IPC总结](#IPC总结)


### IPC管道
#### 匿名管道
匿名管道只能用于相关联进程（如父子）间的通信；
单向通信；
虽然可以对fd进行read,write，但不属于文件系统，只存在于内存；
```cpp
//单方向的数据传输。
int pipe2(int pipefd[2], int flags);

//全双工管道。
int socketpair(int domain, int type, int protocol, int sv[2]);
```

#### 命名管道
可以在无关的进程之间交换数据；
有路径名与之相关联，它以一种特殊设备文件形式存在于文件系统中；
```cpp
int mkfifo(const char *pathname, mode_t mode);
```

### IPC共享内存
共享内存（Shared Memory），指两个或多个进程共享一个给定的存储区。
> 共享内存是最快的一种IPC，因为进程是直接对内存进行存取。
> 因为多个进程可以同时操作，所以需要进行同步来访问共享内存。
>> 信号量+共享内存通常结合在一起使用，信号量用来同步对共享内存的访问。
>

如下是SystemV(five)的方法
```cpp
//创建、获取共享内存；
int shmget(key_t key, size_t size, int shmflg);

//共享内存使用前、后需要关联到进程地址空间、从进程地址空间中分离开。
void *shmat(int shmid, const void *shmaddr, int shmflg);
int shmdt(const void *shmaddr);

int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

#### posix shm\_
如下是共享内存posix方法
```cpp
int shm_open(const char *name, int oflag, mode_t mode);
	//打开同一文件，可实现无相关进程间通信。
	//MAP\_ANONYMOUS匿名内存映射可用于相关进程间通信。
int shm_unlink(const char *name);
```

修改文件大小
```cpp
int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

映射内存
```cpp
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```


### IPC消息队列
消息队列，是消息的链接表，存放在内核中。
一个消息队列由一个标识符（即队列ID）来标识。
> 消息队列是面向记录的，其中的消息具有特定的格式以及特定的优先级。
> 消息队列独立于发送与接收进程。进程终止时，消息队列及其内容并不会被删除。
> 消息队列可以实现消息的随机查询,消息不一定要以先进先出的次序读取,也可以按消息的类型读取。

```cpp
int msgget(key_t key, int msgflg);

int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);

int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```

#### posix mq\_
如下是posix的消息队列
```cpp
#include <fcntl.h>           /* For O_* constants */
#include <sys/stat.h>        /* For mode constants */
#include <mqueue.h>

mqd_t mq_open(const char *name, int oflag);
mqd_t mq_open(const char *name, int oflag, mode_t mode, struct mq_attr *attr);


int mq_close(mqd_t mqdes);
```


### IPC信号量
它是一个非负计数器。
信号量用于实现进程间的互斥与同步，而不是用于存储进程间通信数据。
> 信号量用于进程间同步，若要在进程间传递数据需要结合共享内存。
> 信号量基于操作系统的PV操作，程序对信号量的操作都是原子操作。
> 每次对信号量的PV操作不仅限于对信号量值加1或减1，而且可以加减任意正整数。
> 支持信号量组。

PV操作
> P 如果SV>0，那么-1；如果SV=0那么挂起进程执行；
> V 如果有其它进程因SV而挂起，则唤醒之；否则SV+1；

应用场景
> 互斥：几个线程（进程）设置一个信号量进行竞争；
> 同步：几个线程（进程）设置多个信号量、并且初始化不同的值，以实现它们之间的顺序执行。

```cpp
//创建、获取信号量；
int semget(key_t key, int nsems, int semflg);

//对信号量进行PV操作；
int semop(int semid, struct sembuf *sops, unsigned nsops);
int semtimedop(int semid, struct sembuf *sops, unsigned nsops, struct timespec *timeout);

//直接对信号量进行控制；
int semctl(int semid, int semnum, int cmd, ...);
```

#### posix sem\_ 
如下是posix的信号量
```cpp
sem_t *sem_open(const char *name, int oflag);
int sem_init(sem_t *sem, int pshared, unsigned int value);
	//pshared =0在线程间共享， !=0在进程间共享；
	//value 信号量的初始值；

int sem_wait(sem_t *sem);
	//P操作，如果value=0那么会阻塞进入睡眠失去CPU直到另一个线程唤醒它。
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);

int sem_post(sem_t *sem);
```

### IPCSocket
socket不仅可实现同机器内多进程的通信，亦可实现跨机器的多进程间通信。


### IPC高级IPC
#### Unix domain socket
该socket用于一台主机的进程间通信，不需要基于网络协议，主要是基于文件系统的。
提供的SOCK\_STREAM，类似于TCP，可靠的字节流。
提供的SOCK\_DGRAM，类似于UDP，不可靠的字节流。

demo代码如下：
```cpp
struct sockaddr_un un;
un.sun_family = AF_LOCAL;
strcpy(un.sun_path, filepath);
bind(sockfd, (struct sockaddr*)&un, sizeof(un));
```


### IPC文件锁（记录锁）
#### flock
此函数只能锁定整个文件，无法锁定文件的某一区域。
> dup()或fork()时fd锁状态不会被继承。
> 有生命周期，进程被销毁，那么锁也自动解了。

其api有如下
```cpp
#include <sys/file.h>
int flock(int fd, int operation);
```

- 锁的类型operation
	- LOCK\_SH 加共享锁。多个进程可同时对同一个文件作共享锁定。
	- LOCK\_EX 加互斥锁。一个文件同时只有一个互斥锁定。
	- LOCK\_NB 非阻塞（锁不上时，立即返回）。通常配合加锁使用。
	- LOCK\_UN 解锁。


#### fcntl
文件锁可以对将要修改文件的某个部分进行加锁，精确控制到字节
```cpp
#include <unistd.h>
#include <fcntl.h>
int fcntl(int fd, int cmd, ... /* arg */ );
	//第二参数cmd
		//F_GETLK:测试能否加锁(不过能加也不一定能加上，非原子操作。一般不用)
		//F_SETLKW:对文件加锁，不能加则阻塞；
		//F_SETLK:对文件加锁，不能加则立即出错返回；
	//第三参数struct flock

struct flock {
   ...
   short l_type;    /* Type of lock: F_RDLCK, F_WRLCK, F_UNLCK */
   short l_whence;  /* How to interpret l_start: SEEK_SET, SEEK_CUR, SEEK_END */
   off_t l_start;   /* Starting offset for lock */
   off_t l_len;     /* Number of bytes to lock */
   pid_t l_pid;     /* PID of process blocking our lock(F_GETLK only) */
   ...
};
```


### IPC文件句柄
进程间传递文件描述符
如何让接收方创建出来的fd具有同发送方的fd指向同一文件表项呢（即同一文件句柄）？

这个真的可以吗？？？ -rwhy

#### 父子进程
理所当然父进程中的fd，会复制到子进程中。

#### 不相关进程



## IPC总结

子章节如下：
[SystemV ipcs](#SystemV_ipcs)
[SystemV vs BSD](#SystemV_vs_BSD)
[SystemV vs Posix](#SystemV_vs_Posix)

管道
速度慢，容量有限，只有父子进程能通讯    

FIFO
任何进程间都能通讯，但速度慢    

消息队列
容量受到系统限制，且要注意第一次读的时候，要考虑上一次没有读完数据的问题    

信号量
不能传递复杂消息，只能用来同步    

共享内存区
能够很容易控制容量，速度快，但要保持同步，比如一个进程在写的时候，另一个进程要注意读写的问题，相当于线程中的线程安全；
共享内存区同样可以用作线程间通讯，不过没这个必要，线程间本来就已经共享了同一进程内的一块内存；


### SystemV ipcs
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


### SystemV vs BSD
Unix操作系统在操作风格上主要分为System V和BSD
> SystemV的代表的操作系统有Solaris操作系统。
> BSD的代表的操作系统有FreeBSD。

BSD（Berkeley Software Distribution，伯克利软件套件）是Unix的衍生系统， BSD用来代表由此派生出的各种套件集合。

Linux之所以被称为类Unix操作系统(Unix Like)，部分原因就是Linux的操作风格是介于上述二者之间，且不同厂商为了照顾不同的用户，其发行版的操作风格有存在差异。


总结
> systemV, BSD是unix的不同分支；
> linux即不是SystemV，也不是BSD，但介于两者之间；

### SystemV vs Posix
Posix是Portable Operating System Interface(可移植性操作系统接口)的简称。

将这两个名词放在一起讨论的一般是在Linux的进程间通信中。
> 共享内存、信号量、消息队列，这三种IPC两种风格都有实现；
>> posix的风格是用\_进行串接各单词，而systemV则是连贯的；
>> systemV IPC是基于内核实现的，通常会有个ipc object；调用其接口一般会陷入内核，效率较低；
>> posix IPC是基于内存实现的，用文件系统路径名来进行标志；其接口效率较高；

如在信号量编程中，有Posix信号量和SystemV信号量。
它们都可以用于进程或者线程间的同步。
然而Posix信号量是基于内存的，即信号量值是放在共享内存中的，它使与文件系统中的路径名对应的名字来标识。
Systemv信号量实现基于内核的，它放在内核里面，要使用SystemV信号量需要进入内核态，所以在多线程编程中一般不建议使用SystemV信号量，因为线程相对于进程是轻量级的，从操作系统的调度开销角度看，如果使用SystemV信号量会使得每次调用都要进入内核态，丧失了线程的轻量优势。


