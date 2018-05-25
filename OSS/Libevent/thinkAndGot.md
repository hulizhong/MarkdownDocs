
## ReadMe
What u think is What u got.

## base
base\_loop跑在A线程；
所有注册到base上的event callback都是在A线程上跑的吧；

多线程支持
> 要调用特定的evthread\_use..来声明，向libevent添加锁、条件变量；  
> 当然你也可以自己注册自己的锁、条件变量；  


## event

## timer event
可提供比毫秒级更精确的计时器；

它的小根堆是怎么管理的？
> 


## 一个timeEvent的生命

## io event
存储空间
> 用到了hash table；亦可同于sinal的存储方式；


如何变成active的？
> 


## 一个ioEvent的生命



## signal event
存储空间
> evsignal\* []
> 存储用到了二维数组，以signalNo作为第一维数组的索引；
> 其中evsignal是记录了一个链表头；链表的串接是靠event自身的结构字段进行设计的；

如何变成active的？
> 


## 一个signalEvent的生命

## list操作的宏
把所有list操作的提练成了宏，传入name, type两个参数；而上、下的链接指针则实现到了结构体中；
这样宏的适用范围就大了；只是读代码麻烦了；

## HashTable操作的宏
同List宏，提练成了宏，传入参数name, type；
