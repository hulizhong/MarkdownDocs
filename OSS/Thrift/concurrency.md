[toc]

## ReadMe
讲述thrift并发部分，也就是线程封装部分；
thrift支持以下三种线程；
> std::thread  
> posix thread  
> boost thread  

## Runnable
很简单，就提供了
<font color=red>一个线程的弱引用；</font>
一个run()函数，估计是线程函数吧；
```cpp
class Runnable {
public:
  virtual ~Runnable(){};
  virtual void run() = 0;

  virtual boost::shared_ptr<Thread> thread() { return thread_.lock(); }
  virtual void thread(boost::shared_ptr<Thread> value) { thread_ = value; }

private:
  boost::weak_ptr<Thread> thread_; //注意这是线程的弱引用；
};
```

## ThreadFactory 
创建一个特定平台的线程、并绑定了一个Runnable的对象；
```cpp
class ThreadFactory {
public:
  virtual ~ThreadFactory() {}
  virtual boost::shared_ptr<Thread> newThread(boost::shared_ptr<Runnable> runnable) const = 0;

  static const Thread::id_t unknown_thread_id;

  virtual Thread::id_t getCurrentThreadId() const = 0;
};
```

## PosixThreadFactory 
factory创建出来的所有线程都是强引用对象，如boost::shared\_ptr/weak\_ptr
factory要保证当线程对象引用计数为0时，释放线程本身、及绑定在线程上的Runnable task.
```cpp
class PosixThreadFactory : public ThreadFactory {

public:
	//posix thread调度策略
  enum POLICY { OTHER, FIFO, ROUND_ROBIN };

	//posix thread调度的相对优先级，只是我们抽象出来的；
	//绝对的真正的是由调度策略、OS决定的，还得把这些转换成pthread支持的才行；
  enum PRIORITY {
    LOWEST = 0,
    LOWER = 1,
    LOW = 2,
    NORMAL = 3,
    HIGH = 4,
    HIGHER = 5,
    HIGHEST = 6,
    INCREMENT = 7,
    DECREMENT = 8
  };

  PosixThreadFactory(POLICY policy = ROUND_ROBIN,
                     PRIORITY priority = NORMAL,
                     int stackSize = 1,
                     bool detached = true);

  // From ThreadFactory;
  boost::shared_ptr<Thread> newThread(boost::shared_ptr<Runnable> runnable) const;
  // From ThreadFactory;
  Thread::id_t getCurrentThreadId() const;

	//线程栈大小
  virtual int getStackSize() const;
  virtual void setStackSize(int value);

	//相对优先级；
  virtual PRIORITY getPriority() const;
  virtual void setPriority(PRIORITY priority);

	//分离模式
  virtual void setDetached(bool detached);
  virtual bool isDetached() const;

private:
  class Impl;
  boost::shared_ptr<Impl> impl_;
}
```
### PosixThreadFactory::Impl
具体的PosixThreadFactory实现，是一个内部类；
```cpp
class PosixThreadFactory::Impl {
private:
  POLICY policy_;
  PRIORITY priority_;
  int stackSize_;
  bool detached_;

	//转换POLICY到pthread api中策略SCHED_OTHER,SCHED_FIFO,SCHED_RR.
  static int toPthreadPolicy(POLICY policy) {..}

	//将policy下相对priority转换成pthread的绝对priority.
  static int toPthreadPriority(POLICY policy, PRIORITY priority);

public:
  Impl(POLICY policy, PRIORITY priority, int stackSize, bool detached)
    : policy_(policy), priority_(priority), stackSize_(stackSize), detached_(detached) {}

	//创建一个pthread来运行runnable obj.
  shared_ptr<Thread> newThread(shared_ptr<Runnable> runnable) const {
    shared_ptr<PthreadThread> result
        = shared_ptr<PthreadThread>(new PthreadThread(toPthreadPolicy(policy_),
                                                      toPthreadPriority(policy_, priority_),
                                                      stackSize_,
                                                      detached_,
                                                      runnable));
    result->weakRef(result); //把线程自己的指针赋值给自己（线程类里有个weak_ptr的属性）
    runnable->thread(result);
    return result;
  }

  int getStackSize() const { return stackSize_; }
  void setStackSize(int value) { stackSize_ = value; }

  PRIORITY getPriority() const { return priority_; }
  void setPriority(PRIORITY value) { priority_ = value; }

  bool isDetached() const { return detached_; }
  void setDetached(bool value) { detached_ = value; }

  Thread::id_t getCurrentThreadId() const {
#ifndef _WIN32
    return (Thread::id_t)pthread_self();
#else
    return (Thread::id_t)pthread_self().p;
#endif // _WIN32
  }
};
```

