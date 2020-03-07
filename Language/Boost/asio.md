

## ReadMe

boost asio即异步输入、输出。

> 异步数据处理就是指，任务触发后不需要等待它们完成。 相反，Boost.Asio 会在任务完成时触发一个应用。 异步任务的主要优点在于，在等待任务完成时不需要阻塞应用程序，可以去执行其它任务。

在包含 Asio 头文件之前，定义宏 `BOOST_ASIO_NO_DEPRECATED`，这样在编译时，Asio 就会剔除那些已经过时的接口。



boost asio基于以下两点（<font color=red>编译带-lboost_system</font>）

- io服务（抽象了异步数据处理）；
    - boost::asio::io_service
- io对象（用于初始化一特定的操作）；
    - boost::asio::ip::tcp::socket
    - boost::asio::steady_timer
    - ...





## io service

功能主要有如下：

```cpp
/* 把io服务跑起来，控制权交由系统。
	1. 在同一个线程内触发与其关联的所有io对象。
	2. 如果与其关联的io对象，异步通知回调中存在资源竞争时，那么需要注意其多线程模式下的竞争关系。
*/
io.run();

/* 模式1：多线程跑同一io service，那么当io对象结束时，handler可以被多个线程中选择空闲的线程来进行执行，否则是排队执行的。 */
io_service io;
void routine(void) {
    io.run();
}
boost::thread t1(routine);
boost::thread t2(routine);
t1.join();
t2.join();

/* 模式2：用多个io服务来实现io对象与io服务的1对1关联关系。 */
io_service io1, io2;
void routine1(void) {
    io1.run();
}
void routine2(void) {
    io2.run();
}
```

### asio::io_service



## io object

功能主要有如下：

```cpp
asio::io_service io;
obj(io, ..); //创建io对象时，一般需要传入或者绑定一个io服务。
obj.wait();  //io对象的同步等待，会阻塞直到io完成。
obj.asyn_wait(hander, ..); //io对象的异步等待，不会阻塞住、等io完成时handler会被io服务调用。
	//handler一般为结果处理函数，即io成功、失败了会怎么操作。
```



### asio::ip::tcp

网络数据收发。

```cpp
boost::asio::io_service io; 
tcp::resolver resolver(io); {
    tcp::resolver::query query("www.highscore.de", "80"); //一次查询
    resolver.async_resolve(query, resolve_handler);
}
tcp::socket sock(io);  {
    boost::asio::ip::tcp::resolver::iterator it;
    sock.async_connect(*it, connect_handler);
}

tcp::endpoint endpoint(boost::asio::ip::tcp::v4(), 80); 
tcp::acceptor acceptor(io, endpoint);  {
    listen();
    async_accept();
}
tcp::socket sock(io); 
```



### asio::steady_timer

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





## 扩展自己的异步操作

如何扩展自己的异步操作，有如下几个过程

- 一个派生自`boost::asio::basic_io_object`的类，表示新的io对象。
- 一个派生自`boost::asio::io_service::service`的类，表示一个服务，被注册为io服务。
- 一个不派生自任何其它类的类，表示该服务的具体实现。

io对象如下：
在 I/O 对象的内部，可以通过 service 引用来访问相应的服务，通常的访问就是将方法调用前转至该服务。
服务的具体实现是通过 implementation 属性来访问的。

```cpp
#include <boost/asio.hpp> 
#include <cstddef> 

template <typename Service> 
class basic_timer 
  : public boost::asio::basic_io_object<Service> 
{ 
  public: 
    explicit basic_timer(boost::asio::io_service &io_service) 
      : boost::asio::basic_io_object<Service>(io_service) 
    { 
    } 

    void wait(std::size_t seconds) 
    { 
      return this->service.wait(this->implementation, seconds); 
    } 

    template <typename Handler> 
    void async_wait(std::size_t seconds, Handler handler) 
    { 
      this->service.async_wait(this->implementation, seconds, handler); 
    } 
}; 
```

