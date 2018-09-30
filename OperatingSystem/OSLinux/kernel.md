[TOC]

## ReadMe





## Von Neumann Machine

冯诺依曼体系结构  == 存储程序体系结构 == 存储程序计算机；
从硬件来看：CPU部件 ---总线--- MEM部件
从程序来看：内存存储指令、数据；CPU解释、执行指令。

CPU内部：
16位机上的Instruction Pointer寄存器：当是一个指针吧，总指向内存中的CodeSegment
32位机上的EIP
64位机上的RIP

内存内部：
Code Segment：

CPU内部：
cpu总是从IP所指向的内存，读取一条指令执行、并让IP++指向下一条，如此反复循环。



ApplactionProgramInterface，程序员与计算机的接口界面；
ApplactionBinaryInterface，程序与CPU的接口界面。
	把指令encoding成cpu认识的二进制指令；
	寄存器的布局；
	大部分指令都可以直接访问内存；（对于x86来说）



EIP
执行完当前指令后，总是会自加、指向下一条指令；
每条指令的长度都不一样；
EIP还可被CALL, RET, JMP, conditional JMP指令所改变；



## x86 Assembly 

### Register

通用寄存器

| 16-bit                        | 32-bit | 64-bit | 名称         | 作用           |
| ----------------------------- | ------ | ------ | ------------ | -------------- |
| AX <br />高8位AH<br />低8位AL | EAX    |        | 累加器       | Accumulator    |
| BX  BH,BL                     | EBX    |        | 基地址寄存器 | Base Register  |
| CX  CH,CL                     | ECX    |        | 计数寄存器   | Count Register |
| DX  DH,DL                     | EDX    |        | 数据寄存器   | Data Register  |
| BP                            | EBP    |        | 堆栈基指针   | Base Pointer   |
| SI                            | ESI    |        | 变址寄存器   | Index Register |
| DI                            | EDI    |        | 变址寄存器   | Index Register |
| SP                            | ESP    |        | 堆栈顶指针   | Stack Pointer  |
|                               |        |        |              |                |

函数的返回值默认使用eax寄存器存储返回给上一级函数；



段寄存器

| 名称                   | 名称2        | 作用             |
| ---------------------- | ------------ | ---------------- |
| Code Segment Register  | 代码段寄存器 | 代码段的段值     |
| Data Segment Register  | 数据段寄存器 | 数据段的段值     |
| Extra Segment Register | 附加段寄存器 | 附加数据段的段值 |
| Stack Segment Register | 堆栈段寄存器 | 堆栈段的估值     |
| F Segment Register     | 段寄存器     |                  |
| G Segment Register     | 段寄存器     |                  |
|                        |              |                  |

cpu所取的指令为：CS + EIP；
每个进程都是一个自己的堆栈：SS；



标志寄存器（标识一些状态 ）

| 名称 | 作用 |
| ---- | ---- |
| 暂无 |      |



x86_64寄存器

| 名称 | 作用 |
| ---- | ---- |
| 暂无 |      |

IP变为RIP，即通用寄存器以R为前缀。
增加了一些64位的media and floating-point registers.
增加了一些128位的media registers.



### Instructions

所有指令，如下：

