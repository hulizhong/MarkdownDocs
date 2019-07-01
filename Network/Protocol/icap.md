

[TOC]



## ReadMe

icap协议相关；



## icap protocol

ICAP (Internet Content Adaptation Protocol) <font color=gree>是tcp之上的一种应由协议</font>，本质上是在http/ftp message上执行RPC(远程过程调用)的一种轻量级的协议。即ICAP Client可以把http message（request, response, request+response）传给ICAP Server,  然后ICAP Server对内容进行某种变换或者其他处理(“匹配”)。

rfc doc. https://tools.ietf.org/html/rfc3507#section-4.8



### request mode

| 请求模式 | 描述                         |
| -------- | ---------------------------- |
| REQMOD   | for Request Modification     |
| RESPMOD  | for Response Modification    |
| OPTIONS  | to learn about configuration |

icap server适配之后可能会返回如下：
??



### icap header

Allow: 204
如果client包含了此头，那么server可以返回`204 no content`来标志消息不需要修改。



### icap status

**ICAP status codes:**

**(1yz) Informational codes:**

| Code | Description                  |
| ---- | ---------------------------- |
| 100  | Continue after ICAP preview. |

**(2yz) Success codes:**

| Code | Description              |
| ---- | ------------------------ |
| 204  | No modifications needed. |

**(4yz) Client error codes:**

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 400  | Bad request.                                                 |
| 404  | ICAP Service not found.                                      |
| 405  | Method not allowed for service (e.g., RESPMOD requested for service that supports only REQMOD). |
| 408  | Request timeout. ICAP server gave up waiting for a request from an ICAP client. |

**(5yz) Server error codes:**

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 500  | Server error. Error on the ICAP server, such as "out of disk space". |
| 501  | Method not implemented. This response is illegal for an OPTIONS request since implementation of OPTIONS is mandatory. |
| 502  | Bad Gateway. This is an ICAP proxy and proxying produced an error. |
| 503  | Service overloaded. The ICAP server has exceeded a maximum connection limit associated with this service; the ICAP client should not exceed this limit in the future. |
| 505  | ICAP version not supported by server.                        |



### an icap request pcap

```bash
REQMOD icap://172.18.200.110:1344/dlp ICAP/1.0   ## icap协议头。------注意端口、服务名。
x-client-ip: 172.18.0.112
connection: keep-alive
allow: 204
Encapsulated: req-hdr=0, req-body=259

POST HTTP://172.16.0.1/post.php HTTP/1.1  ## http协议头。
Host: 172.16.0.1
User-Agent: curl/7.45.0
Accept: */*
Proxy-Connection: Keep-Alive
Content-Length: 463
Expect: 100-continue
Content-Type: xx

http body......

ICAP/1.0 204 Unmodified   ## icap server响应。
Server: C-ICAP/0.4.2
Connection: keep-alive
ISTag: CI0001-XXXXXXXXX
```

