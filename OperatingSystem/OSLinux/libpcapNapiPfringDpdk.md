[toc]



# ReadMe

我的问题：

- 各种包捕获技术sk_buffer的拷贝次数？
- 如何查看一台Linux上网卡驱动、及包是怎么到app层的？
- libpcap与eBPF是怎么结合的？是不是有个什么extended libpcap??



旨在说明在linux下台下的包捕获取技术。

- packet capture技术中包的路径，跟linux平时包的路径肯定是不一致的，pcap的目标是尽量高效（丢包少）、低代价（cpu消耗率）的拿到包。
- 零拷贝：通过尽量避免拷贝操作（内核与内核之间、内核与用户态间、io设备与主存之间）来缓解 CPU 的压力。如`mmap(), sendfile(), splice()`.
- 。。



# linux network packet capture

Linux下包捕获，即从网卡驱动处如何将包收到app中、或者怎么把app待发送的数据分发到网卡传输队列中去。
有Linux原始的协议栈、libpcap, napi, pfring, dpdk...众多方法，概要都是如何少让cpu, 内核去处理数据包，让数据最大效率来到app层。

| 技术           | 收包、技术特征             | 包总拷贝次数 | 适用场景 | 支持网卡型号                                    |
| -------------- | -------------------------- | ------------ | -------- | ----------------------------------------------- |
| linux protocol | 中断、拷贝                 | 3？          |          |                                                 |
| libpcap        | 减少拷贝次数               |              |          |                                                 |
| napi           | 轮询收包                   |              |          |                                                 |
| pfring         | 零拷贝收包                 |              |          | broadcom/*; <br />intel/e1000,e1000e,igb,ixgbe; |
| pfring-dna     |                            |              |          |                                                 |
| dpdk           | 数据面、控制面的概念及实现 |              |          |                                                 |



## protocol stack

原生Linux kernel protocol stack的收包流程，<font color=red>这个流程肯定跟PCAP(PacketCapture)流程是有区别的，pcap基本上不走原来的linux protocol stack</font>，但平时网卡的数据怎么到内核呢？应该还是走中断收包那一套吧？

linux中网络子模块：

- 网卡驱动：负责收发包；packet网卡、内核间的拷贝？
- 协议栈：skb的封包、解包、并一些协议控制。
- cpu：参与包的拷贝、运行协议栈对包的控制。



### packet in driver phase

目前基本都是napi模式（中断+轮询），老的一些网卡、及驱动为了适合napi架构，也增加了个伪net_device.

https://www.cnblogs.com/4a8a08f09d37b73795649038408b5f33/p/11475249.html



### packet in protocol phase

https://zhuanlan.zhihu.com/p/137215216





# libpcap

libpcap技术特征：

- 数据链路层增加个旁路处理，将包拷贝发送到BPF中，符合规则的再往上送至libpcap应用层。
- 新增一种socket类型pf_pcap。基于BPF数据包过滤体系。
- 应用：wireshark, tcpdump, sniffer, snort.
- 慢的原因：
    - cpu频繁处于中断状态，可算下100M网卡，即100Mbit/s=100Mbps=100 000 000bps，以太网下最大包1500Bytes,最小包46Bytes，那么中断次数为100,000,000/(1500x8)=8333.3次 - 100,000,000/(46x8)=271739次。
    - 数据包被多次拷贝，从网卡驱动到内核、从内核到用户态。
- 其它补充：
    - 如果一个网卡适配器被PF_RING套接字利用系统调用bind()绑定，这个网卡只能用于只读直到套接字销毁。
    - PF_RING的发包需要使用DNA技术，而此功能需要费用。如果不使用DNA,发包将会走协议栈。





## BPF(cBPF), eBPF

BPF是包过滤系统，即libpcap框架中只有满足BPF的包才会被往上送。

Berkeley Packet Filter，

> 所覆盖的功能范围很简单，就是网络监控和 seccomp 两块，数据接口设计的粗放。
> 代码实现为一个文件(net/core/filter.c)。

extend Berkeley Packet Filter，kernel3.17加入的新特性。 ----<font color=blue>监控、跟踪框架</font>

> 所覆盖的功能要广的多，性能调优、内核监控、流量控制什么的，数据接口的多样性设计。
> 代码实现为一个目录(kernel/bpf)。



编写BPF程序

> BPF Compiler Collection(BCC)，BCC 是一个 python 库，但是其中有很大一部分的实现是基于 C 和 C++的，python实现了对 BCC 应用层接口的封装。
> https://github.com/iovisor/bcc



# NAPI

<font color=red>解决了cpu长时间陷入中断处理的问题</font>，NewAPI技术特征

- 用轮询读取数据包来替换中断读取数据包。
    - 即中断唤醒数据接收的服务程序，然后接收服务关中断、poll数据、开中断，。。。等待下次唤醒。
- 先决条件：
    - 网卡设备用DirectMemoryAccess硬件、支持DMA的环形输入队列即ring_dma.
    - 。。。
- 缺点：app不能及时的处理收到的每个数据包；不适用于大的数据包。
- 适用的流量场景：小包+大流量。





# pfring

<font color=red>解决了cpu大把时间消耗在数据包拷贝上的问题</font>，pf_ring技术特点

- 将网卡接收到的数据存储在一个环状缓存ring中，ring有读、写接口，app与ring之间的读写是用mmap实现。
    - 数据从网卡到ring走NAPI Polling需要cpu参与，DNA模式依赖NIC NPU来拷贝。
- 新增一种socket类型pf_ring, 对应有一个pfring环缓冲区（读、写索引）。
- 三种工作模式0（走napi流程）、1（在原始protocol stack的基础上copy一份走pfring）、2（走pfring流程，不过Kernel）.
- 缺点：不能充分利用网卡的新特性（多接收队列、多队列之间的负载），几种改进
    - TNAPI, 
    - DNA(Direct NIC Access)，将网卡的内存、缓存映射到userland，如此不走NAPI，靠NIC NPU(NetworkProcessUnit)把数据包从网卡拷贝到DMA ring中（<font color=red>比non-dna pfring少拷贝一次</font>），解放cpu。----在超大流量下cpu的利用率也很低，但dna不能进行包过滤，直接全收了。
- 适用的流量场景：



与正常抓包对比，示意图如下：

<img src=img/pfringVSnormalPCap.png style="zoom:60%" align=left />

pfring走的关键路径，如下：

<img src=img/pfringPacketFlow.webp align=left style="zoom:100%" />









# dpdk