```bash
#-----------------cpu对内存、寄存器的操作方法
movl %eax, %edx     #edx=eax; 寄存器寻址模式（操作的都是寄存器、跟内存不打交道）
movl $0x123, %edx   #edx=0x123; 立即寻址（也跟内存没关系、直接就是给的值）
movl 0x123, %edx    #edx=*(int32_t*)0x123; 直接寻址（直接访问一个指定的内存地址的数据）
movl (%ebx), %edx   #edx=*(int_32*)ebx; 间接寻址；（寄存器中存储的是地址、而非数据）
movl 4(%eax), %edx  #edx=*(int_32*)(ebx+4); 变址寻址；（在间接寻址之时改变寄存器的数据）

#------------------栈操作
pushl %eax  #压栈
	#subl $4, %esp; movl %eax, (%esp)
	#堆栈是向下增长的（向下为低地址）；栈底为EBP,栈顶为ESP,只能操作ESP；
popl %eax  #出栈
	#movl (%esp), %eax; addl $4, %esp;

#---------------------------函数调用相关
call 0x12345  #函数调用
	#pushl %eip(*); movl $0x12345, %eip(*)
	#带(*)是伪指令不能被程序员直接使用，即不能直接改变eip的值，而是通过call/ret之类的指令间接修改；
ret   #return
	#popl %eip(*)
enter #栈上堆个新栈
	#pushl %ebp; movl %esp, %ebp;
leave #撤销栈
	#movl %ebp, %esp; popl %ebp;
	

int
	#save cs:eip/ss:esp/eflags(current) to kernel stack.
	#then load cs:eip(entry of a specific ISR) and ss:esp(point to kernel stack)
SAVE_ALL
	#中断时保存其它信息
RESTORE_ALL
	#与SAVE_ALL相反；
iret
	#pop cs:eip/ss:esp/eflags from kernel stack. (与int动作的相反)
```

指令后缀：b, w, l, q分别代表8、16、32、64位；
操作数：%前缀代表一个寄存器；$前缀代表一个立即数（数值）；单纯0x值代表内存地址；(%..) 寄存器中存储的值是一个地址；
大多数指令（x86）能直接访问内存地址；
内存操作指令如：MOV, PUSH, POP ..



AT&T、Inter两种汇编格式略有不同。
linux内存采用的是AT&T汇编格式；



请看如下Demo

```bash
pushl $8         #把8压栈
movl %esp, %ebp  #ebp指向esp
subl $4, %esp    #esp下移
movl $8, (%esp)  #把8放在esp所指的内存中；

pushl $8
movl %esp, %ebp
pushl $8
```



### Inline Assembly in Linux C

more refer: https://blog.csdn.net/zyllong/article/details/42869577

```cpp
//__asm__(汇编语句: 输出参数: 输入参数: 破坏描述部分);

unsigned int v1=1, v2=2, v3=0; //v3=v1+v2;
asm volatile (
    "movl $0, %%eax\n\t" //清空%eax
    "addl %1, %%eax\n\t" //第1个参数 + %eax；（从输出参数开始计数到输入参数结束，从0开始）
    "addl %2, %%eax\n\t" //第2个参数 + %eax
    "movl %%eax, %0\n\t" //%eax的值赋值到第0个参数，即v3
    : "=m"(v3)  //=m指写入到内存变量
    : "c"(v1), "d"(v2)  //用ecx存储v1，用edx存储v2
)
```

其中如上（=m, c, d）这些都是<font color=red>内嵌汇编的限定符</font>。



## Disassembly

```bash
gcc -S -o tst.s tst.c -m32
	#生成的tst.s中，所有以.开头的内容，都是链接用的辅助信息；（可以删除）
```

源代码，如下

```cpp
int g(int x)
{
    return x + 3;
}
                    
int f(int x)
{
    return g(x);
}    
                    
int main(void)
{
    return f(9) + 1;
} 
```

对应汇编代码，如下

```cpp
g:
    pushl   %ebp
    movl    %esp, %ebp
    movl    8(%ebp), %eax
    addl    $3, %eax
    popl    %ebp
    ret
f:
    pushl   %ebp
    movl    %esp, %ebp
    subl    $4, %esp
    movl    8(%ebp), %eax
    movl    %eax, (%esp)
    call    g
    leave
    ret 
main:
    pushl   %ebp
    movl    %esp, %ebp
    subl    $4, %esp
    movl    $9, (%esp)
    call    f
    addl    $1, %eax
    leave
    ret
```



## How Does The OS Work

操作系统是如何工作的

计算机硬件的三个法宝：
存储程序计算机、函数调用堆栈、中断机制



中断
有了中断之后，系统可以有了多道程序设计的概念；



### Stack

