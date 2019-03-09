[TOC]



## ReadMe

c11智能指针。



## refer_count

引用计数实际上就是为了解决这种浅拷贝问题而诞生的，每当对资源（堆内存 ）引用一次就是对计数器+1，每当删除一次，就对计数器减一，直到当资源的引用计数为0时，就证明没有对象对它进行引用了，此时调用析构函数对资源进行释放。





## smart_ptr

内存管理方面，在c98的`std::auto_ptr`基础上，移植了boost中smart pointer的部分实现，并突破了`boost::make_shared()`中构造参数的限制问题。

- 原理
  - 说法一：接受一个申请好的内存地址，构造一个保存在栈上的智能指针对象，当程序退出栈的作用域范围后，由于栈上的变量自动被销毁，智能指针内部保存的内存也就被释放掉了（除非将智能指针保存起来）；
  - 说法二：利用RAII（Resource Acquisition Is Initialization资源获取即初始化）对普通的指针进行封装，使得smart pointer实质上是一个对象，但行为表现却是一个指针；
- 作用
  - 防止忘记调用delete释放内存；
  - 多次释放同一个指针引起的crash；
  - 把值语义转成引用语义；



### shared_ptr

```cpp
#include <memory>

//多个shared_ptr可以指向相同的对象。
//shared_ptr内部的引用计数是线程安全的，但是对象的读取需要加锁。
//什么时候用-> vs .
	//->用于smartptr指向对象的成员。
	//.用于smartptr的成员。
//std::shared_ptr<int> std::shared_ptr<int> pa = new int(1); //error.类、指针。
//不能用一个原始指针初始化多个shared_ptr -->二次释放同一内存。
//不能循环引用  --->内存无法释放，用weak_ptr.
std::shared_ptr<int> std::shared_ptr<int> pa(new int(1)); //用裸指针构造；
std::shared_ptr<int> p = std::make_shared<int>(1);  //make_shared+构造函数的参数；
int *pp = p.get(); //获得裸指针。

```



**Q, shared_ptr VS make_shared ? 尽量使用make_shared<>, make_unique<>.**
shared_ptr是申请两次内存（数据、控制块），make_shared只申请了一次内存（数据 + 控制块）。
表现在当`std::shared_ptr`作为函数参数时的`异常安全`，如下：

```cpp
void f(std::shared_ptr<Lhs> &lhs, std::shared_ptr<Rhs> &rhs) {...}
f(std::shared_ptr<Lhs>(new Lhs()), std::shared_ptr<Rhs>(new Rhs())); //error.
	//因为C++允许参数在计算的时候打乱顺序，因此一个可能的顺序如下:
		//1.new Lhs(), 2.new Rhs(), 3.std::shared_ptr, 4.std::shared_ptr
	//当2失败时，1申请的内存没法释放。
f(std::make_shared<Lhs>(), std::make_shared<Rhs>());  //ok.
```





### unique_ptr

```cpp
//unique_ptr唯一拥有其所指对象。（禁止拷贝语义、只有移动语义）
std::unique_ptr<int> uptr(new int(10));
std::unique_ptr<int> uptr2 = std::move(uptr); //转移所有权。
	//如果函数参数为std::unique_ptr，那么只能用std::move进行转移。
uptr2.release(); //释放所有权。

std::make_unique<int>(3); //c++14支持
```







### weak_ptr

```cpp
//协助shared_ptr进行旁观者一样观察资源使用情况，不具有指针行为（*, ->）。
//由shared_ptr, weak_ptr对象构造，来获得资源的观测权。
std::shared_ptr<int> sh_ptr = std::make_shared<int>(10);
sh_ptr.use_cout(); //当前对象的引用数。
std::weak_ptr<int> wp(sh_ptr);
wp.use_count(); //当前对象的引用数。
if (!wp.expired()) {
    std::shared_ptr<int> sh_ptr2 = wp.lock();  //获得一个可用的shared_ptr对象
    wp.use_cout();
}
else {
    std::shared_ptr<int> sh_ptr2 = wp.lock(); //获得一个存储空的shared_ptr对象。
}
```



## smart_ptr cast

c11定义了如下三个smart_ptr间相互转换的函数。
<font color=gree>转换成功之后，原对象的引用计数会**+1**</font>。

```cpp
template< class T, class U > 
std::shared_ptr<T> static_pointer_cast( const std::shared_ptr<U>& r ) noexcept;

template< class T, class U > 
std::shared_ptr<T> dynamic_pointer_cast( const std::shared_ptr<U>& r ) noexcept;

template< class T, class U > 
std::shared_ptr<T> const_pointer_cast( const std::shared_ptr<U>& r ) noexcept;
```





