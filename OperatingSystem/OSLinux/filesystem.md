[TOC]



## ReadMe

主要讲解linux文件系统、当前文件共享协议等知识；



## Dir Names

概要认识下各目录
/bin(binary) 【】essential command binaries that must be present when the system is mounted in single-user mode.
/sbin(system binary) 【】essential system binaries that are generally intended to be run by the root user for system administration.
/lib 【】Essential Shared Libraries，为/bin, /sbin提供库
/dev 【】Device Files
/opt 【】contains subdirectories for optional software packages. It’s commonly used by proprietary（授权） software that doesn’t obey the standard file system hierarchy.
/usr 【】User Binaries & Read-Only Data
	/usr/bin 【】non-essential applications
	/usr/sbin 【】on-essential system administration binaries
	/usr/lib 为/usr/bin提供库
	/usr/local 【】源码编译安装的app默认地址
		/usr/local/bin/
		/usr/local/lib/
		/usr/local/src/
	/usr/src
/var 【】the writable counterpart to the /usr
/proc 【】Kernel & Process Files. It contains special files that represent system and process information.
/sys
/run 【】Application State Files
/selinux 【】SELinux Virtual File System. Only your Linux distribution uses SELinux for security then have it, the /selinux directory contains special files used by SELinux. 



小结：
系统管理相关文件：/sbin, /usr/sbin, /usr/local/sbin
用户执行相关文件：/bin/,  /usr/bin,  /usr/local/bin
库路径： /lib,  /usr/lib,  /usr/local/lib





## proc



## sys

https://www.ibm.com/developerworks/cn/linux/l-cn-sysfs/

root@~# ls /sys/class/net/
eth0  lo

## var

/var/spool 包含正在等待某种后续处理的数据，在处理后这些数据经常会被删除。

/var/lib/dpkg 系统中所有的package信息都记录在此；
/var/lib/dpkg/avaliable 当前安装源下所有软件包的描述信息；（已安装 + 未安装）
/var/lib/dpkg/info/  保存种软件的配置文件列表；
	如果apt-get一直报某个软件的错误，那么可以在该目录下删除对应软件的配置；





## 文件共享

### NFS

Network File Sharing.
It’s the distributed file system protocol usually used in Unix Operating System. 
It’s effictive for large files and for permanent setup.

NFS dameon is runnning in server sites and configure the list of export folders. After that,install the client package to get access to the network share.



### CIFS

Common Internet File System.
It works on application layer network protocol and also known as Server Message Block (SMB). 
Micorsoft Windows Network share uses this protocol.
Samba is an implemantion of CIFS to work along in Unix system.

-----------

linux上挂载win共享目录

```bash
# mount -t cifs -o username=acessUser,password=acessPwd 172.21.1.1:path linuxMountName
```

设置开机自动挂载

```bash
# vi /etc/fstab<br>
192.168.1.12:Download  /mnt/share   cifs   defaults,username=administrator,password=123456 0 2
```



### Samba

Samba is an open-source project to share resources among windows and linux mixed-network.



