
## ReadMe

Single stepping模式是什么；为什么thread no就会显示这样的信息？？

```bash
set print pretty 
whatis 显示变量或函数类型
ptype 显示一个数据结构（如一个结构或C++类）的定义；
```

## 宏扩展
macro expand MACRO(0)
macro exp MACRO(0)
> 但要在-ggdb3的级别上才行；

## 窗口

```bash
显示汇编窗口
(gdb) layout asm
源码
(gdb) layout src
源码与汇编
(gdb) layout split
寄存器
(gdb) layout regs


查看当前的开启的窗口
(gdb) info win 
        SRC     (24 lines)  <has focus>
        CMD     (25 lines)


窗口调大小
(gdb) winheight src +2
(gdb) wh src +2
(gdb) wh cmd -2


窗口激活
(gdb) focus cmd
Focus set to CMD window.
(gdb) fs src
Focus set to SRC window.
(gdb) fs next
(gdb) fs n
```

## 多线程

当你debug多线程app时，
> 当前线程如果停住，那么其它线程都会停住（即所有线程都停住）；
>> 这就是默认all-stop的缘故；
> 
> 当你按n/s/c的时候，所有的线程都会跑起来，当然当前调式线程只跑一步；
>> 这就是scheduler-locking默认off的缘故；

### all-stop与scheduler-locking
gdb线程的
all-stop模式
> 当你的程序在gdb由于任何原因而停止，所有的线程都会停止，而不仅仅是当前的线程。  
> 一般来说，gdb不能单步所有的线程。因为线程调度是gdb无法控制的。  
> 无论什么时候当gdb停止你的程序，它都会自动切换到触发断点的那个线程。  

non-stop模式 （7.0以后支持）
> 当程序在gdb中停止，只有当前的线程会被停止，而其他线程将会继续运行。  
> 这时候step，next这些命令就只对当前线程起作用。  
> 网络编程常用。

```bash
默认non-stop模式
(gdb) show non-stop 
Controlling the inferior in non-stop mode is off. 默认

开启，在~/.gdbinit中设置如下
set target-async 1 
set pagination off  #不要出现 Type <return> to continue 的提示信息 
set non-stop on
```

gdb的scheduler-locking看如下
```bash
在使用step或者continue命令调试当前线程的时候，其他线程也是同时执行的
那么怎么只让被调试线程执行呢？看如下：
(gdb) show scheduler-locking
Mode for locking scheduler during execution is "off". 
	默认off
(gdb) set scheduler-locking off|on|step 
	off，不锁定任何线程，所有线程都执行；
	on，只有当前线程执行。
	step，Single stepping的优化；
		此模式会阻止其他线程在当前线程单步调试时，抢占当前线。除非以下这些情况其它线程会抢占；
			当你‘next’一个函数调用的时候。
			当你使用诸如‘continue’、‘until‘、’finish‘命令的时候。
			其他线程遇到设置好的断点的时候。

```


### 其它的设置
```bash
查看当前运行的线程
(gdb) info threads 
  Id   Target Id         Frame 
  2    Thread 0x7ffff5de7700 (LWP 6263) ...
* 1    Thread 0x7ffff7fd7720 (LWP 6262) ...


切换线程
(gdb) thread 2
[Switching to thread 2 (Thread 0x7ffff5de7700 (LWP 6263))]
#0  0x00007ffff6acc48d in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6


指令用于哪个线程（只能用于显示，而非设置？怎么打断点不生效rwhy）
(gdb) thread apply all bt
(gdb) thread apply 2 bt
(gdb) thread applay [threadno] [all] args


线程打断点
(gdb) break frik.c:13 thread 2 if bartab > lim
```

## 多进程

```bash
待续
```

## 信号

## 如何gdb死锁程序
分析死锁问题是比较简单的，因为当发生死锁时，进程会僵住，
这时我们只需要杀死进程，让系统产生一个 core dump 文件，
然后再对这个 core dump 文件进行分析即可。

```bash
kill -s SIGSEGV pid
gdb a.out core.dump.file
thread apply all bt
```

