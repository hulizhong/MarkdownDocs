[TOC]

## ReadMe
讲解boost thread同步的一些知识；
外带boost lock机制；

对于mutex和lock，要明确一点，真正起到互斥作用的mutex，而lock只是协助mutex令我们在使用时更方便。

## Mutex
互斥量各类较多，如下：
[mutex](#mutex)
[shared\_mutex](#shared_mutex)
[timed\_mutex](#timed_mutex)
[recursive\_mutex](#recursive_mutex)

头文件
```cpp
#include <boost/thread/mutex.hpp>
```

### mutex
独占互斥量；
```cpp
boost::mutex mt;

mt.lock();
	//..临界区..
mt.unlock();
```

### shared\_mutex
共享互斥量；
```cpp
mt.shared_lock();
mt.shared_unlock();
```

可实现读写锁
```cpp
typedef boost::shared_lock<boost::shared_mutex> readLock;
typedef boost::unique_lock<boost::shared_mutex> writeLock;

boost::shared_mutex rwmutex;

void readOnly() {
	readLock  rdlock( rwmutex );
	/// do something
}

void writeOnly() {
	writeLock  wtlock( rwmutex );
	/// do something
}
```


### timed\_mutex
```cpp
mt.timed_lock();
```

### recursive\_mutex



## Lock
boost实现了各种lock来帮助mutex的使用；
它们的原理就是RAII：在lock的构造函数中调用T.lock()、析构函数中调用T.unlock()；和智能指针类似；
[lock\_guard](#lock_guard)
[unique\_lock](#unique_lock)
[shared\_lock](#shared_lock)
[upgrade\_lock](#upgrade_lock)

头文件
```cpp
#include <boost/thread/mutex.hpp>
#include <boost/thread/lock_guard.hpp>
```

### lock\_guard
其中构造它的互斥量可以是任意类型；
在其内部构造和析构函数分别自动调用 其存储对象的.lock(), .unlock()方法；
```cpp
boost::mutex mt;

{
	boost::lock_guard<boost::mutex> lock(mt); //lock_guard构造函数会调用mt.lock();
	...
} //lock_guard析构函数会调用mt.unlock();
```

### unique\_lock
独占锁；
其中构造它的互斥量可以是任意类型；
unique\_lock与lock\_guard原理相同，但是提供了更多功能：
> 可手动释放锁, ul.unlock()
> 配合boost::condition使用；（只能用此）

```cpp
boost::mutex mt;
{
	boost::unique_lock<boost::mutex> ul(mt);

}

ul.owns_lock(); //检查是否可获得互斥体
ul.lock(); //会一直等待，直到获得一个互斥体。
ul.try_lock() //则不会等待，但如果它只会在互斥体可用的时候才能获得，否则返回 false
ul.timed_lock() //等待一定的时间以获得互斥体
```

### shared\_lock
共享锁；
其中构造它的互斥类只能是共享类互斥量；


### upgrade\_lock



## Condition Variable
消费者
```cpp
boost::condition_variable cond;
boost::mutex mut;
bool data_ready; //等待的条件

void process_data();

void wait_for_data_to_process()
{
    boost::unique_lock<boost::mutex> lock(mut); //必须使用unique_lock
    while(!data_ready) //条件不满足，需要等待；
    {
        cond.wait(lock);
    }
    process_data();
}
```

生产者
```cpp
void retrieve_data();
void prepare_data();

void prepare_data_for_processing()
{
    retrieve_data(); //生产
    prepare_data();  //通知
    {
        boost::lock_guard<boost::mutex> lock(mut);
        data_ready=true;
    }
    cond.notify_one();
}
```

## call\_once
```cpp
#include <iostream>
#include <boost/thread.hpp>

using namespace std;
using boost::thread;
boost::once_flag once = BOOST_ONCE_INIT; // 注意这个操作不要遗漏了
void func() {
    cout << "Will be called but one time!" << endl;
}
void threadFunc() {
    //    func();
    boost::call_once(&func, once);
}
 
int main() {
    boost::thread_group threads;
    for (int i = 0; i < 5; ++i)
        threads.create_thread(&threadFunc);
    threads.join_all();
}
```
