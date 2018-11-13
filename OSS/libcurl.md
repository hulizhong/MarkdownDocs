[TOC]

## ReadMe
libcurl整合了很多协议的客户端开发，常用的是http/https.
```bash
root@~# dpkg -l | grep libcurl3:amd64 
pi  libcurl3:amd64                        7.26.0-1+wheezy18                  amd64        easy-to-use client-side URL transfer library (OpenSSL flavour)
```

## curl指令
```bash
curl -X POST url [-d ""]   #需要转义data中"，而用'则可避免
curl -X POST url [--data ""]

curl -X POST url --data @filename
curl -X POST url -d @filename

curl -X POST url -T filename  #相对直接--data来说：会有个100-continue，并不会判断文件内容类型；
curl -X POST url -T "{file1,file2}"  #多个文件
```

multipart怎么传？rwhy.
```bash
curl  -F "keyName=@fileName;type=text/plain" url  multipart/form-data数据上传
```

认证
```bash
curl -u username:password url

curl -k https://www.baidu.com
	-k, --insecure      Allow connections to SSL sites without certs, 忽略https认证；
```


其它
```bash
--limit-rate 1000B  #限速

-C #断点续传

-o newName, -O  #下载文件重命名

-x proxysever:proxyport  #代理

-H 'key:value'
	#加头, (Wrong: -H '"reqid":123',  Right: -H 'reqid:123')
	#多个头分开加；
```



## ssl相关

```cpp
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
    //libclurl是否验证对端
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);
    //libcurl验证对端证书的方法：0不检查，1是否有CN字段，2在1的基础上验证当前域名是否与CN一致。

////libopenssl中相关的api
SSL_CTX_set_verify(ctx, SSL_VERIFY_PEER | SSL_VERIFY_FAIL_IF_NO_PEER_CERT, NULL);
SSL_CTX_set_cert_verify_callback(ctx, uuid, callback);
```





## bug经验

Handshake Failure (40)
> client没把自己的证书发给server，导致server端握手不成功；

100-continue
> libcurl在不同版本里逻辑不同，有的版本libcurl会在POST数据大于1024字节的时候发送100-continue请求，有的版本则不会。  
> 要注意，server是否支持100-continue，或者由100-continue引起的server端错误。