堆栈向低地址增长（ebp在高地址、esp在低地址，%ebp >= %esp），即%esp-4。



堆栈相关寄存器
esp，堆栈指针（stack pointer）
ebp，基址指针（base pointer）
	在c语言中用作记录<font color=red>当前函数调用</font>的基址；



堆栈操作
push，栈顶地址减4字节。（32位）
pop，栈顶地址增4字节。（32位）



----------

**其它关键寄存器**

cs : eip 总是指向下一条的指令地址
顺序执行：总是指向地址连接的下一条指令。
跳转、分支：执行这样的指令的时候，cs : eip的值会根据程序需要被修改。
call：将当前cs:eip的值压栈，cs:eip指向被调用函数的入口地址。
ret：从栈顶弹出原来保存在这里的cs:eip的值，放入cs:eip中。
发生中断了呢？



### Function Call Stack

调用者

```cpp
//...
call target;
	//1, 将eip中下一条指令的地址A保存在栈顶；
	//2, 设置eip指向被调用程序代码开始处；
//...
```

被调用者

```cpp
pushl %ebp
movl %esp, %ebp
	//建立被调用函数的堆栈框架

//do something.
//--被调用者函数体；
//do something.

movl %ebp, %esp
popl %ebp
ret   //将地址A恢复到eip中；
	//拆除被调用者函数的堆栈框架
```



传参（先于call指令被压入栈）、局部变量（会在预留空间内分配）

编译，gcc -g t.c -o t -m32

```cpp
#include<stdio.h>

void p1(char c)
{
    printf("%c\n", c);
}

int p2(int x, int y)
{
    return x + y;
}

int main(void)
{
    char c = 'a';
    int x, y, z;
    x = 1;
    y = 2;
    p1(c);
    z = p2(x, y);
    printf("%d = %d + %d\n", z, x, y);
}
```



反汇编，objdump -S t

```cpp

void p1(char c)
{
 804841c:       55                      push   %ebp
 804841d:       89 e5                   mov    %esp,%ebp
 804841f:       83 ec 28                sub    $0x28,%esp
 8048422:       8b 45 08                mov    0x8(%ebp),%eax
 8048425:       88 45 f4                mov    %al,-0xc(%ebp)
    printf("%c\n", c);
 8048428:       0f be 45 f4             movsbl -0xc(%ebp),%eax
 804842c:       89 44 24 04             mov    %eax,0x4(%esp)
 8048430:       c7 04 24 50 85 04 08    movl   $0x8048550,(%esp)
 8048437:       e8 c4 fe ff ff          call   8048300 <printf@plt>
}
 804843c:       c9                      leave  
 804843d:       c3                      ret    

0804843e <p2>:

int p2(int x, int y)
{
 804843e:       55                      push   %ebp
 804843f:       89 e5                   mov    %esp,%ebp
    return x + y;
 8048441:       8b 45 0c                mov    0xc(%ebp),%eax
 8048444:       8b 55 08                mov    0x8(%ebp),%edx
 8048447:       01 d0                   add    %edx,%eax
}
 8048449:       5d                      pop    %ebp
 804844a:       c3                      ret    

0804844b <main>:

int main(void)
{
 804844b:       55                      push   %ebp
 804844c:       89 e5                   mov    %esp,%ebp
 804844e:       83 e4 f0                and    $0xfffffff0,%esp
 8048451:       83 ec 20                sub    $0x20,%esp  //预留局部变量空间；
    char c = 'a';
 8048454:       c6 44 24 1f 61          movb   $0x61,0x1f(%esp)
    int x, y, z;
    x = 1;
 8048459:       c7 44 24 18 01 00 00    movl   $0x1,0x18(%esp)
 8048460:       00 
    y = 2;
 8048461:       c7 44 24 14 02 00 00    movl   $0x2,0x14(%esp)
 8048468:       00 
    p1(c);
 8048469:       0f be 44 24 1f          movsbl 0x1f(%esp),%eax
 804846e:       89 04 24                mov    %eax,(%esp)
 8048471:       e8 a6 ff ff ff          call   804841c <p1>
    z = p2(x, y);
 8048476:       8b 44 24 14             mov    0x14(%esp),%eax
 804847a:       89 44 24 04             mov    %eax,0x4(%esp)  //将y入栈（未在p2栈中）
 804847e:       8b 44 24 18             mov    0x18(%esp),%eax
 8048482:       89 04 24                mov    %eax,(%esp)     //将x入栈（未在p2栈中）
 8048485:       e8 b4 ff ff ff          call   804843e <p2>
 804848a:       89 44 24 10             mov    %eax,0x10(%esp)  //返回结果eax赋值到z
    printf("%d = %d + %d\n", z, x, y);
 804848e:       8b 44 24 14             mov    0x14(%esp),%eax
 8048492:       89 44 24 0c             mov    %eax,0xc(%esp)
 8048496:       8b 44 24 18             mov    0x18(%esp),%eax
 804849a:       89 44 24 08             mov    %eax,0x8(%esp)
 804849e:       8b 44 24 10             mov    0x10(%esp),%eax
 80484a2:       89 44 24 04             mov    %eax,0x4(%esp)
 80484a6:       c7 04 24 54 85 04 08    movl   $0x8048554,(%esp)
 80484ad:       e8 4e fe ff ff          call   8048300 <printf@plt>
}
 80484b2:       c9                      leave  
 80484b3:       c3                      ret
```



