[toc]

## ReadMe

对于mutex和lock，要明确一点，真正起到互斥作用的mutex，而lock只是协助mutex令我们在使用时更方便。

## mutex
### boost::mutex
```cpp
.lock()
.unlock()
```

### boost::timed_mutex
```cpp
.timed_lock() 
```

### boost::recursive_mutex

### boost::shared_mutex


## lock
### boost::lock_guard
在其内部构造和析构函数分别自动调用 其存储对象的.lock(), .unlock()方法；
```cpp
boost::mutex mutex;
boost::lock_guard<boost::mutex> lock(mutex);
```

### boost::unique_lock
比lock_guard更为灵活，通过多个构造函数来提供不同的方式获得互斥体。
> 可手动释放锁, .unlock()
> 只能用此配合boost::condition

```cpp
boost::mutex mutex;
boost::unique_lock<boost::mutex> lk(mutex);
lk.owns_lock(); //检查是否可获得互斥体
lk.lock(); //会一直等待，直到获得一个互斥体。
lk.try_lock() //则不会等待，但如果它只会在互斥体可用的时候才能获得，否则返回 false
lk.timed_lock() //等待一定的时间以获得互斥体
```

### boost::shared_lock

### boost::upgrade_lock

## 条件变量

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

## call_once
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
