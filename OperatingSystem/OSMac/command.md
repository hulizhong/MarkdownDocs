[TOC]



## ReadMe

mac特有指令相关记录。



## 用户相关

```bash
sudo -i
	#切换到root

su - xx
	#切换到普通用户xx
```





## 其它

### otool

```bash
otool -L /usr/bin/vim
	#替代ldd
```

### dtruss

```bash
sudo dtruss -aeodf -n wfe_client_util
	#替代strace
```



## Debug 相关

### lldb

http://lldb.llvm.org/tutorial.html
http://www.cocoachina.com/ios/20150819/11558.html
https://www.jianshu.com/p/67f08a4d8cf2



```bash
# 运行
process launch
run
r
# run with attach
process attach --pid 123
process attach --name Sketch
process attach --name Sketch --waitfor

# 线程状态
thread list
thread backtrace
thread backtrace all

# 查看调用栈状态
frame variable
frame variable variableName  #查看特定的变量
frame select 9
```



#### core文件

设置`ulimit -c`，生成在`/cores/core.pid`下。
`g++ -g`编译出的来<font color=red size=4>app调试信息单独在`app.dSYM`目录中，而不在app里面？？？</font>

```bash
# ulimit -c   #设置小了，有可能产生不了core文件，因为实际产生core大小>设置的值，就不会产生！
unlimited
# ls /cores/
core.58704

# lldb a.out /cores/core.58704    得依赖appname.dSYM目录；
# (lldb) r
Process 58814 launched: '/Users/Lizhong/a.out' (x86_64)
size_t.8 int.4 long.8 ptr.8
Process 58814 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x0)
    frame #0: 0x0000000100000eec a.out`main at offsetof.cpp:29
   26       };
   27  
   28       char *p = NULL;
-> 29       *p = 'c';
   30       /* Output is compiler dependent */
   31  
   32       printf("offsets: i=%ld; c=%ld; d=%ld a=%ld\n",

# rm -rf a.out.dSYM
# lldb a.out /cores/core.58704 
#(lldb) r
Process 58790 launched: '/Users/Lizhong/a.out' (x86_64)
size_t.8 int.4 long.8 ptr.8
Process 58790 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x0)
    frame #0: 0x0000000100000eec a.out`main + 108
a.out`main:
->  0x100000eec <+108>: movb   $0x63, (%rcx)
    0x100000eef <+111>: movl   %r9d, %edx
    0x100000ef2 <+114>: movl   %r10d, %ecx
    0x100000ef5 <+117>: movl   %r11d, %r8d
Target 0: (a.out) stopped.
```





## macPorts

```bash
port install ctags
```

## brew

先依赖curl进行包下载，完了之后才是安装！

### usage

brew commands usage as follow.

```bash
brew search [TEXT|/REGEX/]
brew info [FORMULA...]
	#show the infomation which already installed.
brew install FORMULA...
	#brew install --insecure/-k ctags
brew update
brew upgrade [FORMULA...]
brew uninstall FORMULA...
brew list [FORMULA...]  
```



### cant install with ssl.

当不能用ssl进行软件安装时，需要确认如下两点：

1. find the curl that u installed has support ssl. 
   1. 法一，`ucrl -V`，`curl --version` 可查看是否有https协议。
   2. 法二，`curl-config --protocols` 查看是否有https协议。
2. is ur system has the cert.



#### curl level

curl层面上的改动，在curl装有https的基础上，更改环境变量`curl-ca-bundle`的指向，或者让curl连接时不进行ssl认证。

```bash
curl-config --ca
	#查看libcurl/curl的curl-ca-bundle.crt的位置
export CURL_CA_BUNDLE="/etc/ssl/cert.pem"                  
	#如果curl-config没有找到对应位置，那么就将系统默认的ca-bundle.crt赋于curl-ca-bundle.

vim ~/.curlrc
	#注意：这个文件对curl指令不管用；
```



solution 1. config the ca-bundle.crt

> export CURL_CA_BUNDLE='/usr/share/curl/curl-ca-bundle.crt'
>
> cd /usr/share/curl
> mv download.crt curl-ca-bundle.crt



solution 2. disable the curl ssl verify.

> vim ~/.curlrc
> insecure
> cacert=/path/to/my/certs.pem





#### system level

系统层面的解决办法，就是装证书！

不系统没有对应连接证书的CA是，经常会报如下错误：

> **curl: (60) SSL certificate problem: self signed certificate in certificate chain** in Debian.
> U can see "Certificate Unknow" 

> **curl: (60) SSL certificate problem: Invalid certificate chain** in OSX.





------

**Resolve base on Debin/Ubuntu**  ---ca-bundle

1. install ca-certificates

```bash
apt-get install ca-certificates

dpkg -L ca-certificates 
	#/usr/share/ca-certificates
	#/etc/ca-certificates
	#/etc/ssl/certs
	#/usr/sbin/update-ca-certificates

dpkg -L openssl      
	#/etc/ssl/certs
	#/etc/ssl/openssl.cnf
	#/usr/bin/openssl
	#/usr/bin/c_rehash
	#/usr/lib/ssl/certs
	#/usr/lib/ssl/openssl.cnf

dpkg -L curl
	#/usr/bin/curl
```

1. put the ur cert into <font color=red>/usr/share/ca-certificates/</font>
2. modify the <font color=red>/etc/ca-certificates.conf</font>, add the new relative path into it.
3. run <font color=red>update-ca-certificates</font> to update **/etc/ssl/certs/** by reading ca-certificates.conf.



------

**Resolve base on OSX**

```bash
#security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/SkyGuard.crt  
SecCertificateAddToKeychain: write permissions error

#sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/SkyGuard.crt 
Password:
```



------

On Win System.

On CentOS System.

https://stackoverflow.com/questions/9626990/receiving-error-error-ssl-error-self-signed-cert-in-chain-while-using-npm

