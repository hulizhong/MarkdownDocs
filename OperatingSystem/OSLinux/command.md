
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

### curl

```bash
curl -X POST url [-d ""]   #需要转义data中"，而用'则可避免
curl -X POST url [--data ""]

curl -X POST url --data @filename
curl -X POST url -d @filename

curl -X POST url -T filename  #相对直接--data来说：会有个100-continue，并不会判断文件内容类型；
curl -X POST url -T "{file1,file2}"  #多个文件
```

multipart怎么传？rwhy.

```bash
curl  -F "keyName=@fileName;type=text/plain" url  multipart/form-data数据上传
```

认证

```bash
curl -u username:password url

curl -k https://www.baidu.com
	-k, --insecure      Allow connections to SSL sites without certs, 忽略https认证；
```

其它

```bash
--limit-rate 1000B  #限速

-C #断点续传

-o newName, -O  #下载文件重命名

-x proxysever:proxyport  #代理

-H 'key:value'
	#加头, (Wrong: -H '"reqid":123',  Right: -H 'reqid:123')
	#多个头分开加；

-v #显示详细信息，包含连接建立过程，响应头。
```



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



### lsof

`lsof | more` 列出所有打开的文件

> COMMAND     PID   TID          USER   FD      TYPE             DEVICE  SIZE/OFF       NODE NAME
> init          1                root  cwd       DIR              254,0      4096          2 /

`lsof -c apache` 列出apache程序打开的文件。（apache不需要写完，只需要写对打头）

> COMMAND    PID USER   FD   TYPE             DEVICE SIZE/OFF     NODE NAME



```bash
lsof -i tcp/udp
lsof -i tcp:22
lsof -i :22  #默认是tcp的

lsof -p pid

lsof -c appname

lsof -u username

lsof filename   #谁正在用这个文件。
```





### nc 

```bash
# nc ip port
启动ip, port的tcp服务
```

### netstat

```bash
#netstat -natp
Proto Recv-Q  Send-Q  LocalAddress  ForeignAddress    State     PID/Program name
```



`netstat -s` 查看socket的连接队列溢出情况

```bash
# netstat -s | egrep "listen|LISTEN" 
1641906 times the listen queue of a socket overflowed
1641906 SYNs to LISTEN sockets ignored
```

以上，overflowed表示全连接队列溢出次数。
以上，socket ignored表示半连接队列溢出次数。



**连接状态怎么看**？---一定要配合进程id/name进行解读。

> tcp       70      0 172.22.40.1:58903       172.22.40.1:9443        CLOSE_WAIT  3782/java

java进程，利用本地40.1:58901连接9443端口，处于被动关闭的close_wait状态！



## openssl

生成 private/public key 对

```bash
openssl genrsa -out private.key 1024
openssl rsa -in private.key -pubout -out public.key
```

用s_server, s_client测试连接（连接建立之后传数据）

```bash
openssl s_server -accept 12345 -CAfile ca.crt -cert server.pem -key server.key -Verify 5 
	# Verify是强制认证client，且client证书签发机构的级数；
openssl s_client -connect 127.0.0.1:12345 -CAfile ca.crt -cert client.pem -key client.key

#还可以带个参数：-pass file:password.txt 这是什么意思？？？
```

校验证书是否是此CA机构签发

```bash
#openssl verify -CApath CApath alice\demoCA\cacert.pem
openssl verify -CAfile ca.pem  untrust.pem
```

读取x509证书的内容

```bash
openssl x509 -in input.crt -noout -text
```




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



### patch包

diff

- -N 使生成的patch能正确处理新增、删除的文件。
- -u 对比模式Unified，相对默认的模式生成的patch文件会小一点。
- -a 把所有文件当txt文件处理。
- -r 递归处理文件。

`diff -Nuar srcDir destDir > add.patch` 生成patch文件，如下格式：

```bash
diff -Nuar hehe/xx.cpp hehenew/xx.cpp  #注意：前后关系 + 目录级别关系
--- hehe/xx.cpp 2018-11-03 01:12:48.849856628 -0500   #这是patch前，即源。
+++ hehenew/xx.cpp  2018-11-03 01:13:42.233856652 -0500  #这是patch后，即目标。
...
```

patch

- -R反向操作。
- -pNum 从哪一级目录查找目标文件(夹)，-p0从当前目录查找。
  - <font color=blue>这个目录级别是指patch中的文件的目录结构！！！</font>
- -d dir 在处理之前跳转到该目录下。

```bash
#ls 
hehe/ add.patch

#patch -p0 < add.patch   给当前目录下olddir打patch
#patch -Rp0 < add.patch  撤销打包

#cd hehe
#ls 
xx.h xx.cpp
#patch -p1 < add.patch  忽略hehe/目录，向目标文件打patch.
```





