[TOC]



## ReadMe

讨论介绍ceph；

Ceph提供了一个统一存储的方案，包含
	对象存储
	块存储
	文件存储



## 对象存储

对象存储提供Key-Value（简称K/V）方式的RESTful数据读写接口，并且常以网络服务的形式提供数据的访问。如AWS S3、Swift。

对象存储，中的“对象”（Object）和我们平时说的文件类似，如果我们把一个文件传到对象存储系统里面存起来，就叫做一个对象。 



### 对象存储 VS 文件系统

主要区别存取的接口、扁平的数据组织结构。 

#### 存储接口

对于大多数文件系统来说，尤其是POSIX兼容的文件系统，提供open、close、read、write和lseek等接口。 

对象存储的接口是REST风格的，通常是基于HTTP协议的RESTful Web API，通过HTTP请求中的PUT和GET等操作进行文件的上传即写入和下载即读取，通过DELETE操作删除文件。 

最大的区别：我觉得应该是对象存储不支持随机位置的读取、写入。



#### 扁平的数据组织结构

对象存储是没有嵌套的文件夹，而是采用扁平的数据组织结构，往往是两层或者三层。
用户把自己的存储空间划分为若干容器Bucket，然后往Bucket里面放众多Object；
Bucket内不能嵌套Bucket；
每个Bucket、Object均有唯一标识，用户可以通过该标识来访问对应的Bucket, Object，即K/V访问方式；

