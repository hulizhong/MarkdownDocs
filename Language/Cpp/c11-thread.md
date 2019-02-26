[TOC]

## ReadMe
c11线程相关
https://thispointer.com/c11-multithreading-tutorial-series/



## Compile

Compilers Required:  
> Linux: gcc 4.8.1 (Complete Concurrency support)
> Windows: Visual Studio 2012 and MingW

How to compile on Linux:
> g++ –std=c++11 sample.cpp -lpthread

```cpp
int main()
{
	std::thread th(&thCB);
	th.join();
}
```
```bash
root@c11# gcc --version
gcc (Debian 4.7.2-5) 4.7.2
root@c11#
root@c11# g++ thd.cpp  -std=c++11
root@c11# ./a.out 
terminate called after throwing an instance of 'std::system_error'
  what():  Operation not permitted
Aborted
root@c11#
root@c11# g++ thd.cpp  -std=c++11 -lpthread  #为毛还要加个-lpthread
root@c11# ./a.out 
```



## Create Threads

std::thread thObj(CALLBACK); 一个std::thread对象代表一个线程；
CALLBACK可如下：
> Function Pointer
> Function Objects
> Lambda functions

每个std::thread关联了一个id
```cpp
void threadFun() {
	std::thread::id id;
	id = std::this_thread::get_id(); //std::this_thread只适用于当前线程。
		//是不是只能在thread fun中调用；
}

{
	std::thread th;
	std::thread::id id = th.get_id(); //用threadobj调用；
}
```



### watchout

默认只需要把传入线程执行体的参数（<font color=red>突破boost::thread的有限个参数</font>），作为std::thread的构造参数即可；
这些参数默认会被拷贝到新线程；

Don’t pass addresses of variables from local stack to thread’s callback function.
Because it might be possible that local variable in Thread 1 goes out of scope but Thread 2 is still trying to access it through it’s address.
In such scenario accessing invalid address can cause unexpected behaviour.

```cpp
{
	int v;
	std::thread th(cb, &v);  //不要传递局部变量的指针；
}
```

Similarly be careful while passing pointer to memory located on heap to thread.
Because it might be possible that some thread deletes that memory before new thread tries to access it.
```cpp
{
	int *v = new int(3);
	std::thread th(cb, &v);  //堆中的地址亦不可传，小心被其它线程delete.
	delete v;
}
```

因为线程执行体的参数都是拷贝到新线程栈上的，所以引用传参需要特殊处理；
```cpp
{
	int v;
	std::thread th(cb, std::ref(v));  //引用传参的方法，默认是按值传递（即拷贝）。
}
```

如果线程执行体是个类成员函数，那么
std::thread的第一参数是类成员函数的地址，第二参数是类对象的地址；

```cpp
class T {};
T t;
std::thread th(&T::cb, &t);  //需要传送对应对象的地址。
```



## Joining and Detaching Threads

join thread
```cpp
std::thread th(funcPtr); 
th.join(); //当前线程阻塞，等待th结束
```

Detached threads are also called daemon / Background threads.
```cpp
std::thread th(funPtr);
th.detach(); //从此刻开始，th对象并未关联到线程；
```

### watchout
Never call join() or detach() on std::thread object with no associated executing thread.
```cpp
std::thread th(funcPtr); 
#if 0
	th.join();
	th.join(); //th不再关联一个线程了，会导致crash；
#else
	th.detach();
	th.detach(); //同理
#endif

//所以才会有如下防御性代码
if (th.joinable())
	th.join();
if (th.joinable()) //校验同于join.
	th.detach();
```

Never forget to call either join or detach on a std::thread object with associated executing thread.
包含正常退出、异常退出；（建议使用RAII技术）
```cpp
int main()
{
	std::thread th(&thCB);
}
root@c11# g++ thd.cpp  -std=c++11 -lpthread
root@c11# ./a.out                          
terminate called without an active exception
Aborted

//所以就有如下防御性代码
class ThreadRAII
{
	std::thread &mTh;
public:
	ThreadRAII(std::thread &th) : mTh(th) {}
	~TrheadRAII() {
		if (mTh.joinable())
			mTh.join();
	}
}
{
	std::thread th(thCb);
	ThreadRAII thRAII(th);
}
```



## Data Sharing and Race Conditions

### race condition
竞争条件：多个线程或进程并发读、写同一共享数据且执行结果与访问的特定顺序有关。
```cpp
void cb(int v)
{
	v++;
}

{
	int v = 0;
	std::thread th1(cb, std::ref(v);
	std::thread th2(cb, std::ref(v);
	std::cout << v << std::endl; //怎么会是1呢？
}
```
|th1|th2|
|---|---|
|load v to register = 0||
|| load v to register = 0|
|add register = 1||
||add register = 1|
|update v with register = 1||
||update v with register = 1|



### mutex, fix race condition
each thread needs to lock a mutex before modifying or reading the shared data and after modifying the data each thread should unlock the mutex.
```cpp
#include <mutex>
std::mutex mt;
void cb()
{
	mt.lock();
	//...
	mt.unlock();
}

std::thread th1(cb);
std::thread th2(cb);
```

