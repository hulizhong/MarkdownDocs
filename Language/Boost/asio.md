

## ReadMe

boost asio即异步输入、输出。

基于io服务`boost::asio::io_service`抽象了异步数据处理，及io对象（用于初始化一特定的操作）；



## asio::io_service



## asio::ip::tcp::socket

网络数据收发。



## asio::steady_timer

老的计时器`boost::asio::deadline_timer`如下：

```cpp
void cb(const boost::system::error_code &ec) {
    //...
}

boost::asio::io_service ios;
boost::asio::deadline_timer timer(ios, boost::posix_time::seconds(5));
	//把timer这个io对象绑定到ios这个io服务上；
timer.async_wait(cb);  //启动一个异步操作并立即会返回；
	//也可以触发一个同步操作，即.wait();
ios.run();  //异步任务的控制权交给io_service，该api是阻塞运行的；
```

新的计时器`boost::asio::steady_timer`

```cpp
#include <boost/asio/steady_timer.hpp>
//typedef basic_waitable_timer<chrono::steady_clock> steady_timer;

using namespace boost::asio;
io_service io;
steady_timer st(io);
t1.expires_from_now(boost::chrono::milliseconds(5000));
steady_timer st(io, boost::chrono::milliseconds(5000));

st.cancel();
st.cancel_one();
st.expires_at();        //查看超时；
st.expires_from_now();  //查看超时；
st.expires_at(const time_point& expiry_time); //设置超时；
st.expires_from_now(const duration& expiry_time); //设置超时；
st.wait();
st.async_wait(BOOST_ASIO_MOVE_ARG(WaitHandler) handler);
```





