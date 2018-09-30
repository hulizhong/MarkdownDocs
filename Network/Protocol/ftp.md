[TOC]



## ReadMe

ftp相关知识讨论；



## About ftp

ftp client与server之间会建立2个socket，占用20、21端口，分别用于数据、指令的传输。（提高效率）

> 主动模式（PORT），cli连接ser的21端口，并监听本地某一端口N，并告知ser本地的数据监听端口（通过指令PORT N）；然后ser会以20端口连接到cli的N端口进行数据传输。（缺点：外到内的连接有可能被FW拒）
>
> 被动模式（PASV），cli打开N, N+1两个端口，用N连接ser的21端口并上传指令“PASV”；然后ser返回“227 entering passive mode (127,0,0,1,4,18)”，cli用N+1端口连接127.0.0.1 : 4*255+18用这条连接进行数据传输。

ftp client借助指令通道，向server发送指令；server端会给出对应指令的响应，及对指令的执行。



### Request Commands

FTP 每个命令都有 3 到 4 个字母组成，命令后面跟参数，用空格分开。每个命令都以 "\r\n"结束。

```bash
COMMAND params\r\n
```



USER: 指定用户名。通常是控制连接后第一个发出的命令。
	“USER gaoleyi\r\n”： 用户名为gaoleyi 登录。

PASS: 指定用户密码。该命令紧跟 USER 命令后。
	“PASS gaoleyi\r\n”：密码为 gaoleyi。

SIZE: 从服务器上返回指定文件的大小。
	“SIZE file.txt\r\n”：如果 file.txt 文件存在，则返回该文件的大小。

CWD: 改变工作目录。
	如：“CWD dirname\r\n”。

PASV: 让服务器在数据端口监听，进入被动模式。
	如：“PASV\r\n”。

PORT: 告诉 FTP 服务器客户端监听的端口号，让 FTP 服务器采用主动模式连接客户端。
	如：“PORT h1,h2,h3,h4, p1,p2”。<font color=red>监听端口为p1*255 + p2</font> 

RETR: 下载文件。
	“RETR file.txt \r\n”：下载文件 file.txt。

STOR: 上传文件。
	“STOR file.txt\r\n”：上传文件 file.txt。

REST: 该命令并不传送文件，而是略过指定点后的数据。此命令后应该跟其它要求文件传输的 FTP 命令。
	“REST 100\r\n”：重新指定文件传送的偏移量为 100 字节。

QUIT: 关闭与服务器的连接。



### Response Code

响应码由三位数字ABC表示：
A 命令状态的一般性指示，比如响应成功、失败或不完整。
B 响应类型的分类，如 2 代表跟连接有关的响应，3 代表用户认证。
C 提供了更加详细的信息。

第一个数字A的含义如下：
1 表示服务器正确接收信息，还未处理。
2 表示服务器已经正确处理信息。
3 表示服务器正确接收信息，正在处理。
4 表示信息暂时错误。
5 表示信息永久错误。

第二个数字B的含义如下：
0 表示语法。
1 表示系统状态和信息。
2 表示连接状态。
3 表示与用户认证有关的信息。
4 表示未定义。
5 表示与文件系统有关的信息。