但当th1最后忘了unlock()，并退出了。这时怎么办？
这样的执行序列，当再次lock()这个mutex之后有可能带来异常；应该用`lock guard`来避免这种场景；
lock guard是类模板，实现了mutex的RAII，在构造时进行lock()，在析构时进行unlock()；

```cpp
std::mutex mt;
void cb()
{
	std::lock_guard<std::mutex> lg(mt);  //构造lg时mt.lock()
	//... 析构lg时mt.unlock()
}
```



### auto lock & unlock

C++11中引入了std::unique_lock与std::lock_guard两种数据结构。通过对lock和unlock进行一次薄的封装，实现自动lock, unlock的功能。----<font color=red>区域锁</font>！



**std::lock_guard**

```cpp
std::mutex mt;
{
    std::lock_guard<std::mutex> lk(mt); //auto call mt.lock()
} //auto call mt.unlock() when ~lk.
```



**std::unique_lock**
同于std::lock_guard都能实现自动加锁与解锁功能，但是std::unique_lock要比std::lock_guard更灵活，但是更灵活的代价是<font color=red>占用空间相对更大一点</font>且<font color=red>相对更慢一点</font>。



**lock_guard vs unique_lock**

- 都是std::mutex的RAII设计。
- unique提供更丰富的功能，lock_guard只提供了构造、析构两个函数；
  - lock, try_lock, try_lock_for, try_lock_until, unlock
  - swap, release
  - mutex, owns_lock, operator bool
    - operator bool测试关联的mutex是否是lock状态的。
- 与std::condition_variable搭配，只能是unique_lock.





## Event Handling

查看如下demo，并指出其缺点；---如何让它更能节省cpu周期、更好的性能。
```cpp
bool dataReady = false;
std::mutex mt;

void loadData()
{
	std::this_thread::sleep_for(std::chrono::milliseconds(100)); //load need 100ms.
	std::lock_guard<std::mutex> lg(mt);
	dataReady = true;
}
void processData()
{
	//process other task need 80ms.
	std::this_thread::sleep_for(std::chrono::milliseconds(80));

	//wait the data ready.
	mt.lock();
	while (!dataReady) { //缺点*：仅仅为了检查标志（状态），浪费cpu cycles.
		mt.unlock();
		std::this_thread::sleep_for(std::chrono::milliseconds(10));
		mt.lock(); //缺点*：会使loadData线程变慢（因为它也需要获得该锁去更改dataReady状态）；
	}
	mt.unlock();
	//process data, cause data was ready.
	std::this_thread::sleep_for(std::chrono::milliseconds(10));
}

std::thread th1(loadData);
std::thread th2(processData);
```
So obviously we need a better mechanism机制 to achieve this, It would have save many CPU cycles and gave better performance.



### Condition Variables
Condition Variable is a kind of Event used for signaling between two or more threads.
One or more thread can wait on it to get signaled, while an another thread can signal this.

```cpp
void wait( std::unique_lock<std::mutex>& lock );
	//释放关联的锁、阻塞当前线程、把线程放到当前信号的等待队列中（这个过程是原子操作）；等待被唤醒、虚假唤醒；
	//唤醒的过程也是个原子操作（加锁、条件变量）；
	//superiors wakeup虚假唤醒：线程被唤醒、但是等待的条件未满足；
		//由notify_one, notify_all之外的接口唤醒。
		//notify_all唤醒所有线程引起的竞争。----这种不叫虚假唤醒。

cond.notify_one()   //唤醒此信号量的一个等待线程；
cond.notify_all()   //唤醒此信号量的所有等待线程；
```

wait的变种
```cpp
template< class Predicate >
void wait( std::unique\_lock<std::mutex>& lock, Predicate pred ); //带断言的版本。
	//该版本在应对虚假唤醒时，不需要再用while来判断了；
```

demo
```cpp
#include <mutex>
#include <condition_variable>

bool dataReady = false;
std::mutex mt;
std::condition_variable cv;

{
	std::this_thread::sleep_for(std::chrono::milliseconds(100)); //load need 100ms.
	std::lock_guard<std::mutex> lg(mt);
	dataReady = true;  //lg只需要守护`状态变更`这个过程吗？`数据准备`过程呢？ ---是的，要不生成过程也被同步了。
	cv.notify_one();
}

{
	//process other task.
	std::this_thread::sleep_for(std::chrono::milliseconds(80));

	//wait the data ready.
	std::unique_lock<std::mutex> ul(mt);
#if 0
	while (!dataReady) {//重点*. 防止虚假唤醒，所以要检测条件；
		cv.wait(ul); //wait搭配unique_lock使用，而非lock_guard；
	}
#else
	cv.wait(ul, []{return dataReady == true;}); //等待着dataReady变true;
#endif
	//process data, cause data was ready.
	std::this_thread::sleep_for(std::chrono::milliseconds(10));
}
```



