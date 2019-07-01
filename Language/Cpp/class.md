[TOC]

## 友元
允许一个类将对其非公有成员的访问权（protect, private）授予指定的函数或者类。
友元声明可以出现在类中的任何地方，而<font color=gree>不受（public, protect, private）限制</font>。

### 各种友元
- 友元函数

  ```cpp
  class A
  {
  public:
  	friend void set_show(int x, A &a);  //该函数是友元函数的声明
  private:
  	int data;
  };
  
  /*
  友元函数：!!必须要有个类对象的引用!!。
  */
  void set_show(int x, A &a)
  {
  	a.data = x;
  	cout << a.data << endl;
  }
  ```

- 友元类（友元类C的所有函数都可以访问A中的protect, private）

  ```cpp
  class A
  {
  public:
  	friend class C;  //这是友元类的声明
  private:
  	int data;
  };
  
  class C  //友元类定义，为了访问类A中的成员
  {
  public:
  	//一样的，必须要有个被访问类的对象引用。
  	void set_show(int x, A &a) { a.data = x; cout<<a.data<<endl;}
  };
  ```

- 友元类成员函数

  ```cpp
  //当用到友元成员函数时，需注意友元声明与友元定义之间的互相依赖。这是类A的声明
  class A;
  
  class B
  {
  public:
  	void set_show(int x, A &a);    //该函数是类A的友元函数
  };
  
  class A
  {
  public:
  	friend void B::set_show(int x, A &a);   //该函数是友元成员函数的声明
  
  private:
  	int data;
  	void show() { cout << data << endl; }
  };
  
  //只有在定义类A后才能定义该函数，毕竟，它被设为友元是为了访问类A的成员
  void B::set_show(int x, A &a)
  {
  	a.data = x;
  	cout << a.data << endl;
  }
  ```

### 友元的优劣
- 优点
	- 可以灵活地实现需要访问若干类的私有或受保护的成员才能完成的任务；

- 缺点
	- 一个类将对其非公有成员的访问权限授予其他函数或者类，会破坏该类的封装性，降低该类的可靠性和可维护性。

## 各种特殊的类
### 内部类
内部类与外部类没有任何关系；
内部类<font color=gree>受外部类public/protected/private约束</font>；

有内部类的概念是因为：
> 内部类主要是为了避免命名冲突；（内部类定义为public）
> 为了隐藏名称（内部类定义为private/protected）



为public属性

```cpp
class Base    
{             
public:       
	virtual void fun() = 0;
	class Impl; //这样写的最大好外：Base有好几种实现方法（当然Base得有一堆的virtual接口）；
};

//内部类得加上其外部类的类名；
//亦可继承其外部类；
class Base::Impl : public Base                           
{
private:
	//new add property.
}
Base::Impl obj;
obj.fun();
```



为private属性

```cpp
class Base    
{             
public:
	void fun1() {impl_->fun1();}
	void fun2() {impl_->fun2();}
private:
	class Impl; //这样写的最大好外：Base有好几种实现方法。（不能用vitrual，即继承）
	Impl *impl_;
};

class Base::Impl
{
public:
	void fun1();
	void fun2();
}
Base::Base() {impl_ = new Impl()} //如果Impl继承Base，那么构造对象时会发生死循环。
```





### 单例类

请看一种实现
```cpp
class Demo
{
public:
	static Demo* getInstance();

protected:
	Demo();  //构造要置非public，析构则是public
	static Demo* instance;
};


Demo* Demo::instance;
Demo* Demo::getInstance()
{
	if (instance == NULL) {
		instance = new Demo();
	}
	return instance;
}
```

这种实现呢？
```cpp
class Demo
{
public:
	static Demo* getInstance();
	static bool init();
	bool initResource();

protected:
	Demo();  //构造要置非public（还有赋值、拷贝构造），析构则是public
	static Demo* instance;
private:
	bool mBool;
};

Demo* Demo::instance;
Demo* Demo::getInstance()
{
	return instance;
}
bool Demo::init()
{
	if (instance == NULL) {
		instance = new Demo();
	}
	//init other resource.
	instance->initResource();  //rskill要用对象去调用非静态函数；（类静态函数只能调用静态函数、静态属性）
}
bool Demo::initResource()
{
	mBool = true;
}
Demo::init();
Demo *obj = Demo::getInstance();
```

## 类内部关系

### static

static属性的变量、方法均属于类，而非对象；但这些static属性、方法却也属于对象（故可以对象调用static方法、对象可以访问static属性）；
能被继承，但是<font color=gree>与virtual相违背（编译不能通过，没有this指针）</font>；

```cpp
class Base{
public:
	static int s_int;
    static void s_print() {
        //只能处理static的属性、调用static的方法；
    }
    void comman_fun() {
        //可处理static的属性、调用static的方法；
    }
}

int Base::s_int = 0;  //类外初始化，前缀不能再加static
void Base::s_print() {…}  //类外，前缀不能再加static
```

### enum

枚举的使用记得带上类名；包括（枚举变量、枚举常量）  

```cpp
class Base
{   
public:
	enum STATE { UNINITIALIZED, STARTING, STARTED, JOINING, STOPPING, STOPPED };
};  
Base::STATE state; //定义带上类名。
state = Base::UNINITIALIZED;  //使用带上类名，当作域作用符吧。
```



## 类与类的关系
### 继承

virtual

```CPP
class BB : public AA {
public:
	virtual void v_print() override;
}

void Base::v_print() {..}  //类外，前缀不能再加 virtual
```



### 组合

