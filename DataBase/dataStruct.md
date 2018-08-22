[TOC]



## ReadMe

讨论常用的数据结构；



## hash table

hashtable ---用意是在常数时间内提供基本操作。

可以提供任何有名项的存取操作和删除操作，由于操作对象是有名项，故可被视为一种字典结构。  
主要思想是通过hash函数，把对象映射到一个较小的容器里面，并且保证时间复杂度。映射到较小容器很可能出现碰撞问题，解决的方法常见的有：线性探测，二次探测，开链法。





### hash碰撞

#### 线性探测

#### 二次探测

#### 开链法

STL中的hash_map/set利用vector来当容器，采用开链法来解决冲突，从而实现hashtable.
hashtable只能处理char,int,short等类型，不能处理string,double,float类型，想要处理的话必须自己加hash function。