io服务如下：
静态ID用以识别服务。
construct, destroy, shutdown_service接口实现。

```cpp
#include <boost/asio.hpp> 
#include <boost/thread.hpp> 
#include <boost/bind.hpp> 
#include <boost/scoped_ptr.hpp> 
#include <boost/shared_ptr.hpp> 
#include <boost/weak_ptr.hpp> 
#include <boost/system/error_code.hpp> 

template <typename TimerImplementation = timer_impl> 
class basic_timer_service 
  : public boost::asio::io_service::service 
{ 
  public: 
    static boost::asio::io_service::id id;

    explicit basic_timer_service(boost::asio::io_service &io_service) 
      : boost::asio::io_service::service(io_service), 
      async_work_(new boost::asio::io_service::work(async_io_service_)), 
      async_thread_(boost::bind(&boost::asio::io_service::run, &async_io_service_)) 
    { 
    } 

    ~basic_timer_service() { 
      async_work_.reset(); 
      async_io_service_.stop(); 
      async_thread_.join(); 
    } 

    typedef boost::shared_ptr<TimerImplementation> implementation_type; 

    void construct(implementation_type &impl) { 
      impl.reset(new TimerImplementation()); 
    } 

    void destroy(implementation_type &impl) { 
      impl->destroy(); 
      impl.reset(); 
    } 

    void wait(implementation_type &impl, std::size_t seconds) { 
      boost::system::error_code ec; 
      impl->wait(seconds, ec); 
      boost::asio::detail::throw_error(ec); 
    } 

    template <typename Handler> 
    class wait_operation { 
      public: 
        wait_operation(implementation_type &impl, boost::asio::io_service &io_service, std::size_t seconds, Handler handler) 
          : impl_(impl), 
          io_service_(io_service), 
          work_(io_service), 
          seconds_(seconds), 
          handler_(handler) 
        { 
        } 

        void operator()() const { 
          implementation_type impl = impl_.lock(); 
          if (impl) { 
              boost::system::error_code ec; 
              impl->wait(seconds_, ec); 
              this->io_service_.post(boost::asio::detail::bind_handler(handler_, ec)); 
          } 
          else { 
              this->io_service_.post(boost::asio::detail::bind_handler(handler_, boost::asio::error::operation_aborted)); 
          } 
      } 

      private: 
        boost::weak_ptr<TimerImplementation> impl_; 
        boost::asio::io_service &io_service_; 
        boost::asio::io_service::work work_; 
        std::size_t seconds_; 
        Handler handler_; 
    }; 

    template <typename Handler> 
    void async_wait(implementation_type &impl, std::size_t seconds, Handler handler){ 
      this->async_io_service_.post(wait_operation<Handler>(impl, this->get_io_service(), seconds, handler)); 
    } 

  private: 
    void shutdown_service() { 
    } 

    boost::asio::io_service async_io_service_; 
    boost::scoped_ptr<boost::asio::io_service::work> async_work_; 
    boost::thread async_thread_; 
}; 

template <typename TimerImplementation> 
boost::asio::io_service::id basic_timer_service<TimerImplementation>::id; 
```

具体的实现类，如下

```cpp
#include <boost/system/error_code.hpp> 
#include <cstddef> 
#include <windows.h> 

class timer_impl 
{ 
  public: 
    timer_impl() : handle_(CreateEvent(NULL, FALSE, FALSE, NULL)) { 
    } 

    ~timer_impl() { 
      CloseHandle(handle_); 
    } 

    void destroy() { 
      SetEvent(handle_); 
    } 

    void wait(std::size_t seconds, boost::system::error_code &ec) { 
      DWORD res = WaitForSingleObject(handle_, seconds * 1000); 
      if (res == WAIT_OBJECT_0) 
        ec = boost::asio::error::operation_aborted; 
      else 
        ec = boost::system::error_code(); 
    } 

private: 
    HANDLE handle_; 
}; 
```