64位Linux安装32位编译环境

```bash
# dpkg -l | grep libc6
ii  libc6:amd64                        2.13-38+deb7u12                  amd64        Embedded GNU C Library: Shared libraries
ii  libc6-dev:amd64                    2.13-38+deb7u12                  amd64        Embedded GNU C Library: Development Libraries and Header Files


# dpkg -l | grep stdc++
ii  libstdc++6:amd64                   4.7.2-5                          amd64        GNU Standard C++ Library v3
ii  libstdc++6-4.7-dev                 4.7.2-5                          amd64        GNU Standard C++ Library v3 (development files)

#dpkg --add-architecture i386
#apt-get update
#apt-get install apt-get install libc6-dev-i386

# dpkg --remove-architecture i38   
# dpkg --print-architecture
```



## MyKernel

### Mission & Vision

复用x86 cpu + linux内核 来架构一个虚拟平台；

没有中断之前cpu执行完一个程序之后才能进行下一个程序的执行，有了中断之后就有了多道程序的理念；
那么从A程序怎么切换到B程序呢？--由cpu和内核代码共同实现了保存现场、恢复现场！

>当一个中断信号发生的时候，cpu把当前的eip, esp, ebp都压到一个叫内核堆栈的另外一个堆栈中，然后把eip指向一个中断处理程序的入口；（即保存现场、执行中断程序）



假设咱们的虚拟平台只跑了一个进程（一直在执行）、并在这个进程的执行过程中有定时的时钟中断，然后这就是操作系统时间片轮转的雏形。

```cpp
//....硬件初始化...

void __init my_start_kernel(void) //OS的入口，启动操作系统
{
    while (1) {
        //.....
    }
}
```

### Build OS Kernel Base MyKernel

进程PCB的定义；

调度器；

操作系统的“两把剑”：<font color=red>中断上下文切换（即保存现场、恢复现场）</font>、<font color=red>进程上下文切换</font>；





## Kernel Code Read

### Directory Struct

源码目录结构如下

```bash
arch/
	#与cpu相关的代码，一般只专注于arch/x86
init/
	#启动相关代码
	#init/main.c
kernel/
	#核心代码
mm/
	#内存管理相关
net/
	#网络相关
fs/
	#文件系统相关
```





## Build Kernel

步骤，如下

