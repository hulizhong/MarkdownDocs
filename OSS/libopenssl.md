[TOC]

## ReadMe
一般都是client用来验证server是不是自己想要连接的server（是真的server）；（单向认证）
双向认证，增加了server对client的认证；
```bash
libgnutls-openssl27:amd64             2.12.20-8+deb7u5                   amd64        GNU TLS library - OpenSSL wrapper
```

认证原理，主要是从2个方面
> 对端的证书不是伪造（用知名CA证书来校验）
> 证书属于将要连接的对端（cn与connection-dstHost对比）

认证过程
> 客户端加载知名CA机构的证书，连接服务器，客户端用CA证书对服务端证书进行校验；


## 设置认证回调
用以下函数进行认证业务（回调函数）的注册
```cpp
//针对ssl_ctx的
void SSL_CTX_set_verify(SSL_CTX *ctx, int mode, int (*verify_callback)(int, X509_STORE_CTX *)); 
//针对ssl的
void SSL_set_verify(SSL *s, int mode, int (*verify_callback)(int, X509_STORE_CTX *)); 
```

问题：libopenssl中ssl, ssl_ctx分别对应什么？？


## openssl 指令
生成 private/public key 对
```bash
openssl genrsa -out private.key 1024
openssl rsa -in private.key -pubout -out public.key
```

用s_server, s_client测试连接（连接建立之后传数据）
```bash
openssl s_server -accept 12345 -CAfile ca.crt -cert server.pem -key server.key -Verify 5 
	# Verify是强制认证client，且client证书签发机构的级数；
openssl s_client -connect 127.0.0.1:12345 -CAfile ca.crt -cert client.pem -key client.key

#还可以带个参数：-pass file:password.txt 这是什么意思？？？
```

校验证书是否是此CA机构签发
```bash
#openssl verify -CApath CApath alice\demoCA\cacert.pem
openssl verify -CAfile ca.pem  untrust.pem
```

读取x509证书的内容
```bash
openssl x509 -in input.crt -noout -text
```

## 证书的制作

### 证书的形式
#### pfx
此种是证书与key合在一块的，常用于windows或者java上，可以单独拆分出证书、KEY
```bash
openssl pkcs12 -in C:/as-esl99q0huyz1_w3svc1_cert.pfx -out c:/pkey.key

打开pkey.key，只保留以下内容（类似）
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,26D4D5A7FF139423
 
5/eW/r0mSMWbDqyL1wnCZFsgBNggi8UmAtoFwSwN+SS9vrsyNxu8hwIkgDsFIHRw
zFNo5JIgCBd+R9T8uZfa+ejd7FKdnsq3P+1xGyA52jVITh3wwAmHTj/0QNZOyxgO
Q4U2wb52ApSMnNugfkJKxtpBTwitum8satKMH+MvXoc967Gz/YmHtphDzc4xg3Zl
ETKTY9H3fIvNQ8swlZUfb0aJ43NRsPZkbQHeB1gJtlldygS/CvXpKn/dx6pS0pnM
mK8hSsqc1x7SB0vK/SpNtVRE+XHD5Fp0QlhaII/8b6W7xESUcGQLTfwgi4UGwUth
PpWsw8PL6T3NfC4vfzskMdrp0t72mLDYLNFkUwSvHGTodWRA7Xnjpfy2++GBtRkq
xQ4JL18Wfu8TpX0CUxw3vxI6U76c4zD4XBLh9qUJUEwkEGIm7F7w21U4exiXDxJg
kQ4QppSlwq+ByG6NynOobY3O/dCLH6zrUt08hneDqutazwIZ8vN+bzJs9vmu2TAA
g9jIoomnMBpw+dpUoxVTSmWiO9KAzyCYjpTU5H6Vu/8buQU/rj01U3T5+XTKuJsj
D4/m+Idq3fjgMVsqmkloJr1GE4nz5hMMKr2a47v3cp+sgIZIcbHGSwXW1CM09bC5
YWRMkxbsLvwHKq0lcLQ5fmJINF3DIHFevJvEyh9oa9nlESz9lREidli7y+te7otZ
fXbYO77eIRbe0pzp5VDuEApJeAfDHGjmI8j2ZzNU5BY57Q1YnHoBRJynrcKJbYvt
QwTg56hwHJ8OEdX6eGSl1S9YsA8ivJ3CPXPUyPnP7kRMOdNNjng1Qg==
-----END RSA PRIVATE KEY-----
```
