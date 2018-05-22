[toc]


## 信号的一些概念
generate 信号的生成
- int kill(pid_t pid, int sig);
- int raise(int sig);
- unsigned int alarm(unsigned int seconds); 返回0或者之前设置的余额秒数；
- int pause(void); 把调用进程挂起，不再消耗cpu，直到收到一个信号才恢复执行；

pending挂起：生成但未被传递

deliver 信号的传递

信号的寿命 lifetime = deliver - generate


catch捕获：在deliver到进程后执行了信号处理signal handler
- 默认动作
- 忽略
- 用户定义的回调


## 信号的本质

信号的本质是软件层次上对中断的一种模拟（软中断）。
它是一种异步通信的处理机制，事实上，进程并不知道信号何时到来。

信号分类 
- 非实时信号
	- 0-32  
	- 主要是有可能信号会丢失。
- 实时（可靠）信号
	- SIGRTMIN - SIGRTMAX 
	- 是不是后来添加的；（因为最开始的进程结构体中的任务结构中有关signal的就是一个32位的整数）


## 信号的调用时机

以下是信号的调用时机
![](img\signal-callTime.png)




## PCB与信号

进程的任务结构体（代码示意）
```cpp
struct PCB.task_struct
{
	unsigned long signal;
		//存放接收到但未处理的信号；
		//64bitOS sizeof(unsigned long)=8，那么64位机下信号又是怎么样的呢？

	unsigned long blocked;（阻塞的）
		//存放被阻塞的信号；（sigkill, sigstop不能被阻塞）

	struct signal_struct *sig;
		//对每种信号，各进程可以由PCB的sig属性选择使用自定义的处理 函数，或是系统的缺省处理函数。指派各种信息处理函数的结构定义在include/linux/sched.h中。对信号的检查安排在系统调用结束后，以 及“慢速型”中断服务程序结束后(IRQ#_interrupt()
};

struct signal_struct
{
	int count; //共享信号处理函数的计数器；
	struct sigaction action[32];  //该进程的信号处理函数表；
};

struct sigaction {
	void     (*sa_handler)(int);
	void     (*sa_sigaction)(int, siginfo_t *, void *);
	sigset_t   sa_mask;
	int        sa_flags;
};
```

进程的任务结构体（图示意）
![](img\processStruct.task_struct.png)


## API
### 回调注册

signal() 与 sigaction()
```cpp
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
	//param signum
		//可以指定SIGKILL和SIGSTOP以外的所有信号。
```

struct sigaction
```cpp
struct sigaction {
	void (*sa_handler)(int);                        //回调1
	void (*sa_sigaction)(int, siginfo_t *, void *); //回调2，可向回调发送附加信息；
	sigset_t sa_mask; //在处理信号函数时，阻塞sa_mask所指定的信号集。
	int sa_flags;
	void (*sa_restorer)(void); //已废弃，未使用
}

sigaction.sa_flags 改变信号的行为属性
	SA_SIGINFO   //需要传递附加信息给cb，使用*sa_sigaction()。
	SA_RESETHAND //在执行完该信号处理函数之后，重设为SIG_DFL
	SA_RESTART   //如果信号中断了进程的某个系统调用，则系统自动启动该系统调用。
	SA_NODEFER   //默认在信号处理函数运行时会阻塞给定的信息；该选项不会再阻塞指定信号。 
```


### 信号集处理函数
```cpp
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

### 信号掩码处理  
一个进程的信号屏蔽字（信号掩码）规定了当前阻塞而不能递送给该进程的信号集。  
```cpp
int sigprocmask(int how, const sigset_t *newset, sigset_t *oldset);
how标志
	//@param how
		SIG_BLOCK   //在当前的掩码集的基础上加上newset；
		SIG_UNBLOCK //在当前掩码集中移除newset指定的；
		SIG_SETMASK //设置当前掩码集为newset；
```



## core dump
进程在收到大部分信号时所作的默认动作是使该进程正常终止；

但有些信号会引起进程的非正常终止；

### core dump
非正常终止的进程系统默认的动作是：导出进程的内存映像（core dump），即终止那刻程序的各个变量值、硬件寄存器值、内核关于进程的控制信息；

### core dump堆栈收集
```cpp
#include <setjmp.h>
int setjmp(jmp_buf env);
int sigsetjmp(sigjmp_buf env, int savesigs);
//通过保存进程堆栈环境来保存程序的当前位置和信号掩码；

void longjmp(jmp_buf env, int val);
void siglongjmp(sigjmp_buf env, int val);
//把进程控制返回到sigsetjmp所保存的位置上；
```

## signal No

```bash
root@iZ2zeiijjhtmlueeb5oinuZ:~# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

### SIGABRT - 6

产生信号6的原因，如下：

- 同一内存多次free()
- void assert(scalar expression);触发
- void abort(void);触发



