[TOC]

## ReadMe
libcurl整合了很多协议的客户端开发，常用的是http/https.
```bash
root@~# dpkg -l | grep libcurl3:amd64 
pi  libcurl3:amd64                        7.26.0-1+wheezy18                  amd64        easy-to-use client-side URL transfer library (OpenSSL flavour)
```



## Code

### ssl verify

```cpp
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
    //libclurl是否验证对端
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);
    //libcurl验证对端证书的方法：0不检查，1是否有CN字段，2在1的基础上验证当前域名是否与CN一致。

////libopenssl中相关的api
SSL_CTX_set_verify(ctx, SSL_VERIFY_PEER | SSL_VERIFY_FAIL_IF_NO_PEER_CERT, NULL);
SSL_CTX_set_cert_verify_callback(ctx, uuid, callback);
```



### custom curl exception

```cpp
class CurlException : public std::exception
{
public:
	CurlException(const std::string &msg) : mMsg(msg) {}
	CurlException(const CURLcode &err) {
		mMsg = curl_easy_strerror(err);
	}
	const char* what() throw() {
		return mMsg.c_str();
	}
private:
	std::string mMsg;
};
```





### option table

设置选项的api

```cpp
curl_easy_setopt(curl, option_name, option_value);
```

选项列表如下：

| option name            | option value | option des               |
| ---------------------- | ------------ | ------------------------ |
| curlopt_writefunction  |              | 响应数据写回调函数       |
| curlopt_writedata      |              | 响应数据写回调函数的参数 |
|                        |              |                          |
| curlopt_url            |              |                          |
| curlopt_post           |              |                          |
| curlopt_postfilelds    |              |                          |
| curlopt_noprogress     |              |                          |
| curlopt_connecttimeout |              |                          |
| curlopt_timeout        |              |                          |
|                        |              |                          |
| curlopt_ssl_verifypeer |              |                          |
| curlopt_cainfo         |              |                          |
| curlopt_ssl_verifyhost |              |                          |
|                        |              |                          |
|                        |              |                          |
|                        |              |                          |
|                        |              |                          |
|                        |              |                          |
|                        |              |                          |





## bug经验

Handshake Failure (40)
> client没把自己的证书发给server，导致server端握手不成功；

100-continue

> libcurl在不同版本里逻辑不同，有的版本libcurl会在POST数据大于1024字节的时候发送100-continue请求，有的版本则不会。  
> 要注意，server是否支持100-continue，或者由100-continue引起的server端错误。


