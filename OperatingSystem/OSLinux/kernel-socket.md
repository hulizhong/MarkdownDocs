[TOC]



## ReadMe

about socket(tcp/ip) in kernel.





## Others

-------------kenel socket.

4.   最大TCB 数量   MaxFreeTcbs系统为每个TCP 连接分配一个TCP 控制块(TCP control block or TCB)，这个控制块用于缓存TCP连接的一些参数，每个TCB需要分配 0.5 KB的pagepool 和 0.5KB 的Non-pagepool，也就说，每个TCP连接会占用 1KB 的系统内存。非Server版本，MaxFreeTcbs 的默认值为1000 （64M 以上物理内存）Server 版本，这个的默认值为 2000。也就是说，默认情况下，Server 版本最多同时可以建立并保持2000个TCP 连接。

5.   最大TCB Hash table 数量   MaxHashTableSize TCB 是通过Hash table 来管理的。这个值指明分配 pagepool 内存的数量，也就是说，如果MaxFreeTcbs = 1000 , 则 pagepool 的内存数量为 500KB那么 MaxHashTableSize 应大于 500 才行。这个数量越大，则Hash table 的冗余度就越高，每次分配和查找 TCP  连接用时就越少。这个值必须是2的幂，且最大为65536.

https://blog.csdn.net/wangpeihuixyz/article/details/40558009



--linux协议栈 各层分析

https://blog.csdn.net/wangpeihuixyz/article/details/29917803#comments





