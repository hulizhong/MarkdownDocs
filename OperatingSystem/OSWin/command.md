[TOC]

## ReadMe
win平台下的指令集；

## Commands

### About OS

查看os版本

```bash
winver
msinfo32
```





### About Network

netstat, 网络连接状态

```bash
netstat -nato | findstr 445
	#-n 以数字形式显示地址和端口号
	#-a 显示所有连接和监听端口
	#-t 显示当前连接卸载状态
	#-o 显示与每个连接相关的所属进程 ID

```



nbtstat, 早期的netbios名称解析系统

```bash
nbtstat -a 172.22.48.100
	#-a 显示远程计算机的NetBIOS名称表，即在内网中根据ip查找用户
	#-n 显示本机的netbios缓存
```



### About Process

tasklist, 进程列表

```bash
tasklist /fi "imagename eq niginx.exe"
tasklist /fi "pid eq 4"
	#/svc 显示每个进程中的服务
	#/v 显示详述信息
	#/fi 指定筛选器
	#/m 显示每个进程加载的所有模块
	#/m moduleName 列出调用指定的DLL模块的所有进程
```



taskkill, 杀进程

```bash
taskkill /im xx.exe
taskkill /pid 13116
```



---

sc, 服务

```bash
sc create xx
	#安装服务
sc delete xx
	#卸载服务
sc query
	#查询服务，比net start更详细
sc stop secgatorvfs
	#停止服务，返回时能不能保证服务已经stop掉了（或者正在stopping）；
sc start xx
	#启动服务

sc config xx start=demand
	#配置服务为手动启动
sc config xx start=auto
	#配置服务为自动启动
sc config xx start=disabled
	#配置服务为禁用
```



net, 服务

```bash
net start
	#查看运行的服务
net start xx
	#启动服务
net stop xx
	#停止服务，返回时能保证服务已经stop掉了；(net stop xx; net start xx;这样用于bat中)
xx -uninstall
	#卸载服务（卸载前先停止服务）
```






## API
win Sleep(ms);
linx sleep(s);





## Other

aa.lib 是编译用的；

aa.dll 是运行的时候用的，其所有目录加到环境变量PATH中，以方便依赖它的exe能正常加载到它；

---



