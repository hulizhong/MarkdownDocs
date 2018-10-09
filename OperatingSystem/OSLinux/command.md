
## ReadMe
收录指令集


## 查找
grep
```bash
grep key ./ -rn --color --exclude-dir dir1
	在当前目录下查找key，但是排除dir1目录
```

find
```bash
find ./ -name key.txt
	在当前目录下查找key.txt
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



