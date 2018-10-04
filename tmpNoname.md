[TOC]

## Notes.

nc
- nc -l 127.0.0.1 8888
- telnet 127.0.0.1 8888
	- ctrl + ]


API vs SDK
> Software Development Kit
> Application Programming Interface
>
> > Technically, if an API is well-documented, you don't need an SDK to build your own software to use the API. But having an SDK generally makes the process much easier.

- Easier integration 简单
- Faster time to market 快速
- Stronger security 安全
- Reliability 可靠
	- 新特性的引入，会发布新的sdk，sdk是经过验证的；


gdb 
带参数调试
- gdb --args a.out 5
- set args 5; run|start
- run 5


## OSX

brew
不能安装软件 【osx不能用root来安装软件】
```bash
Lizhong$ sudo brew install gdb
Password:
Error: Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.

Lizhong$ sudo chown -R $(whoami) /usr/local

Lizhong$ brew install gdb
	#注意前面不用sudo.
	#或许最开始没加suod，就不会有问题；
```

运行缺少库
```bash
$ ./spe_client_util3
dyld: Library not loaded: libboost_thread.dylib
> export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/Users/Lizhong/swg/external/lib/:/Users/Lizhong/swg/internal/lib/
>> 比linux多个DY
```


errno 55
```cpp
#define ENOANO 55 /* No anode */

strerror(errno)  //No buffer space available

```


```bash
$ sysctl -a | grep tcp.sendspace
net.inet.tcp.sendspace: 131072
$ sudo sysctl net.inet.tcp.sendspace=131072
```


gcc/g++判断OS类型的宏
```cpp
//OS X
defined(Macintosh)                         //Mac OS 9/X
(defined(__APPLE__) && defined(__MACH__))  //Defined by GNU C and Intel C++

//Linix
defined(__linux)      //老式的、废弃了
defined(linux)        //老式的、废弃了
defined(__linux__)

//Windows
defined(_WIN32)
defined(_WIN64)
defined(_WIN32_WCE)   //define by VS c++.
```


### 未解决的
#### gdb签名
- gdb安装后不能使用，还需要签名
```bash
(gdb) r
Starting program: /Users/Lizhong/swg/WebFilteringEngine/build/src/skyguard/policy_engine/tool/spe_client/a.out 
Unable to find Mach task port for process-id 20749: (os/kern) failure (0x5).
 (please check gdb is codesigned - see taskgated(8))
(gdb) 
```



- 生成证书之后执行以下命令：
```bash
bogon:~ Lizhong$ codesign -s gdb_codesign gdb
gdb: No such file or directory
bogon:~ Lizhong$
bogon:~ Lizhong$ which gdb       
/usr/local/bin/gdb
bogon:~ Lizhong$
bogon:~ Lizhong$ codesign -s gdb_codesign /usr/local/bin/gdb
/usr/local/bin/gdb: unknown error -1=ffffffffffffffff

```

- Need do command `codesign ...` in MacOS GUI mode.

- 但是每次启动gdb怎么都需要输入账号密码；





## 科学思维

by 李枫、舒静庐



战略问题、战术解决；
战略是务虚、战术是务实；
战略是跨方步、战术是踏实地；

挽功当挽强、用箭当用长；射人先射马、擒贼先擒王；



创新思维：让思维不同凡响；


逻辑思维：让思想更严密、更准确；
辩证思维：用理性的认识把握事物；
	任何问题的答案都绝非唯一；
	对立统一的法则；
	换位思考，用对方的视角看问题；



形象思维：让思维的翅膀高高的飞翔；
联想思维：打破一切束缚和框框；
	举一反三；
	由此及彼，风马牛不相及；



系统思维：着眼整体，注重综合；
	综合分析；
	用计划作向导；
开放思维：拓开思维的瓶颈；
	换个好思路，就能拥有好出路；



发散思维：张开思维的大伞；
	使人触类旁通；
	善于从不同的方面思考同一问题；
立体思维：多角度、多层面地思考



逆向思维：从相反的方向来思考问题；
	反中求胜；
迂回思维：以曲为直，围魏救赵；
	欲显先隐；



质疑：敢于怀疑一切；
	尊重权威，但不要迷信权威；
	学会提问；
评判：颠覆固有观念和结论；
	人生要敢于向权威进行挑战；



超前：登高望远，领略无限风光；
	需要建立在科学预测的基础上；
冷门：另辟蹊径，创造不同凡响；
	打出奇招可以攻坚克难；



简单：册繁就简，抓住本质；
	看似简单，往往最好；
	把问题简单化，成功其实很简单；
模糊：不求精确，大处着眼；



直觉：发挥人的第6感；
	并非与理性思维对立；
灵感：抓住大脑中灵感的火花；



## 中国人的气质

by 刘文飞，刘晓X



面子

节俭
控制需求、杜绝浪费、尽量的花小钱办大事。

勤劳
长度（投入的时间长度）、广度（愿意勤劳的人数）、强度（投入的精力）这三维的结合。

礼节

漠视时间

漠视精确

误解的才能

拐弯抹角的才能

灵活的固执
这些人全都不相信我们的决断，而对们们自己的判断推崇备至；
灵活：特指给他指正或者提出要求的时候他不拒绝、表示接受，可回过头来又按自己的老一套来弄；

智力混沌

神经麻木

轻视外国人

缺乏公共精神
人人为自己、上帝为大家；----》人人为自己，上帝也为他自己；

保守

漠视舒适和便利

生命力

忍耐和坚韧

知足常乐

孝顺

仁慈

缺乏同情心

社会台风

相互负责和遵纪守法

相互猜疑

缺乏诚信

多神论、泛神论、无神论





https://www.zhihu.com/question/22074764

https://blog.csdn.net/lengxingxing_/article/details/76221848



中国最美的地方 https://baike.baidu.com/item/%E4%B8%AD%E5%9B%BD%E6%9C%80%E7%BE%8E%E7%9A%84%E5%9C%B0%E6%96%B9/2258606





域名劫持（通过arp欺骗设置）
NetFuke为什么源地址写本端GateWay的地址？目标写目标网站的ip。



