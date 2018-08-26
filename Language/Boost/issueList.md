

## 析构、关闭程序

### 类对象释放时mutex析构失败

程序跑起来，关闭时报以下错误
```cpp
class Demo
{
public:
	...
	void threadCb() {
		//not call the th.detach();
		//use mt
	}

	void stopThread() {
		//stop the th;
		//but not wait the th.	(Bug, need join() the th.)
	}

private:
	boost::thread th;
	boost::mutex mt;
};

int main()
{
	Demo d;
	d.stopThread();
}

Assertion failed: (!res), function ~mutex, file /path/include/boost/thread/pthread/mutex.hpp, line 111
tst.sh: line 16: 11959 Abort trap: 6
```

查看mutex.hpp:111行代码如下
```cpp
107         ~mutex()
108         { 
109           int const res = posix::pthread_mutex_destroy(&m);
110           boost::ignore_unused(res);
111           BOOST_ASSERT(!res);
112         }

 76     BOOST_FORCEINLINE int pthread_mutex_destroy(pthread_mutex_t* m)
 77     {                      
 78       return ::pthread_mutex_destroy(m);
 79     } 
```

查看pthread_mutex_destroy的手册
```bash
bash-3.2$ man pthread_mutex_destroy
SYNOPSIS
     #include <pthread.h>

     int pthread_mutex_destroy(pthread_mutex_t *mutex);

ERRORS
     The pthread_mutex_destroy() function will fail if:
     [EINVAL]           The value specified by mutex is invalid.
     [EBUSY]            Mutex is locked by another thread.
```

问题定位：
> 很有可能是类对象d调用stopThread()之后，线程th还未跑完要用到mutex；但对象d要释放了；
> 所以有出现boost::mutex析构的时候有问题；
>
> > 只需要在d.stopThread()的最后，连接下线程th就可以了！



## 其它

