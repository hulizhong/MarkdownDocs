[TOC]

## ReadMe
http（超文本传输协议）
是一个基于请求与响应模式的、无状态的、应用层的协议，常基于TCP的连接方式。   
无状态：第1次请求响应与第2次请求响应无关（1.1加入了cookie） 



## Http Request

### Request Method

请求方法如下表

|方法名|解释|
|-----|---|
|POST|
|DELETE |无验证（不推荐）|
|PUT||
|GET||
|HEAD||
|OPTIONS |请求查询服务器的性能，或者查询与资源相关的选项和需求|
|TRACE |请求服务器回送收到的请求信息，主要用于测试或诊断|
|CONNECT |遂道|



### Request Header

常用的请求头如下表：

|字段名|说明|
|-----|---|
|Accept |客户端能接受哪些类型的信息（MIME类型），注意没有Accept-type这个key；|
|Accept-Charset |客户端能接受的字符集，如不设置则可接受所有；|
|Accept-Encoding |类似于Accept，但是它是用于指定可接受的内容编码，如不设置则可接受所有；<font color=red>如gzip</font>.|
|Accept-Language |类似于Accept，但是它是用于指定一种自然语言，如不设置则可接受所有；|
|Authorization |授权信息，主要用于证明客户端有权查看某个资源；|
|Host |主要用于指定被请求资源的Internet主机和端口号；（必须字段）|
|User-Agent |允许客户端将它的操作系统、浏览器和其它 属性告诉服务器；|
|if-modified-since |只有在指定日期之后修改过才会返回资源，否则返回304.|



### Multipart

一个http请求可以上传多个文件（不相关），规范：

- 必须为POST方法；
- 必须含Content-Type头，且值为multipart/form-data； 
- 每个part如下

```bash
-----------------分隔字符串
Content-Disposition: form-data; name="request"
Content-Type: application/octet-stream
…数据…
```





## Http Response

### Response Code

状态码如下表：

|响应码|说明|
|-----|---|
|1xx |指示信息（表示请求已接收，继续处理）|
|100 |client must send the request body.|
|101 |客户端要求服务器根据请求转换http version.|
| ||
|2xx |成功（表示请求已被成功接收、理解、接受）|
|201 |Created, 已被处理（一个新资源已被创建）。|
|202 |接受和处理，但处理未完成。|
|203 |Non-Authoritative Information, 返回信息不确定或不完整。|
|<font color=red>204</font> |No Content，处理了请求但不需要返回body（Content-Length=0）|
|205 |Reset Content, 处理了请求无需要返回内容，但需要浏览器重置文档视图。|
|<font color=red>206</font> |Partial Content, 处理了部分get请求。（如分段下载大文件）|
|||
|3xx |重定向（要完成请求必须进行更进一步的操作）|
|301 |Moved Permanently|
|302 |Moved Temporarily|
|304 |Not Modified|
|||
|4xx |客户端错误（请求有语法错误或请求无法实现）|
|400 |Bad Request|
|401 |Unauthorized，未授权|
| |401.1 未授权之登录失败|
| |.2 服务器配置问题导致登录失败。|
| |.3 ACL禁止访问资源。|
| |.4 授权被筛选器拒绝。|
| |.5 ISAPI或CGI授权失败。|
|403 |Forbidden|
| |1，禁止可执行访问；2，禁止读访问；3，禁止写访问。|
| |4，要求SSL；5，要求SSL 128|
| |6，IP地址被拒绝|
| |7，要求客户证书；8，禁止问点访问；9，连接的用户过多|
| |10，配置无效；11，密码更改；12，映射器拒绝访问；|
| |13，客户证书已被吊销；15，客户访问许可过多；|
| |16，客户证书不可信或者无效；17，客户证书已到期或者尚未生效；|
|404 |Not Found|
|407 |类似401，用户必须首先在代理服务器上得到授权。|
|||
|5xx |服务器端错误（服务器未能实现合法的请求）|
|500 |Internal Server Error|
| |11，服务器关闭。12，应用程序重新启动。|
| |13，服务器太忙。14，应用程序无效。|
| |15，不允许理论地球化学global.asa。|
| |100，ASP错误。|
|501 |Not Implemented|
|502 |Bad Gateway|
|503 |Service Unavailable|



### Response Header

常用的响应头如下表：

|字段名|说明|
|-----|---|
|allow|服务器支持哪些请求方法。|
|Location |响应报头域用于重定向接受者到一个新的位置；|
|Server |响应报头域包含了服务器用来处理请求的软件信息，与User-Agent请求报头域是相对应的；|
|WWW-Authenticate |客户端收到401响应消息时候，server端指示client的认证方法；|
|Content-Encoding |内容编码，用于body的编码。只有解码后才是content-type指定的类型。|
|transfer-encoding |传输编码，用于网络传输|
|Content-Language |实体报头域描述了资源所用的自然语言；|
|Content-Length |实体报头域用于指明实体正文的长度，以字节方式存储的十进制数字来表示；|
|Content-Type |指明发送给接收者的实体正文的媒体类型（MIME类型）；|
|Last-Modified |指示资源的最后修改日期和时间；|
|Expires |给出响应过期的日期和时间；（与缓存相关）|
|refresh |表示浏览器应该在多少秒后刷新文档。|



