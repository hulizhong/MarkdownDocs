[TOC]

## ReadMe
boost filesystem相关类的用法；

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

```cpp
//一般用简写的fs代替；
namespace fs = boost::filesystem;
```


## path wpath
path的成员方法
```cpp
#include <boost/filesystem/path.hpp>

//------------构造函数
boost::filesystem::basic_path<std::string>  
boost::filesystem::basic_path<std::wstring>

//--------------------------方法
path("/root/temp/a.out");
path.string(); //path to string.
path.filename(); //a.out
path /= std::string; //追加目录
path.has_parent_path()  //返回是否具有父目录；
path.parent_path()  //返回/a/b
```

拼接目录、文件名
```cpp
std::string dirPath("dir1");
dirPath.append("dir2");  //而且跨平台呢？win, linux下目录分隔符是不一样的；

std::filesystem::path dirPath("dir1");
dirPath /= "dir2";   // dir1/dir2 or dir1\dir2
```


## 文件操作
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


## 目录操作
方法
```cpp
boost::filesystem::is_directory(path);
boost::filesystem::create_directories(path);
```

### Directory Traversal 
```cpp
;
```
