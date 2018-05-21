[toc]

## 锁

对于mutex和lock，要明确一点，真正起到互斥作用的mutex，而lock只是协助mutex令我们在使用时更方便。

### mutex




### lock
#### boost::lock_guard
```cpp
boost::mutex mutex;
boost::lock_guard<boost::mutex> lock(mutex);
```

#### boost::unique_lock
只能用此配合boost::condition
```cpp
boost::mutex mutex;
boost::unique_lock<boost::mutex> lock(mutex);
```



### 条件变量

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
