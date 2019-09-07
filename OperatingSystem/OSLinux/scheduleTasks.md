[TOC]



## ReadMe

研究、探索linux下的计划任务，包括开机启动、定时任务；



## 开机自启

linux开机分几个模式，所以程序开机自启也跟各种模式相关；





## 定时任务

定时执行任务这块linux自带了crontab，针对每个用户可以添加自己的一系列定时、周期性的任务；

### crontab

用contab指令编写出的任务配置位于

```bash
-rw------- 1 root root 276 Jul 17 13:50 /var/spool/cron/crontabs/root
	#注意文件权限；
	#注意文件名即为用户名；
```

指令语法

```bash
# crontab -u user -operation
	# -u 如果不指定用户、那么默认为当前登录用户；
	# operation: -l 查看、-r 删除、-e 编辑
```

任务声明格式

```bash
*  *  * *  *  command-to-be-executed
分 时 日 月 星期

*/n    每隔n  * == */1
n1-n2  n1到n2这段期间

* 6-23/1 * * * /usr/bin/python xx.py
	为什么会每分钟执行呢？ ---因为第1个写的是星号；
```



#### 与邮件通知配合

如何让crontab与邮件通知配合呢？

```bash
cron[17138]: Please install an MTA on this system if you want to use sendmail!  
CRON[17187]: (root) MAIL (mailed 688 bytes of output but got status 0x00ff from MTA#012)  
```

rabin todo. 好像是有办法，但得需要在宿主机上装个MTA软件；（但是装个发送邮件的客户端不行吗？为何要装server.）

```bash
MAILTO=""  #最开始加个这个就行了，crontab脚本中。
```


