[TOC]



## ReadMe

记录各种版本linux系统的各种配置，目前包含debian, ubuntu, redhat.



## Debian

### 网络相关

接口ip静态配置；

```bash
root@bogon:/etc# cat /etc/network/interfaces 
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0       #支持热插拔
iface eth0 inet static   #静态配置
address 172.22.48.102
netmask 255.255.0.0
gateway 172.22.0.1
```



DNS配置

```bash
root@bogon:/etc# cat /etc/resolv.conf
domain xx.com   #domain 定义本地域名
search yy.com   #search 定义域名的搜索列表
	#当要查询没有域名的主机，主机将在由search声明的域中分别查找。
	#domain, search不能共存；（谁的配置在最后，将会被使用）
nameserver 172.22.1.2   #nameserver 定义DNS服务器的IP地址
nameserver 172.22.1.3
```





## Ubuntu



## RedHat

