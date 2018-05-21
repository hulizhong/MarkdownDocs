[toc]

## 类内部关系
### 友元
允许一个类将对其非公有成员的访问权（protect, private）授予指定的函数或者类。
友元声明可以出现在类中的任何地方，而不受（public, protect, private）限制。

#### 各种友元
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
	友元函数：必须要有个类对象的引用。
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

- 友元成员函数

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

#### 友元的优劣
- 优点
	- 可以灵活地实现需要访问若干类的私有或受保护的成员才能完成的任务；

- 缺点
	- 一个类将对其非公有成员的访问权限授予其他函数或者类，会破坏该类的封装性，降低该类的可靠性和可维护性。


## 类与类的关系

