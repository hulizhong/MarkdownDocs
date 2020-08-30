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



### libcurl & multi-thread

不能多个线程共享一个curl对象，如下：

```cpp
CURL *curl = curl_easy_init();
// use curl in one thread.
curl_easy_cleanup( curl );
```

如下初始化函数线程不安全（不支持多线程重入）：

```cpp
curl_global_init();
curl_global_cleanup();
```

多线程下域名的超时解析，如下：---rwhy....

```cpp
curl_easy_setopt(curl, CURLOPT_CONNECTTIMEOUT, 30L); //依赖alarm + siglongjmp，多线程会不安全。
curl_easy_setopt(curl, CURLOPT_NOSIGNAL, 1L); //禁用alarm超时，但这样就没有超时机制了。可使用替换技术c-ares。
```

多线程下可以共享dns解析缓存（老接口是curlopt_dns_use_global_cache，已被如下接口替换），如下：

```cpp
curl_share_setopt(curl, CURLSHOPT_SHARE, CURL_LOCK_DATA_DNS);
curl_easy_setopt(curl, CURLOPT_SHARE, share_handle);
curl_easy_setopt(curl, CURLOPT_DNS_CACHE_TIMEOUT, 60 * 5);
```

