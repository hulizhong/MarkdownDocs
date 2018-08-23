[TOC]



## ReadMe

讨论nginx的源码、搭建；





## Nginx Proxy

### WebSocket proxying

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

### ngx_mail_proxy_module



### ngx_stream_proxy_module

module (1.9.0) allows proxying data streams over TCP, UDP (1.9.13), and UNIX-domain sockets.
4层tcp, udp, unix-domain代理；

#### configuration

Defines a timeout for establishing a connection with a proxied server.

```bash
Syntax:	proxy_connect_timeout time;
Default:	proxy_connect_timeout 60s;
Context:	stream, server
```



Sets the timeout between two successive read or write operations on client or proxied server connections. If no data is transmitted within this time, the connection is closed. 

```bash
Syntax:	proxy_timeout timeout;
Default:	proxy_timeout 10m;
Context:	stream, server
```





### ngx_http_proxy_module

7层http负载；



#### configuration

Defines a timeout for establishing a connection with a proxied server. It should be noted that this timeout cannot usually exceed(超过) 75 seconds. 

```bash
Syntax:	proxy_connect_timeout time;
Default:	proxy_connect_timeout 60s;
Context:	http, server, location
```



Defines a timeout for reading a response from the proxied server. The timeout is set only between two successive read operations, not for the transmission of the whole response. If the proxied server does not transmit anything within this time, the connection is closed. 

```bash
Syntax:	proxy_read_timeout time;
Default:	proxy_read_timeout 60s;
Context:	http, server, location
```



Sets a timeout for transmitting a request to the proxied server. The timeout is set only between two successive write operations, not for the transmission of the whole request. If the proxied server does not receive anything within this time, the connection is closed. 

```bash
Syntax:	proxy_send_timeout time;
Default:	proxy_send_timeout 60s;
Context:	http, server, location
```





## Nginx As Http LB

问题：nginx用于反向代理 与 负载均衡 有什么区别？

> LB就是把多个proxy集成在一起，在众多backend server中选择一个server对client进行服务；
> 所以没有proxy就没有LB，LB是一堆具有相同功能、服务的server集群；



### Issue

#### nginx 504(Gateway timeout)

```bash
error.log: upstream timed out(110: connection timed out) while reading response header from upstream.
```

查看是否是http-proxy的send, read超时时间太短了；或者是被代理的业务内容出现比较耗时的代码段；

#### nginx 502(bad-gateway)

```bash
error.log connected failed.(111: connectionn refused) while connecting to upstream.
```

看样子是连不上被代理的服务；



## windows+nginx+php+sqlite

php解释器(php-cgi/php-fastcgi)与nginx是两个服务，它们之间的通信用socket, 或者unixsocket；

### Install

#### install nginx

modify nginx.conf

```bash
#set website service disk path.
location / {
    ..
}

#set website php FastCGI server env.
location ~ .php$ {
    ...
}
```



#### install php

generate php.ini

modify php.ini

```bash
set extension_dir
set cgi.fix_pathinfo
```

run php-cgi: php-cgi.exe -b 127.0.0.1:9000 -c php.ini



### php Dev

工程依赖的第三方php库应该存放于工程目录中，工程用include("*.php")引用



#### Issue

JpGraph Error This PHP installation is not configured with the GD library. Please recompile PHP with GD support to run JpGraph. 

```bash
open extension=php_gd2.dll in php.ini
```







