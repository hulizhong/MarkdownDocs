[toc]

## ReadMe
http（超文本传输协议）
是一个基于请求与响应模式的、无状态的、应用层的协议，常基于TCP的连接方式。   
无状态：第1次请求响应与第2次请求响应无关（1.1加入了cookie） 

## 各版本的不同特性
http各版本

- versin.1.0
- versin.1.1
	- keepalived
	- cookie
	- 100-continue
- versin.2.0（下一代http，不是web2.0）
	- 二进制格式
	- 多路复用
	- 报头压缩
	- server push 


100-continue  
> 这个是http1.1协议为了提高效率设置的。当客户端要POST较大数据给webserver时，可以在发送http头时带上Expect: 100-continue，服务器如果接受这个请求，应答一个HTTP/1.1 100 Continue，那么客户端就继续传输正文，否则应答417，客户端就放弃传送剩余的数据了。这样就避免客户端吭哧吭哧传了一大堆数据上去，结果服务端发现不需要的情况。
	

## http请求方法
|方法名|解释|
|-----|---|
|POST
|DELETE |无验证（不推荐）
|PUT
|GET
|HEAD
|OPTIONS |请求查询服务器的性能，或者查询与资源相关的选项和需求
|TRACE |请求服务器回送收到的请求信息，主要用于测试或诊断
|CONNECT |遂道


## http状态码
|响应码|说明|
|-----|---|
|1xx |指示信息（表示请求已接收，继续处理）
|2xx |成功（表示请求已被成功接收、理解、接受）
|204 |No Content，处理了请求但没有body返回（Content-Length=0）
||
|3xx |重定向（要完成请求必须进行更进一步的操作）
|301 |Moved Permanently
|302 |Moved Temporarily
|304 |Not Modified
||
|4xx |客户端错误（请求有语法错误或请求无法实现）
|400 |Bad Request
|401 |Unauthorized
|403 |Forbidden
|404 |Not Found
||
|5xx |服务器端错误（服务器未能实现合法的请求）
|500 |Internal Server Error
|501 |Not Implemented
|502 |Bad Gateway
|503 |Service Unavailable


## 常用的请求头
|字段名|说明|
|-----|---|
|Accept |指定客户端接受哪些类型的信息，注意没有Accept-type这个key；
|Accept-Charset |指定客户端接受的字符集，如不设置则可接受所有；
|Accept-Encoding |类似于Accept，但是它是用于指定可接受的内容编码，如不设置则可接受所有；
|Accept-Language |类似于Accept，但是它是用于指定一种自然语言，如不设置则可接受所有；
|Authorization |主要用于证明客户端有权查看某个资源；
|Host |主要用于指定被请求资源的Internet主机和端口号；（必须字段）
|User-Agent |允许客户端将它的操作系统、浏览器和其它 属性告诉服务器；


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


## 常用的响应头
|字段名|说明|
|-----|---|
|Location |响应报头域用于重定向接受者到一个新的位置；
|Server |响应报头域包含了服务器用来处理请求的软件信息，与User-Agent请求报头域是相对应的；
|WWW-Authenticate |客户端收到401响应消息时候，server端指示client的认证方法；
|Content-Encoding |内容编码，用于body的编码
|transfer-encoding |传输编码，用于网络传输
|Content-Language |实体报头域描述了资源所用的自然语言；
|Content-Length |实体报头域用于指明实体正文的长度，以字节方式存储的十进制数字来表示；
|Content-Type |指明发送给接收者的实体正文的媒体类型；
|Last-Modified |指示资源的最后修改日期和时间；
|Expires |给出响应过期的日期和时间；（与缓存相关）

### Transfer-Encoding: chunked
chunked编码的基本方法是将大块数据分解成多块小数据，每块都可以自指定长度(Content-Length)，浏览器不需要等到内容字节全部下载完成，只要接收到一个chunked块就可解析页面。




## https
https = http + 加密 + 认证 + 完整性保护


## 认证方法

### basic 
user:passwd的base64

### ntlm(NT LAN Manager)
为Windows NT早期版本的标准安全协议，也称为NT challenge/response(NTCR)协议。  
> 握手过程并非一步到位，所以出现多个401是正常的。 
> 不是一个http认证scheme，而是一个connection认证scheme，虽然它使用http状态码和http头。  


## RESTful Webservice
适用于 无状态、面向资源的设计风格。
> 无状态：上次调用跟这次调用没有关系（没有会话关系）。  
> 面向资源：将操作的对象抽象为资源。  


按照REST原则设计的软件、体系结构，通常被称为“REST式的”（RESTful），在本文中以下称之为RESTful Web服务，以便于和基于SOAP的Web服务区别。   
> SOAP偏向于面向活动，有严格的规范和标准，包括安全，事务等各个方面的内容，同时SOAP强调操作方法和操作对象的分离，有WSDL文件规范和XSD文件分别对其定义。   
> REST强调面向资源，只要我们要操作的对象可以抽象为资源即可以使用REST架构风格。   


REST API设计概要
> POST、DELETE、PUT、GET的http请求动作来表示对资源的CRUD操作；  
> 在URI中体现资源，而非动作；  
> API要带版本号，以达到兼容的目的；（最好加在URI中）  
> 返回值 ；（最好用json，因为有些web语言直接就能将json转成对象；同样针对请求体）  
> 鉴权；（最好将token放在请求头中）  
> HTTP响应码；    
>> 只适合http，而非https吗；（不是这样的，如果针对安全的考虑可以用https）  

RESTful API ?? 
> 没有restfull api这一说；
> 符合rest api设计风格的web service可以叫restfull web service；


## web2.0
Web2.0 是相对于Web1.0 的新的时代。指的是一个利用Web的平台，由用户主导而生成的内容互联网产品模式。
> Web2.0 则更注重用户的交互作用，用户既是网站内容的浏览者，也是网站内容的制造者。  
> 在模式上由单纯的“读”向“写”以及“共同建设”发展；由被动地接收互联网信息向主动创造互联网信息发展，从而更加人性化。

Web2.0模式下的互联网应用具有以下显著特点：
> 去中心化,开放,共享为显著特征。 

