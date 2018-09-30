[TOC]



## ReadMe

thread pool.



什么时候用线程池？---如果：T1 + T3 远大于 T2，则可以采用线程池，以提高服务器性能。

> T1 创建线程时间；
> T2 在线程中执行任务的时间；
> T3 销毁线程时间；



components

- 线程池管理器（ThreadPool）：用于创建并管理线程池。
  - 创建线程池
  - 销毁线程池
  - 添加新任务
- 工作线程（PoolWorker）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
- 任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了
  - 任务的入口
  - 任务执行完后的收尾工作
  - 任务的执行状态等
- 任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。
  - 有了队列，就有了生产者、消费都模型；









## Case Introduction

**版本1**：one socket server <--> many socket clients;

每当一个新的client连接到server，server只有一个thread，只有服务完这个client，才能服务下一个client；



**版本2**：one socket server  <---> many socket clients.

每当一个新的client连接到server，server就会创建一个thread去为这个client进行服务。 ---但这样效率低！

> 1. pthread_create(), pthread_exit()创建、销毁线程占用资源；  ---提前create,程序结束时destory.
> 2. 当线程数量大的时候，线程间切换也是个开销； ---控制线程个数；



**版本3**：one socket server  <---> many socket clients.

每当一个新的client连接到server，server就从thread pool中拿出一个thread服务于这个client, 服务完了把thread放回thread pool。（thread从server的request queue中消费任务、server用io多路复用从多个client中拿请求并放于request queue中等待thread来消费即生产任务；）



thread pool design.

> thread_init_num  初始化个数
> thread_max_num 
> thread_busy_num 
> thread_live_num   有效的个数，busy/live=80%就要扩容、busy/live=20%就要收容
> thread_step 扩容的步长
>
> manager thread to manager the thread pools.



## MyCThreadPool

my thread pool with c implement.



### About Struct

#### threadpool_t

线程池结构

```cpp
struct threadpool_t
{
    pthread_mutex_t lock; //lock the pool.
    pthread_mutex_t thread_counter;  //lock the busy_thr_num.
    
    pthread_cond_t queue_not_full;   //when the queue full, producer wait this.
    pthread_cond_t queue_not_empyt;  //notify the consumer(thead) to pick task up.
    
    ptrhead_t *threads;   //thead pool.
    pthread_t adjust_tid; //manager thead id.
    threadpool_task_t *task_queue; //the beginner addr of task queue.
    
    int min_thr_num;
    int max_thr_num;
    int live_thr_num;
    int busy_thr_num;
    int wait_exit_thr_num;  //the numer of thread that to be destory.
    
    int queue_front;  //the index of head of task_queue.
    int queue_rear;   //the index of rear of task_queue.
    int queue_size;   //the number of tasks in task_queue.
    int queue_max_size;
    
    int shutdown;     //the thread pool usage flag. true or false.
};
```



#### threadpool_task_t

任务结构

```cpp
typedef struct
{
    void* (*function) (void*);  //the task callback.
    void* arg;
}threadpool_task_t;  //the thread task.
```



### Functions

#### threadpool_create

create thead pool.

```cpp
threadpool_t* threadpool_create(int min_thr_num, int max_thr_num, int queue_max_size)
{
    /* Create the thread pool struct. */
    threadpool_t *pool = (threadpool_t*)malloc(sizeof(threadpool_t));
    pool->min_thr_num = pool->live_thr_num = min_thr_num;
    pool->max_thr_num = max_thr_num;
    pool->busy_thr_num = pool->wait_exit_thr_num = 0;
    pool->shutdown = 0; //not close thread pool.
    
    /* init the threads pool space.. */
    pool->threads = (pthread_t*)malloc(pool->max_thr_num * sizeof(pthread_t)); //max
    memset(pool->threads, 0, pool->max_thr_num * sizeof(pthread_t)); //init mem.    
    
    /* init the task queue space. */
    pool->queue_front = pool->queue_rear = pool->queue_size = 0;
    pool->queue_max_size = queue_max_size;
    pool->task_queue = (threadpool_task_t*)malloc(pool->queue_max_size * sizeof(threadpool_task_t)); //max
    memeset(); //init mem.
    
    pthread_mutex_init(&(pool->lock), NULL);
    pthread_mutex_init(&(pool->thread_counter), NULL);
    pthread_cond_init(&(pool->queue_not_empty), NULL);
    pthread_cond_init(&(pool->queue_not_full), NULL);
    
    /* Create the threads, include worker thread, manager thread. */
    for (int i=0; i<pool->min_thr_num; i++) {
        pthread_create(&(pool->threads[i]), NULL, threadpool_thread, (void*)pool);
    }
    pthread_create(&(pool->adjust_tid), NULL, adjust_thread, (void*)pool);
    
    return pool;
}
```