## Protocol

url协议格式如下：

```bash
scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]
	#host域名，字母a-z不区分大小写、数字0-9、连接符-构成。
		#限制：首位只能是字母、数字。
		#长度限制，国际通用顶级如.com不超过26个字符；国家顶级域名如.cn不超过20；
	#query 以?号为起点，以&符为分隔，每对key,value用=连接；并且通常以UTF8的URL编码，避开字符冲突的问题。
	#fragment 锚点，标识文章特定分段（相当于标签的特点，点击就进入对应章节了）；
```



### Transfer-Encoding: chunked

chunked编码的基本方法是将大块数据分解成多块小数据，每块都可以自指定长度(Content-Length)，浏览器不需要等到内容字节全部下载完成，只要接收到一个chunked块就可解析页面。



### 1.0 VS 1.1

http各版本的特性

- versin.1.0
- versin.1.1
  - 连接复用keepalived，减少三次握手开销。
  - 在request中多了host域。（因为一个ip不止对应一个host，如虚拟主机。）
  - 日期时间戳格式。（request中只能用一种了。）
  - add response code. 100, 101, 203, 205...
    - 100 continue，client探测server要不要接收request body,再决定是否发送reqeust body.
  - add request method. (options, put, delete, trace, connect.)
  - cookie
- versin.2.0（下一代http，不是web2.0）
  - 二进制格式
  - 多路复用
  - 报头压缩
  - server push 



#### 100-continue

这个是http1.1协议为了提高效率设置的。
当客户端要POST较大数据给webserver时，可以在发送http头时带上Expect: 100-continue，服务器如果接受这个请求，应答一个HTTP/1.1 100 Continue，那么客户端就继续传输正文，否则应答417，客户端就放弃传送剩余的数据了。这样就避免客户端吭哧吭哧传了一大堆数据上去，结果服务端发现不需要的情况。



### Https

https = http + ssl/tls

提供：加密 + 认证 + 完整性保护



### session VS cookie

Cookie、Session均是<font color=red>解决http无状态的方法，都是保存客户端状态的机制</font>。
Session可以用Cookie来实现，也可以用URL回写的机制来实现。

Cookie VS Session

- Cookie将状态保存在客户端，Session将状态保存在服务器端；
- Cookies是客户端在本地机器上存储的小段文本并随每一个请求发送至同一个服务器。Cookie最早在RFC2109中实现，后续RFC2965做了增强。网络服务器用HTTP头向客户端发送cookies，在客户终端，浏览器解析这些cookies并将它们保存为一个本地文件，它会自动将同一服务器的任何请求缚上这些cookies。Session并没有在HTTP的协议中定义；
- Session是针对每一个用户的，变量的值保存在服务器上，用一个sessionID来区分是哪个用户session变量,这个值是通过用户的浏览器在访问的时候返回给服务器，当客户禁用cookie时，这个值也可能设置为由get来返回给服务器；
- 就安全性来说：当你访问一个使用session 的站点，同时在自己机子上建立一个cookie，建议在服务器端的SESSION机制更安全些.因为它不会任意读取客户存储的信息。



---

**cookie**

与之相关的头

| cookie      | client将server设置的cookie写入到请求中，发给server. |
| ----------- | --------------------------------------------------- |
| set-cookie  | server向client设置cookie.                           |
| cookie2     | client指示服务器支持的cookie版本。                  |
| set-cookie2 | server向client设置cookie.                           |

如下使用流程，实现**会话保持**。

> server在响应中使用set-cookie将cookie回送给client；
> client在后续的请求中将cookie设置到请求头中，发送给server；



---

**session**

session的实现方式：

- 法一：使用cookie来实现。
  - server给每个session分配一个唯一的sessionID，并通过set-cookie发送给client.
  - client后续的请求都带上这个sessionID.
- 法二：使用url回显来实现。
  - server响应给client的页面中所有的链接都带上了sessionID参数；
  - 如此client点击任何一个链接都会把sessionID带回服务器；





### http proxy

主要为了达到以下一些目的：

- 突破访问限制。（场景：只有一些点可对目标资源进行访问。）
- 截获流量进行安全分析。
- 隐藏自己的IP信息。



### web cache

出于`减少client的延迟、降低网络带宽消耗`而架设在server, client之间的一层应用。

依赖的http特性