## Async Threads
[std::future and std::promise](#std::future_std::promise)
[std::async](#std::async)
[std::packaged\_task](#std::packaged_task)

### std::future std::promise
当我们需要线程返回值，而且还是多个值、并且在线程的不同阶段返回呢？
std::future
> 类模板；其对象内部存储了一个值、该值可在未来进行赋值；

std::promise
> 类模板；其对象承诺在未来进行赋值；
> 每个promise对象关联着一个future对象；

future.get()
对future内的值进行访问；如果该值未可用，那么调用者会一直block、直到可用；

请看如下demo.
```cpp
#include <iostream>
#include <thread>
#include <future>
 
void initiazer(std::promise<int> *promObj)
{
    std::cout << "Inside Thread" << std::endl;
	promObj->set_value(35);
}
 
{
    std::promise<int> promiseObj;
    std::future<int> futureObj = promiseObj.get_future();
    std::thread th(initiazer, &promiseObj);
    std::cout << futureObj.get() << std::endl;

    th.join();
    return 0;
}
```

### std::async
函数模板，代表一个异步任务；
接收一个callback(可以是函数指针、函数对象、lambda)、并且可能异步的执行它们；
```cpp
template <class Fn, class... Args>
future<typename result_of<Fn(Args...)>::type> async (launch policy, Fn&& fn, Args&&... args);
//policy参数
	//std::launch::async 异步执行，即立即启动线程去执行fn.
	//std::launch::deferred 不会启动新线程去执行fn，直到手动调用future.get()才开始创建线程并运行fn；
	//std::launch::async|std::launch::deferred 默认；
```

异步任务为函数指针的demo
```cpp
//https://thispointer.com/c11-multithreading-part-9-stdasync-tutorial-example/
std::string fetchDataFromDB(std::string recvdData)
{
	std::this_thread::sleep_for(seconds(5));
	return "DB_" + recvdData;
}

int main()
{
	system_clock::time_point start = system_clock::now();

	std::future<std::string> resultFromDB = std::async(std::launch::async, fetchDataFromDB, "Data");
		//1. 创建一个线程；一个promise对象；
		//2. 传送promise对象到callback，并返回关联的future对象；
		//3. 当callback结束，传入进去的promise对象被设了值，关联的future变的可用；
	std::string dbData = resultFromDB.get(); // Will block till data is available in future<std::string> object.

	auto end = system_clock::now();
	auto diff = duration_cast < std::chrono::seconds > (end - start).count();
	std::cout << "Total Time Taken = " << diff << " Seconds" << std::endl;

	return 0;
}
```

### std::packaged\_task
std::packaged\_task 类模板；
代表着一个异步任务：函数指针、lambda、函数对象；
共享状态(shared state)，存储着关联任务的返回值、异常；

task为函数的demo
```cpp
//https://thispointer.com/c11-multithreading-part-10-packaged_task-example-and-tutorial/
include <thread>
#include <future>
#include <string>
 
std::string getDataFromDB(std::string token)
{
	std::string data = "Data fetched from DB by Filter :: " + token;
	return data;
}
 
int main()
{
	std::packaged_task<std::string (std::string)> task(getDataFromDB);
 
	std::future<std::string> result = task.get_future(); //task传入thread之前，获得future.
	std::thread th(std::move(task), "Arg"); //不能copy，只能move.
 
	th.join();
 
	std::string data =  result.get(); //获取返回值；
	std::cout <<  data << std::endl;
 
	return 0;
}
```

## atomic operation
可实现无锁操作；为了高性能或底层工作,要求线程间的通信没有开销巨大的互斥锁.
https://www.cnblogs.com/dengzz/p/5686866.html
这些类都禁用了拷贝构造函数，原因是原子读和原子写是2个独立原子操作，无法保证2个独立的操作加在一起仍然保证原子性。

### std::atomic\_flag

```cpp
std::atomic_flag af = ATOMIC_FLAG_INIT;
	//std::atomic_flag可用于多线程之间的同步操作，类似于linux中的信号量。使用atomic_flag可实现mutex.
af.test_and_set()
	//会检查变量的值是否为false，如果为false则把值改为true。
	//return.
    	//true，如果atomic_flag对象被设置；
    	//false，如果atomic_flag对象未被设置。
af.clear()
	//清除atomic_flag对象
```

### std::atomic

原子数据类型
从功能上看，不会发生数据竞争，能直接用在多线程中而不必我们用户对其进行添加互斥资源锁的类型。
从实现上，可以理解为这些原子类型内部自己加了锁。

```cpp
#include <atomic>
std::atomic<Type> v;
	//std::atomic_bool
	//std::atomic_int
v.store(); //写
v.load();  //读
v.exchange();  //交换值
bool v.compare_exchange_weak(except, T newvalue);
	//Compare And Set. if(oldvalue==except) oldvalue=newvalue; else except=oldvalue;
	//weak版本的CAS，允许伪失败(spurious failures)：oldvalue与except值一样，但还是返回了false;
	//比strong有更高的性能；
bool v.compare_exchange_strong(except, T);
	//Compare And Set.
	//不允许伪失败；
```

