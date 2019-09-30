[TOC]

## ReadMe

进程只有在编译时附带`debug`信息才能用gdb进行调试，如何查看`elf`文件具有debug信息，如下：

```bash
#静态库、动态库都是可以gdb的；

#readelf -S libfun.so | grep debug  #注意是大写S，如果有信息输出则是带debug信息；
#readelf -S libfun.a | grep debug
#readelf -S mainwitha | grep debug
#readelf -S mainwithso | grep debug

#readelf --debug-dump mainwithso | grep debug #如有信息输出则是带debug信息；
#readelf --debug-dump mainwitha | grep debug
#readelf --debug-dump libfun.so | grep debug
#readelf --debug-dump libfun.a | grep debug
```



Single stepping模式是什么；为什么thread no就会显示这样的信息？？

```bash
set print pretty 
whatis 显示变量或函数类型
ptype 显示一个数据结构（如一个结构或C++类）的定义；
```



gdb attach pid

gdb
attach pid



## Linux.GDB

### 指令集

**T.常规操作**

```bash
#------------参数
gdb -args xx ..
set args
show args
info args

#------------------源代码路径
dir <>
directory <>
directory #不带参数：清除所有自定义的路径；
show directories


```

**T.宏扩展**

```bash
(gdb) macro expand MACRO(0)
(gdb) macro exp MACRO(0)
#但要在-ggdb3的级别上才行；
```

**T.窗口**

```bash
#显示汇编窗口
(gdb) layout asm
#源码
(gdb) layout src
#源码与汇编
(gdb) layout split
#寄存器
(gdb) layout regs


#查看当前的开启的窗口
(gdb) info win 
        SRC     (24 lines)  <has focus>
        CMD     (25 lines)


#窗口调大小
(gdb) winheight src +2
(gdb) wh src +2
(gdb) wh cmd -2


#窗口激活
(gdb) focus cmd
Focus set to CMD window.
(gdb) fs src
Focus set to SRC window.
(gdb) fs next
(gdb) fs n
```

**T.信号**

```bash
handle SIGPIPE nostop noprint ignore
```



### 多线程

当你debug多线程app时，
> 当前线程如果停住，那么其它线程都会停住（即所有线程都停住）；
> > 这就是默认all-stop的缘故；
>
> 当你按n/s/c的时候，所有的线程都会跑起来，当然当前调式线程只跑一步；
>
> > 这就是scheduler-locking默认off的缘故；



**T.不停模式**

all-stop模式

> 当你的程序在gdb由于任何原因而停止，所有的线程都会停止，而不仅仅是当前的线程。  
> 一般来说，gdb不能单步所有的线程。因为线程调度是gdb无法控制的。  
> 无论什么时候当gdb停止你的程序，它都会自动切换到触发断点的那个线程。  

non-stop模式 （7.0以后支持）

> 当程序在gdb中停止，只有当前的线程会被停止，而其他线程将会继续运行。  
> 这时候step，next这些命令就只对当前线程起作用。  
> 网络编程常用。

```bash
#默认non-stop模式
(gdb) show non-stop 
Controlling the inferior in non-stop mode is off. #默认，意味着是all-stop???

#开启，在~/.gdbinit中设置如下
set target-async 1  #异步模式，一般用于non-stop
set pagination off  #不要出现 Type <return> to continue 的提示信息 
set non-stop on
```

gdb启动了不停模式，其实就是说，除了断点有关的线程会被停下来， 其他线程会执行执行。在网络程序调试的时候比较有用！ 

**T.线程锁**

gdb的scheduler-locking（在某些操作系统中，你可以通过锁住OS的调度行为）看如下
```bash
#在使用step或者continue命令调试当前线程的时候，其他线程也是同时执行的
#那么怎么只让被调试线程执行呢？看如下：
(gdb) show scheduler-locking
Mode for locking scheduler during execution is "off". 
	#默认off
(gdb) set scheduler-locking off|on|step 
	#off，不锁定任何线程，所有线程都执行；
	#on，只有当前线程执行。
	#step，Single stepping的优化；
		#此模式会阻止其他线程在当前线程单步调试时，抢占当前线。除非以下这些情况其它线程会抢占；
			#当你‘next’一个函数调用的时候。
			#当你使用诸如‘continue’、‘until‘、’finish‘命令的时候。
			#其他线程遇到设置好的断点的时候。

```



**T.其它的设置**

```bash
#查看当前运行的线程
(gdb) info threads 
  Id   Target Id         Frame 
  2    Thread 0x7ffff5de7700 (LWP 6263) ...
* 1    Thread 0x7ffff7fd7720 (LWP 6262) ...


#切换线程
(gdb) thread 2
[Switching to thread 2 (Thread 0x7ffff5de7700 (LWP 6263))]
#0  0x00007ffff6acc48d in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6


#指令用于哪个线程（只能用于显示，而非设置？怎么打断点不生效rwhy）
(gdb) thread apply all bt
(gdb) thread apply 2 bt
(gdb) thread applay [threadno] [all] args


#线程打断点
(gdb) break frik.c:13 thread 2 if bartab > lim
```



### 多进程

```bash
待续
```





### 经典场景

**T.如何gdb死锁程序**

分析死锁问题是比较简单的，因为当发生死锁时，进程会僵住，
这时我们只需要杀死进程，让系统产生一个 core dump 文件，
然后再对这个 core dump 文件进行分析即可。

```bash
kill -s SIGSEGV pid
gdb a.out core.dump.file
thread apply all bt
```