#### threadpool_add

add task into thread pool.

```cpp
int threadpool_add(threadpool_t *pool, void*(*callback)(void *arg), void *arg)
{
    pthread_mutex_lock(&(pool->lock));
    /* Wait cond when the queue was full. */
    while (pool->queue_size==pool->queue_max_size && (!pool->shutdown)) {
        pthread_cond_wait(&(pool->queue_not_full), &(pool->lock));
    }
    if (pool->shutdown) { //process the pool->shutdown case.
        pthead_cond_broadcast(&(pool->queue_not_empty));
        pthread_mutex_unlock(&(pool->lock));
    }
    
    /* Insert the task into queue. */
    if (pool->task_queue[pool->queue_rear].arg != NULL) {
        pool->task_queue[pool->queue_rear].function = NULL;
        pool->task_queue[pool->queue_rear].arg = NULL;
    }
    pool->task_queue[pool->queue_rear].function = callback; 
    pool->task_queue[pool->queue_rear].arg = arg; //Noite,不能产生临时对象进行赋值
    pool->queue_rear = (pool->queue_rear +1) % pool->queue_max_size; //Notice.
    pool->queue_size++;
    
    /* Notify the consumer. */
    pthread_cond_signal(&(pool->queue_not_empty));
    pthread_umtex_unlock(&(pool->lock));
}
```



#### threadpool_destory

destory thread pool.

```cpp
int threadpool_destory(threadpool_t *pool)
{
    int i;
    if (pool == NULL) {
        return -1;
    }
    
    pool->shutdown = true;
    pthread_join(pool->adjust_tid, NULL);
    for (i=0; i<pool->live_thr_num; i++) {
        pthread_cond_broadcast(&(pool->queue_not_empty));
    }
    for (i=0; i<pool->live_thr_num; i++) {
        pthread_join(pool->threads[i], NULL);
    }
    
    threadpool_free(pool);
    return 0;
}
```

threadpool_free

```cpp
int threadpool_free(threadpool_t *pool)
{
    if (pool == NULL) {
        return -1;
    }
    if (pool->task_queue) {
        free(pool->task_queue);
    }
    if (pool->threads) {
        free(pool->threads);
        pthread_mutex_lock(&(pool->lock));
        pthread_mutex_destroy(&(pool->lock));
        pthread_mutex_lock(&(pool->thread_counter));
        pthread_mutex_destroy(&(pool->thread_counter));
        pthread_cond_destroy(&(pool->queue_not_empty));
        pthread_cond_destroy(&(pool->queue_not_full));
    }
    
    free(pool);
    pool = NULL; //not use, cause not **pool.
}
```



#### threadpool_thread

threadpool_thread 工作线程的执行体

