
## ReadMe

boost::thread的cb为函数对象boost::function

编译
```cpp
g++ tst.cpp -lboost_thread -lboost_system
```

## boost::thread
### thread cb无需参数
```cpp
boost::thread(ThreadFunction); //ThreadFunction是线程的启动函数。
```


### thread cb需要参数
```cpp
//法一
boost::thread(ThreadFunction, Arg1, Arg2, ...);

//法二：用绑定器
boost::thread(boost::bind(ThreadFunction, Arg1, Arg2 ...));
```

### thread cb是一个对象的成员函数
如果线程函数是一个对象的成员函数，可以用绑定器，
```cpp
//用绑定器：把类对象、成员函数绑在一块；
boost::thread(boost::bind(&MyClass::ThreadFunction, MyObject)); //MyClass是一个声明的类，MyObject是这个类的一个对象。
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
	group.join_all(); //等待所有线程执行结束
	system("pause");
	return 1 ;
}
```

## interrupt