```bash
#----------下载内核、编译内核
cd ~/kernel
tar zxvf linux-3.18.6.tar.gz
cd linux-3.18.6
make help
make x64_defconfig
make -j 4   

#--------制作根文件系统
cd ~/kernel
mkdir rootfs
git clone https://github.com/mengning/menu.git
cd menu
gcc -o init linktable.c menu.c test.c -static -lpthread
cd ../rootfs
cp ../menu/init ./
find . | cpio -o -Hnewc | gzip -9 > ../rootfs.img


#-------------启动menuOS系统
cd ~/kernel
qemu -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img
	#Could not initialize SDL(No available video device) - exiting
qemu-system-x86_64 -curses -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img
	#And to exit from that, use ESC + 2 then q + ENTER.
	#-kernel bzImage use 'bzImage' as kernel image
	#-initrd file    use 'file' as initial ram disk
```



gdb调试kernel

```bash
make menuconfig
	#重新配置、编译linux，使之带调试信息
	#kernel hacking -> comile-time checks and compiler options -> compile the kernel with debug info.
make
qemu-system-x86_64 -curses -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img -s -S
	#-S  freeze CPU at startup (use 'c' to start execution)
	#-s  shorthand for -gdb tcp::1234, 可用'-gdb tcp:xx'选项来代替'-s'

gdb
  file linux-3.18.6/vmlinux
  target remote:1234
  break start_kernel
```

make menuconfig错误

```bash
root@bogon:~/Kernel/linux-3.18.6# make menuconfig
  HOSTCC  scripts/kconfig/mconf.o
In file included from scripts/kconfig/mconf.c:23:0:
scripts/kconfig/lxdialog/dialog.h:38:20: fatal error: curses.h: No such file or directory
compilation terminated.
make[1]: *** [scripts/kconfig/mconf.o] Error 1
make: *** [menuconfig] Error 2
#apt-get install libncurses5-dev
#make menuconfig
```





## Kernel Start

### init/main.c

start_kernel, 0号进程

```cpp
asmlinkage __visible void __init start_kernel(void)
{
    trap_init(); //初始化中断向量
    
    rest_init(); //其它初始化
}
```

rest_init()

```cpp
static noinline void __init_refok rest_init(void)
{
    kernel_thread(kernel_init, NULL, CLONE_FS); //kernel_init会初始化1号进程，即init进程
    //...
    cpu_startup_entry(CPUHP_ONLINE); //0号进程在这个函数里面while(1)
}
```



### sched/idle.c 

cpu_startup_entry

```cpp
void cpu_startup_entry(enum cpuhp_state state)
{
    cpu_idle_loop(); //这是个while(1)，即0号进程；
    //...
}
```



cpu_idle_loop

```cpp
static void cpu_idle_loop(void)
{
    while (1) {
        ;
    }
}
```





## User/Kernel Mode, Interrupt

### User/Kernel Mode

一般现代CPU都有几种不同的指令执行级别
在高执行级别下，代码可以执行<font color=red>特权指令</font>，访问<font color=red>任意的物理地址</font>，这种CPU执行级别就对应着内核态；
在低执行级别下，代码的掌控范围受到限制，只能在对应级别允许的范围内活动；

> 例如，inter x86 cpu有4种不同的执行级别0-3，linux只使用了其中0级、3级来分别表示内核态、用户态；



那怎么在代码中区分内核态、用户态呢？
cs寄存器的最低两位表明了当前代码的特权级；
特权模式下cs:eip在32位机上的寻址空间是4G（逻辑地址，而非物理地址，由MMU将物理地址映射成逻辑地址）
用户态则只能访问4G中的某一部分；

> 一般来说在linux下，0xc0000000以上的地址只能在内核态下访问；
> 0x00000000 - 0xbfffffff 之间的地址空间，用户态、内核态都可以访问；



### Interrupt

中断是用户态进入内核态的主要方式。

> 在用户态执行过程中，有硬件中断触发，进入中断处理程序；
> 在用户态执行过程中，程序进行系统调用，陷入到内核态中；（即trap）



中断指令/中断向量/INT指令/中断信号发生之后，就进程中断处理程序。

