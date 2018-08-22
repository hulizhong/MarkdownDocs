[TOC]



## ReadMe

cpp的重点难道；



## 临时对象 

问题：注意临时对象的销毁时间？  

> 由于表达式的计算产生的临时对象会在表达式计算完成（遇到；号）之后被销毁。  

问题：临时对象的产生时机？  

> 类型转换、值传递（参数、结果）、对象定义

如下代码断会有什么问题？

```cpp
const char* p = string("hello temprary string").c_str();  
cout << p; 
```



## RAII

Resource Acquisition is Initialization直译过来是“资源获取即初始化”，也就是说在构造函数中申请分配资源，在析构函数中释放资源。因为C++的语言机制保证了，当一个对象创建的时候，自动调用构造函数，当对象超出作用域的时候会自动调用析构函数。所以，在RAII的指导下，我们应该使用类来管理资源，将资源和对象的生命周期绑定。

> RAII的核心思想是将资源、状态与对象的生命周期绑定，通过C++的语言机制，实现资源和状态的安全管理。 



典型应用：智能指针
智能指针是RAII典型的用法之一。
可以实现自动的内存管理，再也不需要担心忘记delete造成的内存泄漏。 



典型应用：安全的状态管理

```cpp
std::mutex mutex_;
void function()
{
    mutex_.lock();
    ......
    ......//这些中途路径，程序异常退出、或者return了呢？（这些状态怎么管理？？？）
    mutex_.unlock();
}

//------然后就有了：lock_guard, unique_lock;
std::mutex mutex_;
void function()
{
    std::lock_guard<std::mutex> lock(mutex_);
    ......
    ......
}
```







## 其它

首先问个问题：如何不创建对象，而调用类中的函数？？

> AA *a = NULL;  a->fun();
>
> reinterpret_cast<AA*>(NULL)->fun();



