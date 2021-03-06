[TOC]



## ReadMe



## Software Engineering

软件工程相关知识；

**可行性研究和计划阶段，需求分析阶段，设计阶段，实现阶段，测试阶段，运行和维护阶段。**

用户需求报告
需求文档 ---软件需求说明书

设计的目标：逐步消化、细化需求，用程序的逻辑来表达文字性的描述。
概要设计 ---概要设计说明书。
​	<font color=red>功能说明书是否是这个阶段的输出</font>？ ---我觉得应该是替代概要设计一类的文档。
详细设计 ---详细设计说明书。
接口设计 ---接口设计说明书，也可以内置在详细设计说明书。



**如何对项目计划进行跟踪？**
每周对计划完成情况进行审核、评估，及时进行反馈和调整。









## Software Testing

### About concepts.

覆盖率、
测试报告、
测试过程：分支测试》单元测试》集成测试》系统测试



-------------

**测试过程**

0. Pre Testing.
   审评测试：对功能说明文档、设计说明文档进行检查。在需求、设计阶段进行。---没有检查需求文档。
   编写Test Case：测试用例针对某一功能而编制的一组输入、限定条件、预期结果。在产品编译阶段进行编写。

1. Unit Testing.  
   目标：检验程序最小单元（接口、数据结构、边界、覆盖、逻辑）有无错误。--由开发人员完成。
   时机：一个子模块、主要功能的函数实现后。
   Notice：包含<font color=red>分支测试、功能测试</font>。

2. Integration Testing.  
   目标：检验组成系统的模块接口有无错误；  ---由开发人员完成。
   时机：主要的单元测试完成后。

3. System Testing. 
   目标：检验系统与用户需求是否吻合；
   时机：集成测试之后。
4. Acceptance验收 Testing.
   目标：代客户验收（系统是否符合事先约定的验收标准）签字。
   时机：系统测试完成后，在项目组看来开发、测试工作已完成，可以交付使用。
5. Regression回归 Testing.
   目标：验证程序修改、或者新版本，以前正确的功能和其它指标仍旧正确。
   时机：每次错误修改之后、或者版本更新之后。---穿插整个System Testing过程。
6. Defect缺陷 Tracing跟踪
   目标：确保所有发现的错误被正确记录、分发、评估、关闭、统计。
   时机：从错误发现开始到错误关闭为止，每次错误状态修改之后。 ---穿插整个Sysytem Test过程。





## Pressure Test

压力测试是评估系统处于或超过预期负载时系统的运行情况。压力测试的关注点在于系统在峰值负载或超出最大载荷情况下的处理能力。在压力级别逐渐增加时，系统性能应该按照预期缓慢下降，但是不应该崩溃。压力测试还可以发现系统崩溃的临界点，从而发现系统中的薄弱环节。



压力测试 VS 性能测试？

> 是什么？

windows平台上进行压力测试时，注意查看`cpu, mem, disk io, network io`的表现。

> 其中CPU的显示方法，最好调成`逻辑处理器`模式，对单独的核进行观察，防止压力只落在一个核上。





## Performance Test

性能测试主要评价系统或组件的性能是否和具体的性能需求一致，例如：对访问速度的性能需求或对内存使用情况的需求。特定性能测试的关注点在于组件或系统在规定的时间内和特定的条件下响应用户或系统输入的能力。

不同的性能的度量方法取决于不同的被测对象。对于一个单独软件组件，其性能可以根据CPU主频来判定。而带客户端的系统，其性能则要根据系统处理特定用户请求的响应时间来判定。对于那些由多种组件（如客户端、服务器、数据库）构成的系统，则要进行各组件之间的性能测试。



performance testing 性能
load testing 负载
stress testing 压力
concurrency testing 并发
reliability testing 可靠性

承受多少并发；
​	并发用户和并发线程其实是同一个概念，只是在不同的性能测试工具中其叫法不同而已。
TPS transactions per second是多少；
响应时间是否在接受的范围内；



### System Throughput

影响系统吞吐量几个重要参数

> QPS（TPS）：每秒钟request/事务数量；
> 并发数：系统同时处理的request/事务数；
> 响应时间：一般取平均响应时间；

由QPS（TPS）、并发数两个因素决定，因为：QPS（TPS）= 并发数/平均响应时间



## Performance Optimize

性能优化的意义
在不追加硬件资源的前提下，通过更改系统配置/代码，来满足性能需求，最大化挖掘系统产出。

[refer c++ program optimize](#C++ Program Optimize)



## C++ Program Optimize

想要优化性能，首先需要定位到性能差的代码段，然后才能就不同的场景进行针对性的优化；

### Trouble shoot

如何快速的定位性能差或者执行时间较长的代码段；

> 最有效的方法就是量化，用时间差去量化。比如一个transaction走下来，哪个步骤最耗时间。

#### timeElapse

实现一个工具类，可达到以下效果

```cpp
TimeElapse t;
t.start();
//logic process code.
t.stop();
t.getMs(); //获得start,stop之间的毫秒数；
t.getUs(); //获得start,stop之间的微秒数；
```



linux下

```cpp
#include <time.h>
int clock_gettime(clockid_t clk_id, struct timespec *tp);
	//CLOCK_REALTIME   系统实时时间,随系统实时时间改变而改变
	//CLOCK_MONOTONIC  从系统启动这一刻起开始计时,不受系统时间被用户改变的影响
	//CLOCK_PROCESS_CPUTIME_ID 本进程到当前代码系统CPU花费的时间
	//CLOCK_THREAD_CPUTIME_ID  本线程到当前代码系统CPU花费的时间

struct timespec {
	time_t   tv_sec;        /* seconds */
	long     tv_nsec;       /* nanoseconds 纳秒*/
};
// 1s = 1,000ms毫 = 1,000,000us微 = 1,000,000,000ns纳 = 1,000,000,000,000ps皮
```





### Analysis Tools

#### strace, dtruss

查看系统调用；

#### tcpdump

网络程序、client, server之间数据传输的时间、大小、质量等情况。

如tcp client主要时间卡在recv()调用上，那么可用tcpdump验证下，server端回传response的时间是否与recv()的时间对的上；

> 这种场景主要卡在client对socket io的处理上、或者server端对transaction的处理太慢；





### Check Point

一般的检查点、或者怀疑点有，有如下：

<font color=red>线程过多</font>



---

<font color=red>大量的日志</font>



----