## Win.WinDbg

可以解析如下经典问题：

- crash问题；（查看堆栈 + 变量的值（可attach到进程亦可生成dump文件））
- 死锁问题；（打开所有堆栈即可，知道卡在哪里！（可attach到进程亦可生成dump文件））
- dll库加载的路径；（attach到进程，用`lmf`指令查看模块；而`lm`则可看pdb文件）
- 。。。



### usage

**T.显示堆栈**。

```bash
[~Thread] k[b|p|P|v] [c] [n] [f] [L] [M] [FrameCount]
[~Thread] k[b|p|P|v] [c] [n] [f] [L] [M] = BasePtr [FrameCount]
[~Thread] k[b|p|P|v] [c] [n] [f] [L] [M] = BasePtr StackPtr InstructionPtr
[~Thread] kd [WordCount]
	#Thread线程ID，默认为当前线程，*为所有线程。
	#b 显示每个函数的前3个参数。
	#p 显示每个函数的所有参数。参数列表包括每个参数的类型、名称、值。
	#P 类似p。不同之处在于，每个参数显示在单独的行上面。
	#n 显示调用堆栈中每帧的序号（一般称栈帧，如栈帧3）。
	#FrameCount 指定显示调用堆栈的帧数，即调用堆栈的深度。默认为16进制格式。默认帧数为0x14=20
~*kPn
```

**T.切换调用帧**

```bash
.frame [/c] [/r] [FrameNumber]
	#/r 显示执行该帧时寄存器的值。
```





### start exe with param

打开cmd窗口；
进入WinDbg目录，运行WinDbg.exe exeName param1 param2



### attach to pid

管理员运行WinDbg.
有时候产生不了dump文件（注册表设置了不一定能产生），那么就需要用windbug attach到目标进程进行调试运行；

```bash
file >> attach to process.
g  #继续调试运行
```

直到程序crash，就会显示如下

```bash
(126f4.10460): C++ EH exception - code e06d7363 (first chance)
(126f4.10460): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=00af00c0 ebx=0053005c ecx=005c0064 edx=005c0064 esi=0053005c edi=0ffed040
eip=66bfcf5e esp=0125e248 ebp=0125e26c iopl=0         nv up ei pl nz na pe cy
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010207
*** ERROR: Symbol file could not be found.  Defaulted to export symbols for C:\...\bin\VCRUNTIME140.dll - 
VCRUNTIME140!memcpy+0x4e:
66bfcf5e f3a4            rep movs byte ptr es:[edi],byte ptr [esi]
0:003> kv #or kPn
..
```

此时就可以进行堆栈进行分析了

```bash
file >> symbol file path.  #设置pdb路径，最好跟exe在同一目录；
.reload /f xx.exe          #重新加载exe，就会加载上pdb中的信息；
lm                         #查看进程加载的模块，看看pdb有没有正确被加载；
!analyze -v                #分析出问题堆栈。（！！不加载pdb文件，有可能显示出来是错误的堆栈！！）

#-------------------------举例
0:067> !analyze -v 
Last event: 5d30.5e78: Exit process 0:5d30, code 80  这个应该是没问题的吧！
  debugger time: Sat Dec  8 20:33:39.134 2018 (UTC + 8:00)
```



### analysis dump file

生成dmp文件：任务管理器》右键生成dump文件；（或者crash会生成dump文件）
windbg加载dmp文件：file > open crash dump.






## OSX.lldb

http://lldb.llvm.org/tutorial.html
http://www.cocoachina.com/ios/20150819/11558.html
https://www.jianshu.com/p/67f08a4d8cf2



```bash
# 运行
process launch
run
r
# run with attach
process attach --pid 123
process attach --name Sketch
process attach --name Sketch --waitfor

# 线程状态
thread list
thread backtrace
thread backtrace all

# 查看调用栈状态
frame variable
frame variable variableName  #查看特定的变量
frame select 9
```



### debug core file

`g++ -g`编译出的来<font color=red size=4>app调试信息单独在`app.dSYM`目录中，而不在app里面？？？</font>

```bash
# ulimit -c   #设置小了，有可能产生不了core文件，因为实际产生core大小>设置的值，就不会产生！
unlimited
# ls /cores/
core.58704

# lldb a.out /cores/core.58704    得依赖appname.dSYM目录；
# (lldb) r
Process 58814 launched: '/Users/Lizhong/a.out' (x86_64)
size_t.8 int.4 long.8 ptr.8
Process 58814 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x0)
    frame #0: 0x0000000100000eec a.out`main at offsetof.cpp:29
   26       };
   27  
   28       char *p = NULL;
-> 29       *p = 'c';
   30       /* Output is compiler dependent */
   31  
   32       printf("offsets: i=%ld; c=%ld; d=%ld a=%ld\n",

# rm -rf a.out.dSYM
# lldb a.out /cores/core.58704 
#(lldb) r
Process 58790 launched: '/Users/Lizhong/a.out' (x86_64)
size_t.8 int.4 long.8 ptr.8
Process 58790 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x0)
    frame #0: 0x0000000100000eec a.out`main + 108
a.out`main:
->  0x100000eec <+108>: movb   $0x63, (%rcx)
    0x100000eef <+111>: movl   %r9d, %edx
    0x100000ef2 <+114>: movl   %r10d, %ecx
    0x100000ef5 <+117>: movl   %r11d, %r8d
Target 0: (a.out) stopped.
```