#### PosixThreadFactory::Impl::toPthreadPriority
转换线程相对优先级到pthread基于策略的绝对优先级；
把pthread所支持的policy下所有优先级，按照PRIORITY中lowest到highest的个数进行几等份；
```cpp
  static int toPthreadPriority(POLICY policy, PRIORITY priority) {
    int pthread_policy = toPthreadPolicy(policy);
    int min_priority = 0;
    int max_priority = 0;
#ifdef HAVE_SCHED_GET_PRIORITY_MIN
    min_priority = sched_get_priority_min(pthread_policy);
#endif
#ifdef HAVE_SCHED_GET_PRIORITY_MAX
    max_priority = sched_get_priority_max(pthread_policy);
#endif
    int quanta = (HIGHEST - LOWEST) + 1;
    //lowest到highest的步阶；
    float stepsperquanta = (float)(max_priority - min_priority) / quanta;

    if (priority <= HIGHEST) {
      return (int)(min_priority + stepsperquanta * priority);
    } else {
      // should never get here for priority increments.
      assert(false);
      return (int)(min_priority + stepsperquanta * NORMAL);
    }
  }
```

## Thread
支持三种类型线程：boost thread, std::thread, pthread  
该类没有什么，只有个Runnable属性；
```cpp
class Thread {
public:
#if USE_BOOST_THREAD
  typedef boost::thread::id id_t;

  static inline bool is_current(id_t t) { return t == boost::this_thread::get_id(); }
  static inline id_t get_current() { return boost::this_thread::get_id(); }
#elif USE_STD_THREAD
  typedef std::thread::id id_t;

  static inline bool is_current(id_t t) { return t == std::this_thread::get_id(); }
  static inline id_t get_current() { return std::this_thread::get_id(); }
#else
  typedef pthread_t id_t;

  static inline bool is_current(id_t t) { return pthread_equal(pthread_self(), t); }
  static inline id_t get_current() { return pthread_self(); }
#endif

  virtual ~Thread(){};

	//根据平台配置线程、启动线程；（不会阻塞的）
	//线程函数，需要运行绑定在此线程Runnable的run()
  virtual void start() = 0;

	//join这个线程，调用者会被阻塞直到该线程结束；
  virtual void join() = 0;

  virtual id_t getId() = 0;

	//返回绑定在此线程上的runnable对象；
  virtual boost::shared_ptr<Runnable> runnable() const { return _runnable; }

protected:
  virtual void runnable(boost::shared_ptr<Runnable> value) { _runnable = value; }

private:
  boost::shared_ptr<Runnable> _runnable; //子类看不到它，只能靠上述两个api进行访问；
};
```


