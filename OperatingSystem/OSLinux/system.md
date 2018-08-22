[TOC]



## ReadMe

从全局入手，了解深入linux操作系统（当然深入部分只能作成索引，链接到对应目录章节）；



## CPU架构

cpu的架构跟编程中的大小端、一些代码的深度优化（只对某些cpu有效，代码不可移植性）有关系；

http://blog.csdn.net/djinglan/article/details/8591131

查看cpu的字长、架构；

```bash
root@deboost# getconf LONG_BIT
64

root@deboost# uname -m  //print the machine hardware name
x86_64
	
root@deboost# arch
x86_64
	
root@deboost# uname -r //print the kernel release
3.2.0-4-amd64
```



## 内核模块化

查看、插入、删除模块

```bash
# lsmod
```



## 安全相关

### netfilter

提供了内核级的包过滤技术、早期支撑起了简单的防火墙功能（2、3、4层包过滤）及一些NAT功能；

Refer: [netfileter文章](netfilter.md)



### netfilter之iptable

netfilet只是内核中的包过滤框架，但包过滤的规则得靠管理员手动输入，所以有了iptable, iptalbes.

Refer: [netfileter文章中相关章节](netfilter.md#iptables)



### selinux





## 容器支持

当下流行的应用容器，如docker之类底层依赖于linux的这些feature.
cgroup + namespace + union filesystem.

### cgroup

control group是将任意进程进行分组化管理的Linux内核功能，提供将进程进行分组化管理的功能和接口。



### namespace

namespace

| Namespace | 系统调用参数  | 隔离内容                   |
| --------- | ------------- | -------------------------- |
| UTS       | CLONE_NEWUTS  | 主机名与域名               |
| IPC       | CLONE_NEWIPC  | 信号量、消息队列和共享内存 |
| PID       | CLONE_NEWPID  | 进程编号                   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等等 |
| Mount     | CLONE_NEWNS   | 挂载点（文件系统）         |
| User      | CLONE_NEWUSER | 用户和用户组               |



### union filesystem

union文件系统



## 内核

内核这东西应该力致极简、极快、极稳定。

### 编译、替换、启动

