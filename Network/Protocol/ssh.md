[TOC]



## ReadMe

ssh协议相关内容；



## SSH

SSH（Secure SHell protocol），安全外壳协议（SSH）是一种在不安全网络上提供安全远程登录及其它安全网络服务的协议。
> Openssh实现了ssh 1.0，2.0版本；



### sshd_config

cat /etc/ssh/sshd_config

```bash
 HostKey /etc/ssh/ssh_host_key # SSH version 1 使用的私钥
 HostKey /etc/ssh/ssh_host_rsa_key # SSH version 2 使用的 RSA 私钥
 HostKey /etc/ssh/ssh_host_dsa_key # SSH version 2 使用的 DSA 私钥
 PermitRootLogin yes
 	#是否允许root进行远程ssh连接。（否则只能本地console登录）
 StrictModes yes
 	#是否让 sshd 去检查用户家目录或相关文件的权限数据  （一般都是’只有属主有权限’，即600）；
 
 PubkeyAuthentication yes
 	#登录验证，基于密钥对验证。
 AuthorizedKeysFile .ssh/authorized_keys
 	#登录用户的公钥存放位置；

 PasswordAuthentication yes
 	#登录验证，基于密码验证；
 PermitEmptyPasswords no #否允许以空的密码登入
 
 RhostsAuthentication no #系统不使用 .rhosts认证
 IgnoreRhosts yes #是否取消使用 ~/.ssh/.rhosts 来做为认证
 RhostsRSAAuthentication no #专门给 version 1 用的，使用 rhosts 文件在 /etc/hosts.equiv
 HostbasedAuthentication no #上面的项目类似，不过是给 version 2 使用的
 IgnoreUserKnownHosts no #是否忽略家目录内的 ~/.ssh/known_hosts
 ChallengeResponseAuthentication no #允许任何的密码认证
 UsePAM yes #利用 PAM 管理使用者认证，可以记录与管理
 
 UseDNS no
 	#关闭dns反向解析；
 	#sshd server默认会对连进来的ip进行dns反向解析，会浪费一定的连接时间；
```





### Authenticate Methods

#### 基于口令

只要知道用户名与密码就能登录远程服务器；  
可能会遭受中间人攻击；  



#### 基于密钥

自己生成一对密钥对，公钥放服务器上（文件~/.ssh/authorized_keys内，并且文件权限为600），当你登录时就进行（公钥）是否匹配；  
不会遭受中间人攻击；  


ssh-keygen 生成的id_rsa.pub与平时所见的证书不一样；得用openssl加工下；
> openssl genrsa -out id_rsa 1024   
> openssl rsa -in id_rsa -pubout -out id_rsa.pub  


问题：验证id_rsa.pub和id_rsa是否匹配？
> ssh-keygen  -y -f id_rsa > id_rsa.pub.tobecompared   
> 然后比较id_rsa.pub.tobecompared 与 id_rsa.pub 的内容是否一致  
>
> > 这么说RSA中，只要有key就能推导出pub？？？？ 



#### pam认证方式

Pluggable Authentication Modules(可插入认证模块)

```bash
#/etc/ssh/sshd_config
	UsePAM yes
#/etc/pam.conf
#/etc/pam.d/*
```

> http://www.cnblogs.com/ilinuxer/p/5087447.html



#### 集中式认证方式

LDAP是我们常用的一种集中式认证方式。



### tcp wrappers

/etc/hosts.deny

```bash
sshd: ALL
	#拒绝所有远程主机访问sshd服务。
```



/etc/hosts.allow

```bash
sshd: 172.22.48.
	#只允许xx网段访问sshd服务；
```







## SSH vs SSL

在最初的设计意图中
SSL(Secure Sockets Layer (SSL) and Transport Layer Security (TLS))被设计为加强Web安全传输(HTTP/HTTPS/)的协议(事实上还有SMTP/NNTP等)。 

> 加强web的 

SSH(Secure Shell)更多的则被设计为加强Telnet/FTP安全的传输协议,默认地,它使用22端口。  
> 加强fpt/telnet的



### The Same

如果按五层协议来划分的话，那SSH与SSL都可以算是应用层协议；  

他们都使用了非对称加密，将应用层进行了加密；  

他们其实都是比较基础的应用层协议，即它们上面还可以再放其它应用层协议，如FTP协议等；  



### The Difference

SSH不需要证书，即不需要公证机构（从这点来说，SSH的安全性要比SSH弱一些，貌似SSH无法解决中间人攻击）；  

SSH实现了主机用户名和密码的认证，这是SSH的SSH-TRANS部分完成的，而SSL没有这个功能。   

SSL只是数据加密， 而SSH则包括三部分，连接(ssh-connect)协议、认证(ssh-userauth)协议、加密(ssh-trans)协议。   



### Use Case

SSH一般用于私有访问，它提供了主机用户名和密码的验证功能（以及其它认证功能）；

但对于Web等这种提供公共服务的访问呢？
这种情况下，我并不想让用户登录主机的，也要求保证数据传输的安全，这时我们用SSL/TLS，我们会在自己设计的应用层（即应用程序）中验证用户的身份。更通俗的说

- SSH可以让用户以某个主机用户的身份登录主机，并对主机执行操作（即执行一些命令），目前用的最多的就是远程登录和SFTP（还有简易版的SCP）；  
- SSL则和主机用户名登录没有任何关系，它本身并不实现主机登录的功能，它只的一个单纯的加密功能。为了方便理解，可以简单的认为SSH=SSL+主机用户登录功能等应用层协议。





私有访问 还是 公有访问 ？？

> 私有访问，已经包含了用户认证这一块。


