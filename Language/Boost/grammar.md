[toc]

## ReadMe
boost

- 是std的补充（TR1, C++11, C++14很多特性都是从boost库来的）。
- 什么时候用boost；
	- std不够用了；
	- gcc太老不支持std=c++11/14之类的；
- 学习方法
	- 它是很多库的集合，只需要做到用哪些库看哪些库。
	- 源码很复杂，会用就行；（到后期发展需要或者好奇心可以看下源码）


## RAII
构造函数时资源申请、析构函数时资源释放；

### boost\_scope\_exit
退出时调用{}中的内容；

demo
```cpp
#include <boost/scope_exit.hpp>
int fun()
{
	int fd = open("tst.md", "rb");
	//1. fd此时为按值传递（捕获），亦可按引用传递（捕获）；多个参数之间用逗号分隔；
	//2. 引用捕获时有些编译器不支持指针的引用捕获；
	//3. 类成员函数捕获类对象本身用this_；
	BOOST_SCOPE_EXIT(fd){
		//当程序离开fun()函数时，执行exit,exit_end之间的代码；
		close(fd);
	}BOOST_SCOPE_EXIT_END
}
```


## uuid
源码目录架构   
```bash
▾ uuid/                 
  ▸ detail/             
    name_generator.hpp  
    nil_generator.hpp   
    random_generator.hpp
    seed_rng.hpp        
    sha1.hpp            
    string_generator.hpp
    uuid.hpp            
    uuid_generators.hpp 
    uuid_io.hpp         
    uuid_serialize.hpp  
```

生成uuid
```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <boost/uuid/uuid_generators.hpp>
 
boost::uuids::uuid a_uuid = boost::uuids::random_generator()();
	// 这里是两个() ，因为这里是调用的 () 的运算符重载.
const string str_uuid = boost::uuids::to_string(a_uuid);
	//uuid转string.
```

## 特殊功能类
收录一些功能类，我们只需要继承这些类就能实现特定功能，如下：
[noncopyable](#boost::noncopyable)

### boost::noncopyable
禁止拷贝类
主要看点：拷贝构造函数、=重载符 置为私有的属性。如下：  
```cpp
class noncopyable
{
 protected: //为什么是protected？
    noncopyable() {}
    ~noncopyable() {}
 private:  // emphasize the following members are private
    noncopyable( const noncopyable& ); 
    const noncopyable& operator=( const noncopyable& );
};
```

