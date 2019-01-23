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

gdb的升级；



### core switch

设置`ulimit -c`，生成在`/cores/core.pid`下。





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