```cpp
void threadpool_thread(void *arg)
{
    struct threadpool_t *pool = (struct threadpool_t*)arg;
    threadpool_task_t task;
    while (true) {
    	pthread_mutex_lock(&(pool->lock));
    	while ((pool->queue_size==0) && (!pool->shutdown)) {//shutdown status.
            pthread_cond_wait(&(pool->queue_not_empty), &(pool->lock));
            /* The worker threads too much, so destory some of them. */
            if (pool->wait_exit_thr_num > 0) {
                pool->live_thr_num--;
                pthread_mutex_unlock(&(pool->lock));
                pthread_exit(NULL);
            }
        }
        if (pool->shutdown) { //process one of conditions above.
            pthread_mutex_unlock(&(pool->lock));
            ptrhead_detach(ptrhead_self());
            pthread_exit(NULL);
        }
        
    	//threadpool_task_t task = pool->task_queue[pool->queue_front++];
        task.function = pool->task_queue[pool->queue_front].function;
        task.arg = pool->task_queue[pool->queue_front].arg;
        pool->queue_front = (pool->queue_front + 1) % pool->queue_max_size();
        pool->queue_size--;

        ptrhead_cond_broadcast(&(pool->queue_not_full));
        pthread_mutex_unlock(&(pool->lock));
    
    	pthread_mutex_lock(&(pool->thread_counter));
    	pool->busy_thr_num++;
    	pthread_mutex_unlock(&(pool->thread_counter));
        //task.function(task.arg);
        (*(task.function))(task.arg);
    	pthread_mutex_lock(&(pool->thread_counter));
    	pool->busy_thr_num--;
    	pthread_mutex_unlock(&(pool->thread_counter));        
    }
    ptrhead_exit(NULL);
}
```



#### adjust_thread

adjust_thread 管理线程执行体

```cpp
void *adjust_thread(void *arg)
{
    threadpool_t *pool = (threadpool_t*)arg;
    while (!pool->shutdown) {
        sleep(CHECK_TIME); //#define CHECK_TIME 10
        pthread_mutex_lock(&(pool->lock));
        int queue_size = pool->queue_size;
        int live_thr_num = pool->live_thr_num;
        pthread_mutex_unlock(&(pool->lock));
        
        pthread_mutex_lock(&(pool->thread_counter));
        int busy_thr_num = pool->busy_thr_num;
        pthread_mutex_unlock(&(pool->thread_counter));
        
        /* Create the new thread when workerthread not enough. */
        if (queue_size>=MIN_WAIT_TASK_NUM && live_thr_num<pool->max_thr_num) {
            pthread_mutex_lock(&(pool->lock));
            int add = 0;
            for (i=0; i<pool->max_thr_num && add<ADD_THREAD_STEP && pool->live_thr_num<pool->max_thr_num; i++) {
                if (pool->threads[i]==0 || !is_thread_alive(pool->threads[i])) {
                    pthread_create(&(pool->threads[i]), NULL, threadpool_thread, (void*)pool);
                    add++;
                    pool->live_thr_num++;
                }
            }
            pthread_mutex_unlock(&(pool->lock));
        }
        
        /* Del the thread when tasks not enough. */
        if ((busy_thr_num*2)<live_thr_num && live_thr_num>pool->min_thr_num) {
            ptrhead_mutex_lock(&(pool->lock));
            pool->wait_exit_thr_num = DEL_THREAD_STEP;  //del threads num step.
            pthread_mutex_unlock(&(pool->lock));
            
            for (i=0; i<DEL_THREAD_STEP; i++) {
                pthread_cond_signal(&(pool->queue_not_empty));
            }
        }
    }
    return NULL;
}
```





### Usage



the producer.

1. creat thread pool.
2. add task into thread pool. (the consumer thread process task with callback.)
3. destory thread pool.



```cpp
void* process(void *arg)
{
    printf("Thr 0x%x process task %d\n", (unsigned int)pthread_self(), (int)arg);
    sleep(1);
    printf("Thr 0x%x process task %d ok\n", (unsigned int)pthread_self(), (int)arg);
}

int usage()
{
    threadpool_t *pool = threadpool_create(3, 100, 100);
    int num[20], i;
    for (i=0; i<20; i++) {
        threadpool_add(pool, process, (void*)&num[i]);
    }
    
    sleep(10);
    threadpool_destory();
}
```