> expires. 指明内容过期时间，GMT时间（格林威治）。
> cache-control. 更细致的cache控制。
> last-modified. 资源的最后一次修改时间。
> etag. 资源的检验值，在server上某段时间内是唯一的。
> date. 服务器时间。
> if-modified-since. 客户端存取的该资源最后一次修改的时间，同于last-modified.
> if-none-match. 客户端上该资源的校验值，同于etag.



客户端缓存生效流程

```bash
# ----GET xx---->
# <----200 ok, last-modified + etag----
# client将xx放在cache中，并记录上述两属性，当再有相同的请求时（get xx），如下：
# ----GET xx, if-modifed-since + if-none-match --->
# <-------------------------------------304 not modified-----
```



http三种缓存机制

- freshness
- validation
- invalidation

https://www.cnblogs.com/chenqf/p/6386163.html   ---rtodo



### Resume transmission

依赖http 1.1的如下特性：

> range: start-stop
> content-range: start-stop/len
> accept-ranges: bytes
> content-length: 
>
> 200
> 206 partial content.
> 416 requested range not satisfiable.（client发来的range不合理）

`断点续传`、`多线程下载`这些都是靠http range来支持的。

```bash
GET /test.rar HTTP/1.1 
range: bytes=100-200

HTTP/1.1 200 OK 
content-range: bytes 100-200/100  即body为该资源的100-200字节，共200字节

#-----------------------------
GET /123.zip HTTP/1.1

HTTP/1.1 200 OK 
Accept-Ranges : bytes   #告诉客户端，我们是支持断点传输的。
Content-Length : 1900   #文件总大小 
Content-Type : image/jpeg  #文件类型

#-------------------------------
GET /123.zip HTTP/1.1 
Range：bytes=580-

HTTP/1.1 206 Partial Content
Accept-Ranges : bytes
Content-Type : image/jpeg #文件类型
Content-Length : （1900 - 580） #长度则不是总长度了，而580到1900共有多少字节。
Content-Range :bytes 580-（1900-1 ) / 1900   #为什么结束字节要减1呢。这是因为发来的Range请求头文件下标是0开始，那么结尾数显示也要减1；但是实际上输出的字节是不减1的，完全是写法问题。
```

连接断开重连时，只请求资源未下载部分。
多线程下载，每个线程只下载一段，最后合并各个分段。



## Authenticate Methods

### basic 
user:passwd的base64

### ntlm(NT LAN Manager)
为Windows NT早期版本的标准安全协议，也称为NT challenge/response(NTCR)协议。  
> 握手过程并非一步到位，所以出现多个401是正常的。 
> 不是一个http认证scheme，而是一个connection认证scheme，虽然它使用http状态码和http头。  



## Virtual host

虚拟主机（网站空间）：在http服务器上划分出一定的磁盘空间供用户放置站点、应用组件等。

> 就是一台运行在互联网上的服务器划分成多个“虚拟”的服务器，每一个虚拟主机都**具有独立的域名和完整的Internet服务器**（支持www, ftp, email等）功能。



实现原理

> 请求头的host字段。

客户端发送HTTP请求的时候，会携带Host头，Host头记录的是客户端输入的域名。这样服务器可以根据Host头确认客户要访问的是哪一个域名。





## RESTful Webservice

适用于 无状态、面向资源的设计风格。
> 无状态：上次调用跟这次调用没有关系（没有会话关系）。  
> 面向资源：将操作的对象抽象为资源。  


按照REST原则设计的软件、体系结构，通常被称为“REST式的”（RESTful），在本文中以下称之为RESTful Web服务，以便于和基于SOAP的Web服务区别。   
> SOAP偏向于面向活动，有严格的规范和标准，包括安全，事务等各个方面的内容，同时SOAP强调操作方法和操作对象的分离，有WSDL文件规范和XSD文件分别对其定义。   
> REST强调面向资源，只要我们要操作的对象可以抽象为资源即可以使用REST架构风格。   

RESTful API ?? 

> 没有restfull api这一说；
> 符合rest api设计风格的web service可以叫restfull web service；



----

REST API设计概要

1. POST、DELETE、PUT、GET的http请求动作来表示对资源的CRUD操作；

2. 在URI中体现资源，而非动作；

3. API要带版本号，以达到兼容的目的；（最好加在URI中）

4. 返回值 ；（最好用json，因为有些web语言直接就能将json转成对象；同样针对请求体）

5. 鉴权；（最好将token放在请求头中）  





## web2.0

Web2.0 是相对于Web1.0 的新的时代。指的是一个利用Web的平台，由用户主导而生成的内容互联网产品模式。
> Web2.0 则更注重用户的交互作用，用户既是网站内容的浏览者，也是网站内容的制造者。  
> 在模式上由单纯的“读”向“写”以及“共同建设”发展；由被动地接收互联网信息向主动创造互联网信息发展，从而更加人性化。

Web2.0模式下的互联网应用具有以下显著特点：
> 去中心化,开放,共享为显著特征。 

