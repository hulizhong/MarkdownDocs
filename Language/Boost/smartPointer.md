[toc]

## ReadMe
问题：智能指针为什么会出现呢？解决什么样的问题？
> 我觉得应该是c++的难点，那就是内存管理；
> 有了smartPtr之后只需要专注对象的创建、使用，释放的问题交给smartPtr；

boost 1.58的智能指针源码目录结构如下：
```bash
▾ smart_ptr/                   
  ▸ detail/                    
    allocate_shared_array.hpp  
    bad_weak_ptr.hpp           
    enable_shared_from_raw.hpp 
    enable_shared_from_this.hpp
    intrusive_ptr.hpp          
    intrusive_ref_counter.hpp  
    make_shared.hpp            
    make_shared_array.hpp      
    make_shared_object.hpp     
    make_unique.hpp            
    make_unique_array.hpp      
    make_unique_object.hpp     
    owner_less.hpp             
    scoped_array.hpp           
    scoped_ptr.hpp             
    shared_array.hpp           
    shared_ptr.hpp             
    weak_ptr.hpp               
```


## shared\_ptr
自带引用计数器，用以表示类型T的对象引用次数；
如果count==0那么该对象已无人使用，需要释放对象空间了；

### 成员函数
构造函数
```cpp
class Person {
public:
	Person(int v) {
		mem = malloc(v);
	}
};

boost::shared_ptr<Person> p2(new Person(2));  
boost::shared_ptr<Person> p2 = boost::shared_ptr<Person>(new Person(2)); //这样行吗？用=号了。。。 rwhy
	//这句话不可以分成两部的，看如下解释：
		// Person *ptr = New Person(2); p2 = ptr; 因为一个是类，一个是指针所以是没法赋值的！

#include <boost/make_shared.hpp>
boost::shared_ptr<Person> p1 = boost::make_shared<Person>(1);  
	//make_shared方法

boost::shared_ptr<Person> p3 = boost::shared_ptr<Person>(new Person(3), boost::bind(&Person::del, this, _1));
	//指定删除器；
	//同于std::shared_ptr的默认删除器不支持数组对象；

p2.reset();
	//语义等价于：referCount--; if(referCount == 0) delete(p1.get());
p2.reset(new Person(5));
	//新对一个对象；执行reset语义; 给p2赋上一个新的对象；
	//这个有风险，如果reset count!=0呢，原有对象有没有释放呢？？？ rwhy
```

get()获得裸指针
```cpp
boost::shared_ptr<int> intPtr(new int(99));

//如何转void*，.get()之后就是裸指针了，到时随便转；
int* newPtr = intPtr.get()
(void*)newPtr;

//判空：可用==NULL,也可以用重载bool类型的
if (intPtr)
	cout << " ..has value " << endl;
```


## weak\_ptr
弱指针，shared\_ptr的助手，配合shared\_ptr避免循环引用；
没有指针的行为；
> 重载\*
> 重载->

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



## intrusive\_ptr
比shared\_ptr更好的指针；但需要T类型自己提供引用记数机制；



## scoped\_ptr
当离开作用域就会自动释放T类型的对象；
不会传递所有权；


## shared\_array
和shared\_ptr类似，用来处理数组；



## scoped\_array
和scoped\_ptr类似，用来处理数组；


## 智能指针小结
- scoped\_ptr
	- 不共享所有权、不转移所有权、不管理数组对象；
	- 可以置换所有权；

- shared\_ptr
	- 共享所有权； 
	- 解决不了循环引用问题，所以有了弱引用；

- weak_ptr
	- 弱引用
	- boost::weak_ptr必须从一个boost::share\_ptr或另一个boost::weak_ptr转换而来；
		- 这也说明，进行该对象的内存管理的是那个强引用的boost::share_ptr。
		- boost::weak_ptr只是提供了对管理对象的一个访问手段。

什么是强引用、弱引用？
> 主要看是否参与对象内存的管理；


- 强引用
	- 会改变引用计数；
	- 参与对象内存的管理；
		- 只要（强）引用存在，对象就不能被销毁；
- 弱引用
	- 不会改变引用计数；
	- 不参与对象内存的管理；
	- 在功能上类似于普通指针，但是弱引用能检测到所管理的对象是否已经被释放，从而避免访问非法内存。
		- 当被引用的对象消失时，弱引用会自动设置为nil；



## 智能指针的转换
如同我们用dynamic\_cast对“裸”指针进行类层次上的上下行转换时一样；
当我们对智能指针进行类层次上的上下行转换时，则需要如下:
```cpp
boost::static_pointer_cast
boost::dynamic_pointer_cast
boost::const_pointer_cast
```

reinterpret\_cast重解释转换
这个转换是最“不安全”的，两个没有任何关系的类指针之间转换都可以用这个转换实现，如下：
```cpp
// reinterpret_cast<new_type>(expression)

boost::shared_ptr<PthreadThread> *arg = new boost::shared_ptr<PthreadThread>;
static void* threadMain(void* arg) {
	shared_ptr<PthreadThread> thread = *(shared_ptr<PthreadThread>*)arg;
	delete reinterpret_cast<shared_ptr<PthreadThread>*>(arg);
}
```

具体如下
参见[static\_pointer\_cast](#static_pointer_cast)
参见[dynamic\_pointer\_cast](#dynamic_pointer_cast)
参见[const\_pointer\_cast](#const_pointer_cast)
[](#)

### static\_pointer\_cast
### dynamic\_pointer\_cast
### const\_pointer\_cast