## PthreadThread 
它是posix thread即ptrhead的封装；
```cpp
class PthreadThread : public Thread {
public:
  enum STATE { uninitialized, starting, started, stopping, stopped }; //线程状态；
  static const int MB = 1024 * 1024;
  static void* threadMain(void* arg); //线程函数

private:
  pthread_t pthread_;
  STATE state_;
  int policy_;
  int priority_;
  int stackSize_;
  weak_ptr<PthreadThread> self_; //指向自己的一个弱引用；
  bool detached_;

public:
  PthreadThread(int policy,
                int priority,
                int stackSize,
                bool detached,
                shared_ptr<Runnable> runnable)
    :

#ifndef _WIN32
      pthread_(0),
#endif // _WIN32

      state_(uninitialized), //未初始化状态；
      policy_(policy),
      priority_(priority),
      stackSize_(stackSize),
      detached_(detached) {

    this->Thread::runnable(runnable);  //绑定此线程的任务；
  }

	//如果不是detached状态，那么调用join hold住；
  ~PthreadThread() {
    if (!detached_) {
      try {
        join();
      } catch (...) {
        // We're really hosed.
      }
    }
  }

  void start(); //启动线程
  void join();  //连接线程

  Thread::id_t getId() {

#ifndef _WIN32
    return (Thread::id_t)pthread_;
#else
    return (Thread::id_t)pthread_.p;
#endif // _WIN32
  }

  shared_ptr<Runnable> runnable() const { return Thread::runnable(); }
  void runnable(shared_ptr<Runnable> value) { Thread::runnable(value); }

  void weakRef(shared_ptr<PthreadThread> self) {
    assert(self.get() == this);
    self_ = weak_ptr<PthreadThread>(self);
  }
};
```
重要的函数如下：
[PthreadThread::start](#PthreadThread::start)
[PthreadThread::threadMain](#PthreadThread::threadMain)
[PthreadThread::join](#PthreadThread::join)


### PthreadThread::start
设置线程属性、优先级并进行启动；
不会阻塞的；
```cpp
void PthreadThread::start() {
  if (state_ != uninitialized) {
    return;
  }

  //设置线程的分离、栈大小属性；
  pthread_attr_t thread_attr;
  if (pthread_attr_init(&thread_attr) != 0) {
    throw SystemResourceException("pthread_attr_init failed");
  }

  if (pthread_attr_setdetachstate(&thread_attr,
                                  detached_ ? PTHREAD_CREATE_DETACHED : PTHREAD_CREATE_JOINABLE)
      != 0) {
    throw SystemResourceException("pthread_attr_setdetachstate failed");
  }

  // Set thread stack size
  if (pthread_attr_setstacksize(&thread_attr, MB * stackSize_) != 0) {
    throw SystemResourceException("pthread_attr_setstacksize failed");
  }

	//设置的调度策略；
#ifdef _WIN32
	//win32只支持other策略（分时调度）；
  policy_ = PosixThreadFactory::OTHER;
#endif
  if (pthread_attr_setschedpolicy(&thread_attr, policy_) != 0) {
    throw SystemResourceException("pthread_attr_setschedpolicy failed");
  }

	//设置的调度优先级；
  struct sched_param sched_param;
  sched_param.sched_priority = priority_;
  // Set thread priority
  if (pthread_attr_setschedparam(&thread_attr, &sched_param) != 0) {
    throw SystemResourceException("pthread_attr_setschedparam failed");
  }

  // Create reference
  shared_ptr<PthreadThread>* selfRef = new shared_ptr<PthreadThread>(); //只是一个指针，没有涉及到对象的创建；
  *selfRef = self_.lock(); //指针内容为，PthreadThread对象本身；

  state_ = starting;

  if (pthread_create(&pthread_, &thread_attr, threadMain, (void*)selfRef) != 0) {
    throw SystemResourceException("pthread_create failed");
  }
}
```

### PthreadThread::threadMain
线程函数，运行runnable.run()
```cpp
void* PthreadThread::threadMain(void* arg) {
  shared_ptr<PthreadThread> thread = *(shared_ptr<PthreadThread>*)arg;
  delete reinterpret_cast<shared_ptr<PthreadThread>*>(arg);

  if (thread == NULL) { //线程不存在
	return (void*)0;
  }
  if (thread->state_ != starting) { //线程状态不对
	return (void*)0;
  }

#if GOOGLE_PERFTOOLS_REGISTER_THREAD
  ProfilerRegisterThread();
#endif

  thread->state_ = started;
  thread->runnable()->run(); //运行线程上的任务.run()
  if (thread->state_ != stopping && thread->state_ != stopped) {
	thread->state_ = stopping;
  }

  return (void*)0;
}
```

### PthreadThread::join
连接此线程
```cpp
void PthreadThread::join() {
  //只要线程跑起来、并且不是分离状态的；
  if (!detached_ && state_ != uninitialized) {
    void* ignore;
    /* XXX
       If join fails it is most likely due to the fact
       that the last reference was the thread itself and cannot
       join.  This results in leaked threads and will eventually
       cause the process to run out of thread resources.
       We're beyond the point of throwing an exception.  Not clear how
       best to handle this. */
  	//怎么去处理，连接线程失败呢？我们没有很好的办法；
  	//线程连接失败，很大可能是因为最后一次引用是线程本身；如果join()失败，那么泄漏的线程最终会用完进程资源；
    int res = pthread_join(pthread_, &ignore);
    detached_ = (res == 0);
    if (res != 0) {
      GlobalOutput.printf("PthreadThread::join(): fail with code %d", res);
    }
  } else {
    GlobalOutput.printf("PthreadThread::join(): detached thread");
  }
}
```


