
## ReadMe
收录指令集


## find something
### grep

```bash
grep key ./ -rn --color --exclude-dir dir1
	#在当前目录下查找key，但是排除dir1目录
```

### find

```bash
find ./ -name key.txt
	#在当前目录下查找key.txt
```



## network

### ss

socket statics.

```bash
ss -lnt
	#t tcp
	#l 监听状态
	#n 不解析服务名

-h：显示帮助信息；
-V：显示指令版本信息；
-n：不解析服务名称，以数字方式显示；
-a：显示所有的套接字；
-l：显示处于监听状态的套接字；
-o：显示计时器信息；
-m：显示套接字的内存使用情况；
-p：显示使用套接字的进程信息；
-i：显示内部的TCP信息；
-4：只显示ipv4的套接字；
-6：只显示ipv6的套接字；
-t：只显示tcp套接字；
-u：只显示udp套接字；
-d：只显示DCCP套接字；
-w：仅显示RAW套接字；
-x：仅显示UNIX域套接字。
```



`ss -lnt`  查看socket的连接队列使用情况

```bash
# ss -lnt
Recv-Q Send-Q Local Address:Port  Peer Address:Port 
0        50               *:3306             *:* 
```

以上send-q=50为 port.3306监听socket的accept queue length.
以上recv-q=0为accept queue中已使用的个数。







### netstat

```bash
netstat -natp
```



`netstat -s` 查看socket的连接队列溢出情况

```bash
# netstat -s | egrep "listen|LISTEN" 
1641906 times the listen queue of a socket overflowed
1641906 SYNs to LISTEN sockets ignored
```

以上，overflowed表示全连接队列溢出次数。
以上，socket ignored表示半连接队列溢出次数。







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



## Use Case

### 踢人下线

who 当前登录用户  
w [user] 登录用户行为  
pkill -u user 踢除user这个用户和他的所有开启的程序  
更为安全的作法： ps -ef| grep pts/0;  kill -9 pid  



