

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

### boost\_scope\_exit

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


## 文件系统 
目录架构   
```bash
▾ filesystem/       
  ▸ detail/         
    config.hpp      
    convenience.hpp 
    exception.hpp   
    fstream.hpp     
    operations.hpp  
    path.hpp        
    path_traits.hpp 
```

收录如下信息：
[path wpath](#path_wpath)
[文件操作](#文件操作)
[](#)

```cpp
//一般用简写的fs代替；
namespace fs = boost::filesystem;
```

### path wpath
```cpp
boost::filesystem::basic_path<std::string>  
boost::filesystem::basic_path<std::wstring>

boost::filesystem::is_directory(path);
boost::filesystem::create_directories(path);
```

path类提供的成员函数
```cpp
path pt("/a/b");
pt /= "c.txt" 追加，现在成/a/b/c.txt

pt.has_parent_path() 返回是否具有父目录；
pt.parent_path()  返回/a/b
```

### 文件操作
写文件
```cpp
boost::filesystem::ofstream file(path);
if (file.is_open()) {
	file << data;
	file.close();
}
```

读文件
```cpp
boost::filesystem::ifstream file(path);
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


