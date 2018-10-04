[TOC]



## ReadMe



管理相关

```bash
sudo -i
	#切换到root
su - xx
	#切换到普通用户xx


```





指令集

```bash
otool -L /usr/bin/vim
	#替代ldd

sudo dtruss -aeodf -n wfe_client_util
	#替代strace
```



## Commands

### install pacakge

#### Using Homebrew

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

**Notice**. when cant install with ssl.

> 1. find the curl that u installed has support ssl. (curl -V/--version,  curl-config --protocols)
> 2. is ur system has the cert.



solution 1. config the ca-bundle.crt

> export CURL_CA_BUNDLE='/usr/share/curl/curl-ca-bundle.crt'
>
> cd /usr/share/curl
> mv download.crt curl-ca-bundle.crt

solution 2. disable the curl ssl verify.

> vim ~/.curlrc
> insecure
> cacert=/path/to/my/certs.pem

```bash
export CURL_CA_BUNDLE='/usr/share/curl/curl-ca-bundle.crt'
curl-config --ca

curl-ca-bundle.crt
ca-bundle.crt
```



use curl command

```bash
curl-config --ca
	#查看libcurl/curl的curl-ca-bundle.crt的位置
export CURL_CA_BUNDLE="/etc/ssl/cert.pem"                  
	#如果curl-config没有找到对应位置，那么就将系统默认的ca-bundle.crt赋于curl-ca-bundle.

vim ~/.curlrc
	#注意：这个文件对curl指令不管用；
```



#### System Without Certificate

will report error as follow.

Case 1. **curl: (60) SSL certificate problem: self signed certificate in certificate chain** in Debian.
U can see "Certificate Unknow" 

Case 2. **curl: (60) SSL certificate problem: Invalid certificate chain** in OSX.

then how to resolve ??



------------------

On Debin/Ubuntu System.

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

2. put the ur cert into <font color=red>/usr/share/ca-certificates/</font>
3. modify the <font color=red>/etc/ca-certificates.conf</font>, add the new relative path into it.
4. run <font color=red>update-ca-certificates</font> to update /etc/ssl/certs by reading ca-certificates.conf.



-----------

On OSX System.

```bash
#security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/SkyGuard.crt  
SecCertificateAddToKeychain: write permissions error

#sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/SkyGuard.crt 
Password:
```



---------

On Win System.

On CentOS System.

https://stackoverflow.com/questions/9626990/receiving-error-error-ssl-error-self-signed-cert-in-chain-while-using-npm





#### Using MacPorts

```bash
port install ctags
```