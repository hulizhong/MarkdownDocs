
## ReadMe

线程对象boost::thread
> 不可复制，但可转移；
> 对象析构了，线程还存在，只是为detached状态；
>
> > 亦可调用detach()来让线程与线程对象分离；
>

callback的形式
> boost线程的cb为函数对象boost::function，而非传统的函数指针；

boost线程退出的正常手法是
> 在线程函数中设置很多检查点，当检测到被置退出标志时马上退出；（参见interrupt）

编译
```cpp
g++ tst.cpp -lboost_thread -lboost_system
```

## Thread Create
### cb was NonClassMemFun
cb不需要参数
```cpp
void fun1()
{  
	cout << "thread.cb fun1" << endl;
}  
 
int main(int argc, char* argv[])
{  
	//无需参数
		//1、函数可取地址符、亦可不取；
	boost::thread t1(&fun1);
	boost::thread t1(fun1);
	t1.join();
 
	return 0;
}
```

cb需要参数
```cpp
void fun2(string str)
{   
	cout << "thread.cb " << str << endl;
}   
      
int main(int argc, char* argv[])
{   
	//需要参数
		//法一：直接展开
	boost::thread t1(fun2, "with arg.");
	boost::thread t1(&fun2, "with arg.");
		//法二：用绑定器
	boost::thread t1(boost::bind(fun2, "with arg."));
	boost::thread t1(boost::bind(&fun2, "with arg."));
	t1.join();

	return 0;
}                      
```

### cb was ClassMemberFun
- 线程在哪里创建
	- 类内创建
		- cb是否为静态成员函数
			- 静态：不绑定this
			- 非静态：需要类名声明cb, 绑定this
	- 类外创建
		- cb是否为静态成员函数
			- 静态：需要类名声明cb, 不绑定对象
			- 非静态：需要类名声明cb, 绑定对象


类内创建
```cpp
class HelloWorld
{       
public: 
	static void helloS() {
		std::cout << "static thread cb" << std::endl;
	}   
	static void startS() {
		//cb为静态成员函数
			//1、无需类声明，无需绑定this
		boost::thread thrd(helloS);
		boost::thread thrd(&helloS);
		boost::thread thrd(HelloWorld::helloS);
		boost::thread thrd(&HelloWorld::helloS);
		thrd.join();
	}   

	void hello() {
		std::cout << "nonstatic thread cb" << std::endl;
	}   
	void start() {
		//cb为非静态成员函数
			//1、需要类声明函数，需要绑定this
		boost::thread thrd(&HelloWorld::hello, this);
		boost::thread thrd(boost::bind(&HelloWorld::hello, this));
		thrd.join();
	}   
};      
          
int main(int argc, char* argv[])
{       
	HelloWorld::startS();
	HelloWorld obj;
	obj.start();
	return 0;
}
```


类外创建
```cpp
class HelloWorld
{   
public:
	static void helloS(string arg) {
		std::cout << "static thread cb " << arg << std::endl;
	} 
	void hello(string arg) {
		std::cout << "nonstatic thread cb " << arg << std::endl;
	} 
};  
  
int main(int argc, char* argv[])
{   
	HelloWorld obj;
	//静态成员函数
		//1、不需要绑定对象；
		//2、成员函数可取地址符、亦可不取；
	boost::thread t(HelloWorld::helloS, "With arg.");
	boost::thread t(&HelloWorld::helloS, "With arg.");
	boost::thread t(boost::bind(HelloWorld::helloS, "With arg."));
	boost::thread t(boost::bind(&HelloWorld::helloS, "With arg."));

	//非静态成员函数
		//1、需要绑定对象；
		//2、成员函数必须取地址符；
	boost::thread t(&HelloWorld::hello, obj, "With arg.");
	boost::thread t(boost::bind(&HelloWorld::hello, obj, "With arg."));
	t.join();
	return 0;
}
```

## Thread Usage
线程的使用；

### API
睡眠
```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
boost::this_thread::sleep(boost::posix_time::seconds(5));
boost::this_thread::sleep(boost::posix_time::milliseconds(100));

boost::thread::sleep(boost::get_system_time() + boost::posix_time::seconds(1));

#include <boost/chrono.hpp>
boost::this_thread::sleep_for(boost::chrono::milliseconds(100));
```

```cpp
boost::this_thread::get_id();

boost::thread::hardware_concurrency(); //返回cpu的线程数；

{
	//当线程对象t析构时，线程变为detached, 但线程很可能并未结束；
	boost::thread t(fun);
	t.join(); //阻塞调用：它可以暂停当前线程，直到t运行结束。
	t.timed_join();
}
```

## boost::thread\_group
用于管理一组线程（如同线程池一样），内部使用std::list；
```cpp
#include <boost/thread/thread.hpp>
#include <boost/bind.hpp>

int fun()
{
	boost::thread_group group ;
	for (int num = 0 ; num < 10 ; ++num)
	{
		//create_thread()是一个工厂函数，可以创建thead对象并运行线程，同时加入内部的list
		group.create_thread(boost::bind(&runchild , num));
		group.create_thread(boost::bind(dummy , num));
	}
	//add_thread
	group.add_thread(new boost::thread(f2));
	group.join_all(); //等待所有线程执行结束
	system("pause");
	return 1 ;
}
```

## Interrupt
在一个线程中调用interrupt()，那么会在被设置线程内产生一个中断异常；（boost::thread_interrupted异常）
但是这个中断只能在中断点被检测到；
线程只会在中断点才会去检测自己是否被中断过，从而决定是否抛出boost::thread_interrupted异常；
```cpp
void thread() 
{ 
	try { 
		//...
	} 
	catch (boost::thread_interrupted&) 
	{
		//...
	} 
} 
```

### Interrupt Point
Boost.Thread定义了一系列的中断点如下：
```cpp
boost::thread::join()
boost::thread::timed_join()

boost::condition_variable::wait()
boost::condition_variable::timed_wait()
boost::condition_variable_any::wait()
boost::condition_variable_any::timed_wait()

boost::thread::sleep()
boost::this_thread::sleep()

boost::this_thread::interruption_point()
```

### Disable/Enable Interrupt
以下类型是通过RIIA来实现的
```cpp
//如下类型变量，到该变量析构间的代码不能被中断；
boost::this_thread::disable_interruption di;

//如下类型变量，到该变量析构间的代码能被中断；
boost::this_thread::restore_interruption ri(di);
```

## Thread Local Storage
TLS（线程本地存储）的变量可以被看作是一个只对某个特定线程而非整个程序可见的全局变量。
```cpp
//定义一个bool类型的tls变量，它的行为就像一个普通的指针（->, *）；
boost::thread_specific_ptr<bool> tls;
```


