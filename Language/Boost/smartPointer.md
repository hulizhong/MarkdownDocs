[toc]

## ReadMe
问题：智能指针为什么会出现呢？解决什么样的问题？
> 我觉得应该是c++的难点，那就是内存管理；
> 有了smartPtr之后只需要专注对象的创建、使用，释放的问题交给smartPtr；



## shared\_ptr<T>
自带引用计数器，用以表示类型T的对象引用次数；
如果count==0那么该对象已无人使用，需要释放对象空间了；

### 成员函数
```cpp
//shared_ptr
```

### 经典用例
```cpp
boost::shared_ptr<int> intPtr(new int(99));

//如何转void*，.get()之后就是裸指针了，到时随便转；
int* newPtr = intPtr.get()
(void*)newPtr;

//判空：可用==NULL,也可以用重载bool类型的
if (intPtr)
	cout << " ..has value " << endl;
```





## intrusive\_ptr<T>
比shared\_ptr更好的指针；但需要T类型自己提供引用记数机制；



## weak\_ptr<T>
弱指针，shared\_ptr的助手，配合shared\_ptr避免循环引用；
没有指针的行为；

- 重载\*
- 重载->

是shared\_ptr的观察者对象，而不进行引用（即赋值之后，引用计数不会+1）；

### 成员函数
```cpp
//可以观测资源的引用计数
use_count()

//用于检测所管理的对象是否已经释放；
expired()

//用于获取所管理的对象的强引用指针。
lock()
```



## scoped\_ptr<T>
当离开作用域就会自动释放T类型的对象；
不会传递所有权；


## shared\_array<T>
和shared\_ptr类似，用来处理数组；



## scoped\_array<T>
和scoped\_ptr类似，用来处理数组；




## 智能指针的转换
如同我们用dynamic\_cast对“裸”指针进行类层次上的上下行转换时一样；
当我们对智能指针进行类层次上的上下行转换时，则需要如下:
```cpp
boost::static_pointer_cast
boost::dynamic_pointer_cast
boost::const_pointer_cast
```

### static\_pointer\_cast
### dynamic\_pointer\_cast
### const\_pointer\_cast


