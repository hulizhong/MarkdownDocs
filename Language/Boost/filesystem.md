[TOC]

## ReadMe

boost中文件系统相关操作；

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
//一般用简写的bfs代替；
namespace bfs = boost::filesystem;
```


## path wpath
path的成员方法
```cpp
#include <boost/filesystem/path.hpp>

//------------构造函数
bfs::basic_path<std::string>  
bfs::basic_path<std::wstring>

//--------------------------方法
path("/root/temp/a.out");
path.string(); //path to string.
path.filename();  //a.out
path.extension(); //.out
path /= std::string; //追加目录
path.has_parent_path();  //返回是否具有父目录；
path.parent_path();  //返回/a/b
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
bfs::ofstream file(path);
if (file.is_open()) {
	file << data;
	file.close();
}
```

读文件
```cpp
bfs::ifstream file(path);
```



api list. https://www.boost.org/doc/libs/1_53_0/libs/filesystem/doc/reference.html#copy_option

```cpp
void copy_file(const path& from, const path& to, copy_option option);
	//目标存在会抛出异常。
	//copy_file(source_path, destination_path, copy_option::overwrite_if_exists);
```





## 目录操作

方法
```cpp
bfs::is_directory(path);
bfs::create_directories(path);
```

### Directory Traversal 
```cpp
;
```

