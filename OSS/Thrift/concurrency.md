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

## ThreadManager
线程管理类是个接口类，不能实例化，主要功能如下：
> 启动一堆（指定个数）线程来跑一堆任务；
> 线程函数为类中封装类Worker; 
> 任务为类中封装类Task；

包含如下模块：
[线程管理器中任务类：ThreadManager::Task](#ThreadManager::Task)
[线程管理器中各线程的线程函数：ThreadManager::Worker](#ThreadManager::Worker)
[ThreadManager::Impl](#ThreadManager::Impl)

```cpp
class ThreadManager {
protected:
  ThreadManager() {}

public:
  typedef apache::thrift::stdcxx::function<void(boost::shared_ptr<Runnable>)> ExpireCallback;

  virtual ~ThreadManager() {}

  /**
   * Starts the thread manager. Verifies all attributes have been properly
   * initialized, then allocates necessary resources to begin operation
   */
  virtual void start() = 0;

  /**
   * Stops the thread manager. Aborts all remaining unprocessed task, shuts
   * down all created worker threads, and realeases all allocated resources.
   * This method blocks for all worker threads to complete, thus it can
   * potentially block forever if a worker thread is running a task that
   * won't terminate.
   */
  virtual void stop() = 0;

  /**
   * Joins the thread manager. This is the same as stop, except that it will
   * block until all the workers have finished their work. At that point
   * the ThreadManager will transition into the STOPPED state.
   */
  virtual void join() = 0;

  enum STATE { UNINITIALIZED, STARTING, STARTED, JOINING, STOPPING, STOPPED };

  virtual STATE state() const = 0;

  virtual boost::shared_ptr<ThreadFactory> threadFactory() const = 0;

  virtual void threadFactory(boost::shared_ptr<ThreadFactory> value) = 0;

  virtual void addWorker(size_t value = 1) = 0;

  virtual void removeWorker(size_t value = 1) = 0;

  /**
   * Gets the current number of idle worker threads
   */
  virtual size_t idleWorkerCount() const = 0;

  /**
   * Gets the current number of total worker threads
   */
  virtual size_t workerCount() const = 0;

  /**
   * Gets the current number of pending tasks
   */
  virtual size_t pendingTaskCount() const = 0;

  /**
   * Gets the current number of pending and executing tasks
   */
  virtual size_t totalTaskCount() const = 0;

  /**
   * Gets the maximum pending task count.  0 indicates no maximum
   */
  virtual size_t pendingTaskCountMax() const = 0;

  /**
   * Gets the number of tasks which have been expired without being run.
   */
  virtual size_t expiredTaskCount() = 0;

  /**
   * Adds a task to be executed at some time in the future by a worker thread.
   *
   * This method will block if pendingTaskCountMax() in not zero and pendingTaskCount()
   * is greater than or equalt to pendingTaskCountMax().  If this method is called in the
   * context of a ThreadManager worker thread it will throw a
   * TooManyPendingTasksException
   *
   * @param task  The task to queue for execution
   *
   * @param timeout Time to wait in milliseconds to add a task when a pending-task-count
   * is specified. Specific cases:
   * timeout = 0  : Wait forever to queue task.
   * timeout = -1 : Return immediately if pending task count exceeds specified max
   * @param expiration when nonzero, the number of milliseconds the task is valid
   * to be run; if exceeded, the task will be dropped off the queue and not run.
   *
   * @throws TooManyPendingTasksException Pending task count exceeds max pending task count
   */
  virtual void add(boost::shared_ptr<Runnable> task,
                   int64_t timeout = 0LL,
                   int64_t expiration = 0LL) = 0;

  /**
   * Removes a pending task
   */
  virtual void remove(boost::shared_ptr<Runnable> task) = 0;

  /**
   * Remove the next pending task which would be run.
   *
   * @return the task removed.
   */
  virtual boost::shared_ptr<Runnable> removeNextPending() = 0;

  /**
   * Remove tasks from front of task queue that have expired.
   */
  virtual void removeExpiredTasks() = 0;

  /**
   * Set a callback to be called when a task is expired and not run.
   *
   * @param expireCallback a function called with the shared_ptr<Runnable> for
   * the expired task.
   */
  virtual void setExpireCallback(ExpireCallback expireCallback) = 0;

  static boost::shared_ptr<ThreadManager> newThreadManager();

  /**
   * Creates a simple thread manager the uses count number of worker threads and has
   * a pendingTaskCountMax maximum pending tasks. The default, 0, specified no limit
   * on pending tasks
   */
  static boost::shared_ptr<ThreadManager> newSimpleThreadManager(size_t count = 4,
                                                                 size_t pendingTaskCountMax = 0);

  class Task;

  class Worker;

  class Impl;
};
```

### ThreadManager::Task
```cpp
class ThreadManager::Task : public Runnable {
public:
  enum STATE { WAITING, EXECUTING, CANCELLED, COMPLETE };

  Task(shared_ptr<Runnable> runnable, int64_t expiration = 0LL)
    : runnable_(runnable),
      state_(WAITING),
      expireTime_(expiration != 0LL ? Util::currentTime() + expiration : 0LL) {}

  ~Task() {}

  void run() {
    if (state_ == EXECUTING) {
      runnable_->run();
      state_ = COMPLETE;
    }
  }

  shared_ptr<Runnable> getRunnable() { return runnable_; }

  int64_t getExpireTime() const { return expireTime_; }

private:
  shared_ptr<Runnable> runnable_;
  friend class ThreadManager::Worker;
  STATE state_;
  int64_t expireTime_;
};
```

### ThreadManager::Worker
```cpp
class ThreadManager::Worker : public Runnable {
  enum STATE { UNINITIALIZED, STARTING, STARTED, STOPPING, STOPPED };

public:
  Worker(ThreadManager::Impl* manager) : manager_(manager), state_(UNINITIALIZED), idle_(false) {}

  ~Worker() {}

private:
  bool isActive() const {
    return (manager_->workerCount_ <= manager_->workerMaxCount_)
           || (manager_->state_ == JOINING && !manager_->tasks_.empty());
  }

public:
  /**
   * Worker entry point
   *
   * As long as worker thread is running, pull tasks off the task queue and
   * execute.
   */
  void run() {
    bool active = false;
    bool notifyManager = false;

    /**
     * Increment worker semaphore and notify manager if worker count reached
     * desired max
     *
     * Note: We have to release the monitor and acquire the workerMonitor
     * since that is what the manager blocks on for worker add/remove
     */
    {
      Synchronized s(manager_->monitor_);
      active = manager_->workerCount_ < manager_->workerMaxCount_;
      if (active) {
        manager_->workerCount_++;
        notifyManager = manager_->workerCount_ == manager_->workerMaxCount_;
      }
    }

    if (notifyManager) {
      Synchronized s(manager_->workerMonitor_);
      manager_->workerMonitor_.notify();
      notifyManager = false;
    }

    while (active) {
      shared_ptr<ThreadManager::Task> task;

      /**
       * While holding manager monitor block for non-empty task queue (Also
       * check that the thread hasn't been requested to stop). Once the queue
       * is non-empty, dequeue a task, release monitor, and execute. If the
       * worker max count has been decremented such that we exceed it, mark
       * ourself inactive, decrement the worker count and notify the manager
       * (technically we're notifying the next blocked thread but eventually
       * the manager will see it.
       */
      {
        Guard g(manager_->mutex_);
        active = isActive();

		//未关闭状态下、任务队列为空，那么就睡眠等待任务到来；
        while (active && manager_->tasks_.empty()) {
          manager_->idleCount_++;
          idle_ = true;
          manager_->monitor_.wait();
          active = isActive();
          idle_ = false;
          manager_->idleCount_--;
        }

        if (active) {
          manager_->removeExpiredTasks(); //删除到期任务，会调用超时回调；

          if (!manager_->tasks_.empty()) {
            task = manager_->tasks_.front(); //取出最头部的一个任务；
            manager_->tasks_.pop();
            if (task->state_ == ThreadManager::Task::WAITING) {
              task->state_ = ThreadManager::Task::EXECUTING; //更改任务状态；
            }
          }

			//如果此时插入任务被阻塞，那么唤醒之；
          if (manager_->pendingTaskCountMax_ != 0
                  && manager_->tasks_.size() <= manager_->pendingTaskCountMax_ - 1) {
              manager_->maxMonitor_.notify();
          }
        } else {
          idle_ = true;
          manager_->workerCount_--;
          notifyManager = (manager_->workerCount_ == manager_->workerMaxCount_);
        }
      }

      if (task) {
        if (task->state_ == ThreadManager::Task::EXECUTING) {
          try {
            task->run(); //执行任务的run()
          } catch (const std::exception& e) {
            GlobalOutput.printf("[ERROR] task->run() raised an exception: %s", e.what());
          } catch (...) {
            GlobalOutput.printf("[ERROR] task->run() raised an unknown exception");
          }
        }
      }
    }

    {
      Synchronized s(manager_->workerMonitor_);
      manager_->deadWorkers_.insert(this->thread());
      if (notifyManager) {
        manager_->workerMonitor_.notify();
      }
    }

    return;
  }

private:
  ThreadManager::Impl* manager_;
  friend class ThreadManager::Impl;
  STATE state_;
  bool idle_;  //当前线程是否是空闲状态；
};
```

### ThreadManager::Impl
```cpp
class ThreadManager::Impl : public ThreadManager {

public:
  Impl()
    : workerCount_(0),
      workerMaxCount_(0),
      idleCount_(0),
      pendingTaskCountMax_(0),
      expiredCount_(0),
      state_(ThreadManager::UNINITIALIZED),
      monitor_(&mutex_),
      maxMonitor_(&mutex_) {}

  ~Impl() { stop(); }

  void start();

  void stop() { stopImpl(false); }

  void join() { stopImpl(true); }

  ThreadManager::STATE state() const { return state_; }

  shared_ptr<ThreadFactory> threadFactory() const {
    Synchronized s(monitor_);
    return threadFactory_;
  }

  void threadFactory(shared_ptr<ThreadFactory> value) {
    Synchronized s(monitor_);
    threadFactory_ = value;
  }

  void addWorker(size_t value);

  void removeWorker(size_t value);

  size_t idleWorkerCount() const { return idleCount_; }

  size_t workerCount() const {
    Synchronized s(monitor_);
    return workerCount_;
  }

  size_t pendingTaskCount() const {
    Synchronized s(monitor_);
    return tasks_.size();
  }

  size_t totalTaskCount() const {
    Synchronized s(monitor_);
    return tasks_.size() + workerCount_ - idleCount_;
  }

  size_t pendingTaskCountMax() const {
    Synchronized s(monitor_);
    return pendingTaskCountMax_;
  }

  size_t expiredTaskCount() {
    Synchronized s(monitor_);
    size_t result = expiredCount_;
    expiredCount_ = 0;
    return result;
  }

  void pendingTaskCountMax(const size_t value) {
    Synchronized s(monitor_);
    pendingTaskCountMax_ = value;
  }

  bool canSleep();

  void add(shared_ptr<Runnable> value, int64_t timeout, int64_t expiration);

  void remove(shared_ptr<Runnable> task);

  shared_ptr<Runnable> removeNextPending();

  void removeExpiredTasks();

  void setExpireCallback(ExpireCallback expireCallback);

private:
  void stopImpl(bool join);

  size_t workerCount_;      //当前正在工作的线程数；
  size_t workerMaxCount_;   //最大工作线程数
  size_t idleCount_;       //空闲的线程数；
  size_t pendingTaskCountMax_;  //pending的任务最大数；
  size_t expiredCount_;
  ExpireCallback expireCallback_;  //超时的task需要执行的cb

  ThreadManager::STATE state_;
  shared_ptr<ThreadFactory> threadFactory_;

  friend class ThreadManager::Task;
  std::queue<shared_ptr<Task> > tasks_;  //任务列表
  Mutex mutex_;
  Monitor monitor_;
  Monitor maxMonitor_;
  Monitor workerMonitor_;

  friend class ThreadManager::Worker;
  std::set<shared_ptr<Thread> > workers_;      //工作线程集
  std::set<shared_ptr<Thread> > deadWorkers_;  //？
  std::map<const Thread::id_t, shared_ptr<Thread> > idMap_;
};
```

## SimpleThreadManager 
```cpp
class SimpleThreadManager : public ThreadManager::Impl {

public:
  SimpleThreadManager(size_t workerCount = 4, size_t pendingTaskCountMax = 0)
    : workerCount_(workerCount), pendingTaskCountMax_(pendingTaskCountMax) {}

  void start() {
    ThreadManager::Impl::pendingTaskCountMax(pendingTaskCountMax_);
    ThreadManager::Impl::start();
    addWorker(workerCount_);
  }

private:
  const size_t workerCount_;
  const size_t pendingTaskCountMax_;
  Monitor monitor_;
};
```

## TimerManager
定时器管理类，主要功能如下：
> 启动一个线程来跑一堆的超时任务；
> 线程函数为Dispatcher; 
> 超时任务为类中封装类Task；

包含如下模块：
[定时器线程函数：TimerManager::Dispatcher](#TimerManager::Dispatcher)

```cpp
class TimerManager {
public:
  TimerManager();

  virtual ~TimerManager();

  virtual boost::shared_ptr<const ThreadFactory> threadFactory() const;

  virtual void threadFactory(boost::shared_ptr<const ThreadFactory> value);

  /**
   * Starts the timer manager service
   *
   * @throws IllegalArgumentException Missing thread factory attribute
   */
  virtual void start();

  /**
   * Stops the timer manager service
   */
  virtual void stop();

  virtual size_t taskCount() const;

  /**
   * Adds a task to be executed at some time in the future by a worker thread.
   *
   * @param task The task to execute
   * @param timeout Time in milliseconds to delay before executing task
   */
  virtual void add(boost::shared_ptr<Runnable> task, int64_t timeout);

  /**
   * Adds a task to be executed at some time in the future by a worker thread.
   *
   * @param task The task to execute
   * @param timeout Absolute time in the future to execute task.
   */
  virtual void add(boost::shared_ptr<Runnable> task, const struct THRIFT_TIMESPEC& timeout);

  /**
   * Adds a task to be executed at some time in the future by a worker thread.
   *
   * @param task The task to execute
   * @param timeout Absolute time in the future to execute task.
   */
  virtual void add(boost::shared_ptr<Runnable> task, const struct timeval& timeout);

  /**
   * Removes a pending task
   *
   * @throws NoSuchTaskException Specified task doesn't exist. It was either
   *                             processed already or this call was made for a
   *                             task that was never added to this timer
   *
   * @throws UncancellableTaskException Specified task is already being
   *                                    executed or has completed execution.
   */
  virtual void remove(boost::shared_ptr<Runnable> task);

  enum STATE { UNINITIALIZED, STARTING, STARTED, STOPPING, STOPPED };

  virtual STATE state() const;

private:
  boost::shared_ptr<const ThreadFactory> threadFactory_;
  class Task; //定时任务类
  friend class Task;
  std::multimap<int64_t, boost::shared_ptr<Task> > taskMap_;  //定时任务集；key为到时时间，从小到大排序；
  size_t taskCount_;  //定时任务个数
  Monitor monitor_;
  STATE state_;
  class Dispatcher;  //定时任务线程函数，即定时器分发器；
  friend class Dispatcher;
  boost::shared_ptr<Dispatcher> dispatcher_;   //定时任务线程函数；
  boost::shared_ptr<Thread> dispatcherThread_; //定时任务线程；
  typedef std::multimap<int64_t, boost::shared_ptr<TimerManager::Task> >::iterator task_iterator;
  typedef std::pair<task_iterator, task_iterator> task_range;
};
```

### TimerManager::Dispatcher
定时器分发器；
```cpp
class TimerManager::Dispatcher : public Runnable {

public:
  Dispatcher(TimerManager* manager) : manager_(manager) {}

  ~Dispatcher() {}

  /**
   * Dispatcher entry point
   *
   * As long as dispatcher thread is running, pull tasks off the task taskMap_
   * and execute.
   */
  void run() {
    {
      Synchronized s(manager_->monitor_);
      if (manager_->state_ == TimerManager::STARTING) {
        manager_->state_ = TimerManager::STARTED;
        manager_->monitor_.notifyAll();
      }
    }

    do {
      std::set<shared_ptr<TimerManager::Task> > expiredTasks;
      {
        Synchronized s(manager_->monitor_);
        task_iterator expiredTaskEnd;
        int64_t now = Util::currentTime();

		//轮询等待超时任务的时机到来；
		//taskMap_={5点, 7点, 9点}, now=4点，那么第1个元素5大于Now，就说明现在没有一个任务是超时的；
        while (manager_->state_ == TimerManager::STARTED && 
              (expiredTaskEnd = manager_->taskMap_.upper_bound(now)) == manager_->taskMap_.begin()) {
          int64_t timeout = 0LL;
          if (!manager_->taskMap_.empty()) {
            timeout = manager_->taskMap_.begin()->first - now;
          }
          assert((timeout != 0 && manager_->taskCount_ > 0)
                 || (timeout == 0 && manager_->taskCount_ == 0));
          try {
            manager_->monitor_.wait(timeout);
          } catch (TimedOutException&) {
          }
          now = Util::currentTime();
        }

		//找出超时的任务、删除，并置TimeManager状态；
        if (manager_->state_ == TimerManager::STARTED) {
          for (task_iterator ix = manager_->taskMap_.begin(); ix != expiredTaskEnd; ix++) {
            shared_ptr<TimerManager::Task> task = ix->second;
            expiredTasks.insert(task);
            if (task->state_ == TimerManager::Task::WAITING) {
              task->state_ = TimerManager::Task::EXECUTING;
            }
            manager_->taskCount_--;
          }
          manager_->taskMap_.erase(manager_->taskMap_.begin(), expiredTaskEnd);
        }
      }

		//执行超时任务；
      for (std::set<shared_ptr<Task> >::iterator ix = expiredTasks.begin();
           ix != expiredTasks.end();
           ++ix) {
        (*ix)->run();
      }

    } while (manager_->state_ == TimerManager::STARTED);

    {
      Synchronized s(manager_->monitor_);
      if (manager_->state_ == TimerManager::STOPPING) {
        manager_->state_ = TimerManager::STOPPED;
        manager_->monitor_.notify();
      }
    }
    return;
  }

private:
  TimerManager* manager_;
  friend class TimerManager;
};
```