**中断的处理过程为**

1. 关中断（在此中断处理完成前，不处理其它中断）

2. 保护现场：保存需要用到的寄存器的数据。

   对应SAVE_ALL宏指令。

3. 执行中断服务程序

4. 恢复现场：退出中断程序，恢复保存寄存器的数据。

   对应RESTORE_ALL指令。

1. 开中断



**中断的处理过程对应代码**（参照：x86 Assembly/Instructions）

1. INT
2. SAVE_ALL
3. RESTORE_ALL
4. IRET







## System Call Summarize

linux上系统调用的三层皮。-----------API, system_call, sys_xx

系统调用：OS为用户态进程与硬件设备进行交互提供了一组接口。

> 将用户从底层的硬件编程中解放出来；
> 提高系统的安全性；
> 使用户程序具有可移植性；（与硬件解耦）



应用程序编程接口(Application Program Interface)  VS 系统调用？

> api只是一个函数定义；
> 系统调用通过软中断（TRAP）向内核发出一个明确的请求；

那么libC呢？

> libc库定义的一些API引用了封装例程（wrapper routine, 唯一目的就是发布系统调用）。
> 一般每个系统调用对应一个封装例程；库再用这些封装例程定义出给用户的API；



linux上系统调用需要传入什么？

> 系统调用号；----存储于%eax
> 系统调用的参数；---寄存器%ebx, %ecx, %edx, %esi, %edi, %ebp传参；
> 	每个参数长度不能超过寄存器长度；
> 	如果参数个数超过6个，那么把某一个寄存器的值当作指针（把其余参数放这里）；



### With API or ASM

使用库函数api、c中嵌入汇编来触发同一个系统调用 

with api

```cpp
#include <time.h>
time_t tt;
tt = time(NULL);
```

with asm

```cpp
#include <time.h>
time_t tt;
asm volatile (
	"mov $0, %%ebx\n\t"    //传参
    "mov $0xd, %%eax\n\t"  //传系统调用号
    "int $80\n\t"    //系统调用触发
    "mov %%eax, %0\n\t"   //得到系统调用结果
    : "=m"(tt)
)
```



### How SystemCall In Kernel

系统调用在内核代码中的处理过程



#### Interrupt Vector Init

中断向量system_call的初始化过程；

init/main.c

```cpp
start_kernel()
{
    trap_init();
}

```

arch/x86/kernel/traps.c

```cpp
void __init trap_init(void)
{
	#ifdef CONFIG_X86_32
		set_system_trap_gate(SYSCALL_VECTOR, &system_call);
    	set_bit(SYSCALL_VECTOR, used_vectors);
	#endif
}
```

linux-3.18.6/arch/x86/kernel/entry_32.S

```bash
ENTRY(system_call)
    RING0_INT_FRAME         # can't unwind into user space anyway
    ASM_CLAC
    pushl_cfi %eax          # save orig_eax
    SAVE_ALL
    GET_THREAD_INFO(%ebp)
                    # system call tracing in operation / emulation
    testl $_TIF_WORK_SYSCALL_ENTRY,TI_flags(%ebp)
    jnz syscall_trace_entry
    cmpl $(NR_syscalls), %eax 
    jae syscall_badsys
syscall_call:
    call *sys_call_table(,%eax,4)  #系统调用
syscall_after_call:
    movl %eax,PT_EAX(%esp)      # store the return value
syscall_exit:
    LOCKDEP_SYS_EXIT
    DISABLE_INTERRUPTS(CLBR_ANY)    # make sure we don't miss an interrupt
                    # setting need_resched or sigpending
                    # between sampling and the iret
    TRACE_IRQS_OFF
    movl TI_flags(%ebp), %ecx 
    testl $_TIF_ALLWORK_MASK, %ecx  # current->work
    jne syscall_exit_work
           
restore_all:
    TRACE_IRQS_IRET
#...
```





## GDB The sys_time

使用gdb跟踪系统调用内核函数sys_time











