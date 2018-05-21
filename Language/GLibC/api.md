[toc]

## 目录

- getcwd() 当前工作目录的绝对路径

	```cpp
	#include <unistd.h>

	//废弃不用了，用getcwd()
	char *getwd(char *buf);

	char *getcwd(char *buf, size_t size); 
	/*
		getcwd(NULL, 0);  返回char*的当前目录路径
		getcwd(buf, size);  当前目录放置于buf中；
	*/
	```




## 信号

信号的本质是软件层次上对中断的一种模拟（软中断）。
它是一种异步通信的处理机制，事实上，进程并不知道信号何时到来。

信号分类 
- 非实时信号
	- 0-32  
	- 主要是有可能信号会丢失。
- 实时（可靠）信号
	- SIGRTMIN - SIGRTMAX 


### 信号的调用时机

以下是信号的调用时机
![](img\signal-callTime.png)


```cpp
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
```




### PCB与信号

- unsigned long signal;（未决的？）
> 64bitOS sizeof(unsigned long) = 8, 32bit为4

	进程接收到的信号。每位表示一种信号，共32种。置位有效。

- unsigned long blocked;（阻塞的）

	进程所能接受信号的位掩码。置位表示屏蔽，复位表示不屏蔽。

- struct signal_struct *sig;

	因为signal和blocked都是32位的变量，Linux最多只能接受32种信号。对每种信号，各进程可以由PCB的sig属性选择使用自定义的处理 函数，或是系统的缺省处理函数。指派各种信息处理函数的结构定义在include/linux/sched.h中。对信号的检查安排在系统调用结束后，以 及“慢速型”中断服务程序结束后(IRQ#_interrupt()，参见9。5节“启动内核”)。



## 信号处理函数注册
```cpp
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
	//param signum
		//可以指定SIGKILL和SIGSTOP以外的所有信号。
	//param act
		struct sigaction {
			void (*sa_handler)(int);
			void (*sa_sigaction)(int, siginfo_t *, void *);
			sigset_t sa_mask;
			int sa_flags;
			void (*sa_restorer)(void);
		}
/*
信号回调函数可以采用void (*sa_handler)(int)或void (*sa_sigaction)(int, siginfo_t *, void *)。
	到底采用哪个要看sa_flags中是否设置了SA_SIGINFO位，如果设置了就采用sa_sigaction此时可以向处理函数发送附加信息；
	默认情况下采用void (*sa_handler)(int)，此时只能向处理函数发送信号的数值。
sa_mask 用来设置在处理该信号时暂时将sa_mask指定的信号集搁置。
sa_flags 标志：
	SA_RESETHAND：当调用信号处理函数时，将信号的处理函数重置为缺省值SIG_DFL
	SA_RESTART：如果信号中断了进程的某个系统调用，则系统自动启动该系统调用
	SA_NODEFER ：一般情况下， 当信号处理函数运行时，内核将阻塞该给定信号。但是如果设置了 SA_NODEFER标记， 那么在该信号处理函数运行时，内核将不会阻塞该信号[1] 
sa_restorer 此参数没有使用。
*/
```

## 信号集处理函数
```c
#include <signal.h>
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signum);
int sigdelset(sigset_t *set, int signum);
int sigismember(const sigset_t *set, int signum);
typedef struct {
	unsigned long sig[_NSIG_WORDS]；
} sigset_t
```

## 信号掩码处理  
> 一个进程的信号屏蔽字（信号掩码）规定了当前阻塞而不能递送给该进程的信号集。  

```c
int sigprocmask(int how, const sigset_t *newset, sigset_t *oldset);
how标志
	SIG_BLOCK 在当前的掩码集的基础上加上newset；
	SIG_UNBLOCK 在当前掩码集中移除newset指定的；
	SIG_SETMASK 设置当前掩码集为newset；
```