## 循环引用

### 版本一

裸指针 + 从类内部构造对象；

```cpp
class B;
 
class A { 
public:
#if 1
    A();
    ~A();
#else
    A() {
        cout << "+A" << endl;
        pb = new B(); //invalid use of incomplete type ‘class B’
            //move this function after B's declaration.
    }
    ~A() {
        cout << "~A" << endl;
        delete pb;
    }
#endif
private:
    B *pb;
};
 
class B { 
public:
    B() {
        cout << "+B" << endl;
        pa = new A();
    }   
    ~B() {
        cout << "~B" << endl;
        delete pa; 
    }   
private:
    A *pa;
};
 
#if 1
A::A() {
    cout << "+A" << endl;
    pb = new B();
}
A::~A() {
    cout << "~A" << endl;
    delete pb; 
}
#endif
 
int main()
{
    A a;
    //B b;
    return 0;
}
//这段代码会导致死循环，那A、B、A、B...
	//循环引用这个还是只能从外部建立对象再传进来。
```

### 版本二

裸指针 + 从类外部构造对象。

```cpp
class B;
 
class A { 
public:
    A() {                                         
        cout << "+A" << endl;
    }   
    ~A() {
        cout << "~A" << endl;
        delete pb; 
    }   
    bool setB(B *b) {pb = b;} 
private:
    B *pb;
};
 
class B { 
public:
    B() {
        cout << "+B" << endl;
    }   
    ~B() {
        cout << "~B" << endl;
        delete pa; 
    }   
    bool setA(A *a) {pa = a;} 
private:
    A *pa;
};
 
 
int main()
{
    A *pa = new A;
    B *pb = new B;
    pa->setB(pb);
    pb->setA(pa);
    delete pa; //only has '~A', why not has '~B'.
    return 0;
}
```

### 版本三

换上shared_ptr

```cpp
class B;                               
                                       
class A {                              
public:                                
    A() {                              
        cout << "+A" << endl;          
    }                                  
    ~A() {                             
        cout << "~A" << endl;          
    }                                  
    bool setB(std::shared_ptr<B> b) {pb = b;}
private:                               
    std::shared_ptr<B> pb;             
};                                     
                                       
class B {                              
public:                                
    B() {                              
        cout << "+B" << endl;          
    }                                  
    ~B() {                             
        cout << "~B" << endl;          
    }                                  
    bool setA(std::shared_ptr<A> a) {pa = a;}
private:                               
    std::shared_ptr<A> pa;             
};                                     
                                       
                                       
int main()                             
{                                      
    std::weak_ptr<A> wpa;              
    std::weak_ptr<B> wpb;              
    {                                  
        std::shared_ptr<A> pa(new A);  
        std::shared_ptr<B> pb(new B);  
        pa->setB(pb);                  
        pb->setA(pa);                  
        wpa = pa;                      
        wpb = pb;                      
        cout << pa.use_count() << endl; //2
        cout << pb.use_count() << endl; //2
    }                                        
    cout << wpa.use_count() << endl; //1
    cout << wpb.use_count() << endl; //1
    return 0;                          
}
//一个析构都没有调用。
```



### 版本四

weak_ptr配合shared_ptr.

```cpp
class B;    
            
class A {   
public:     
    A() {   
        cout << "+A" << endl;          
    }       
    ~A() {  
        cout << "~A" << endl;          
    }       
    bool setB(std::shared_ptr<B> b) {pb = b;}
private:    
    //std::shared_ptr<B> pb;           
    std::weak_ptr<B> pb;               
};          
            
class B {   
public:     
    B() {   
        cout << "+B" << endl;          
    }       
    ~B() {  
        cout << "~B" << endl;          
    }       
    bool setA(std::shared_ptr<A> a) {pa = a;}
private:    
    std::shared_ptr<A> pa;             
};          
            
            
int main()  
{           
    std::weak_ptr<A> wpa;              
    std::weak_ptr<B> wpb;              
    {       
        std::shared_ptr<A> pa(new A);  
        std::shared_ptr<B> pb(new B);  
        pa->setB(pb);                  
        pb->setA(pa);                  
        wpa = pa;                      
        wpb = pb;                      
        cout << pa.use_count() << endl; //2
        cout << pb.use_count() << endl; //1
    }       
    cout << wpa.use_count() << endl; //0
    cout << wpb.use_count() << endl; //0
    return 0;
}
```

