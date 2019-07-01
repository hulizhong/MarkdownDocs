[TOC]

## ReadMe
讲解boost thread同步的一些知识；
外带boost lock机制；-------**其实表现出来的这些mutex, lock之类的都是各平台基础库上封装而来的。（Linux下是pthread）**。

对于mutex和lock，要明确一点，真正起到互斥作用的mutex，而lock只是协助我们更方便的使用mutex。



### outline

| 大类     | 子类 | 类型                   | 说明                                                      |
| -------- | ---- | ---------------------- | --------------------------------------------------------- |
| 互斥类   | 独占 | mutex                  |                                                           |
|          |      | timed_mutex            |                                                           |
|          | 递归 | recursive_mutex        | 同一线程可递归加                                          |
|          |      | recursive_timed_mutex  |                                                           |
|          | 共享 | shared_mutex           | 能提供多写单写的功能                                      |
|          |      |                        |                                                           |
|          |      |                        |                                                           |
| 锁       | 独占 | unique_lock            | 内部mutex可任意；                                         |
|          |      | lock_guard             | 同于unique_lock，但功能更丰富，如提前unlock, 加超时时间。 |
|          | 共享 | shared_lock            | 只能是shared_mutex.                                       |
|          |      | upgrade_lock           |                                                           |
|          |      |                        |                                                           |
| 条件变量 |      | condition_variable     | 要求传入的lock是boost::unique_lock\<boost::mutex>         |
|          |      | condition_variable_any | 接受任意的锁、互斥量                                      |

**备注：**还有try_mutex, recursive_try_mutex是为了于以前版本兼容，其实是等于mutex, recursive_mutex的。





## Mutex
**互斥量**各类较多，如下：
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
共享互斥量，提供如下功能；
```cpp
mt.lock_shared();  //及其它一些变种。
mt.unlock_shared();
mt.lock();  //及其它一些变种。
mt.unlock();
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
**范围锁**，其中构造它的互斥量可以是任意类型；
其构造、析构内分别自动调用 其存储对象的`.lock(); .unlock()`方法；

```cpp
boost::mutex mt;

{
	boost::lock_guard<boost::mutex> lock(mt); //lock_guard构造函数会调用mt.lock();
	...
} //lock_guard析构函数会调用mt.unlock();
```

### unique\_lock
**独占锁**，其中构造它的<font color=gree>互斥量可以是任意类型</font>；unlock
其析构内自动调用 其存储对象的<font color=gree>`unlock()`</font>方法；构造内<font color=gree>`unlock_and_lock_shared, lock, ..`</font>;
`unique_lock`与`lock_guard`原理相同，但是提供了更多功能：

> 可手动释放锁, ul.unlock()
> 配合boost::condition使用；（只能用此）

```cpp
boost::mutex mt;
{
	boost::unique_lock<boost::mutex> ul(mt);

}

ul.owns_lock(); //检查是否可获得互斥体
ul.lock(); //会一直等待，直到获得一个互斥体。
ul.try_lock(); //则不会等待，但如果它只会在互斥体可用的时候才能获得，否则返回 false
ul.timed_lock(); //等待一定的时间以获得互斥体

lfv.swap(rv);   //可以交换锁、锁的状态。？？？ ----这个神奇了。rabin.
	//因为没有make_unique_lock之类的函数，只能先构造个临时的，再进行交换？？
```

#### scoped_lock

`boost::mutex::scoped_lock` is a `typedef` for `boost::unique_lock<boost::mutex>`, not `lock_guard`. `scoped_lock` is not available in C++0x.

```cpp
//boost/thread/pthread/mutex.hpp
#if defined BOOST_THREAD_PROVIDES_NESTED_LOCKS
        typedef unique_lock<mutex> scoped_lock;
        typedef detail::try_lock_wrapper<mutex> scoped_try_lock;
#endif
```



### shared\_lock

**共享锁**，其中构造它的互斥类<font color=gree>只能是共享类互斥量</font>；
其析构内自动调用 其存储对象的`.unlock_shared()`方法；


### upgrade\_lock

**升级锁**，对shared_lock的扩展使用，可以升级共享权限为独占权限。 ---这个牛逼，怎么实现的呢？
升级的时候，如果没有其它线程与之共享，那么立即会升级成功。此后必须使用`unlock()`来释放独占锁。



## Condition Variable

### condition_variable

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



### condition_variable_any







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
