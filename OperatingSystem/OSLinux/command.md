[TOC]



## ReadMe

linux指令集



## apt

debian 7.8用163的源，安装g++

```bash
#apt-get install g++
The following packages have unmet dependencies:
 g++ : Depends: g++-4.7 (>= 4.7.2-1~) but it is not going to be installed
E: Unable to correct problems, you have held broken packages.

#-----------换成aliyun的源之后没有问题；
```

中途试图重装libc, 即apt-get install libc=xxx，导致系统崩溃，重新申请了台虚拟机，还是不要轻易动这些底层库！



