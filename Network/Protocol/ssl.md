[TOC]



## ReadMe

SSL_CTX_set_verify(ctx, SSL_VERIFY_PEER | SSL_VERIFY_FAIL_IF_NO_PEER_CERT, NULL);
SSL_CTX_set_cert_verify_callback(ctx, uuid, callback);

curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
​    libclurl是否验证对端
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);
​    libcurl验证对端证书的方法：0不检查，1是否有CN字段，2在1的基础上验证当前域名是否与CN一致。





## 版本

|          |             |                       |
| -------- | ----------- | --------------------- |
| Protocol | Published   | Status                |
| SSL 1.0  | Unpublished | Unpublished           |
| SSL 2.0  | 1995        | Prohibited in 2011    |
| SSL 3.0  | 1996        | Prohibited in 2015    |
| TLS 1.0  | 1999        | Deprecation suggested |
| TLS 1.1  | 2006        |                       |
| TLS 1.2  | 2008        |                       |
| TLS 1.3  | 2018        |                       |



## 握手

https://en.wikipedia.org/wiki/Transport_Layer_Security





协议选择？？



错误码：



### 问题

Case 1. 系统时间错误引起的证书不能用，注意这种情况！

> Could not validate certificate: current time .







SNI（Server Name Indication）是为了解决一个服务器使用多个域名和证书的SSL/TLS扩展。定义在RFC 4366。是一项用于改善SSL/TLS的技术，在SSLv3/TLSv1中被启用。它允许客户端在发起SSL握手请求时（具体说来，是客户端发出SSL请求中的ClientHello阶段），就提交请求的Host信息，使得服务器能够切换到正确的域并返回相应的证书。一句话简述它的工作原理就是，在连接到服务器建立SSL链接之前先发送要访问站点的域名（Hostname），这样服务器根据这个域名返回一个合适的证书。目前，大多数操作系统和浏览器都已经很好地支持SNI扩展，OpenSSL 0.9.8已经内置这一功能，据说新版的nginx也支持SNI。  --rtodo

ssl在 Web 服务(HTTPS)相关的扩展，如 SNI, NPN, ALPN。  --rtodo



