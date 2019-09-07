[TOC]

## ReadMe

boost中文件系统相关操作；https://www.boost.org/doc/libs/1_53_0/libs/filesystem/doc/reference.html

```bash
▾ filesystem/       
  ▸ detail/         
    config.hpp      
    convenience.hpp 
    exception.hpp   
    fstream.hpp     #文件内容相关操作
    operations.hpp  #文件、目录相关操作
    path.hpp        #路径相关操作
    path_traits.hpp 
```

```cpp
//一般用简写的bfs代替；
namespace bfs = boost::filesystem;

try {
    //多数函数存在两个变体：在失败时:
    	//一个会抛出类型为 boost::filesystem::filesystem_error 的异常。
    	//一个则返回类型为 boost::system::error_code 的对象。 
}
catch (boost::filesystem::filesystem_error &e) {
    e.what();
    e.code();
    e.path1();
    e.path2();
}
```


## path wpath
path的成员方法（**相当于一些字符串的操作，未涉及到文件的操作**）。
```cpp
#include <boost/filesystem/path.hpp>

//------------构造函数
bfs::basic_path<std::string>  
bfs::basic_path<std::wstring>

bfs::path p("/root/temp/file.pdf");

p.string();  //path to string.
p += "+=str";  //追加字符串，root/temp/file.pdf+=str
p /= str;      //追加目录, /root/temp/file.pdf/str

//--------------------------文件名处理
p.stem();      //file
p.filename();  //file.pdf
p.extension(); //.pdf

//--------------------------目录处理
p.root_name();   //
p.root_directory(); //get /
p.root_path();      //get /
p.relative_path();  //get root/temp/file.pdf
p.parent_path();    //get /root/temp
p.has_parent_path(); //get true or false.
p.parent_path();     //get /root/temp
```

拼接目录、文件名
```cpp
std::string dirPath("dir1");
dirPath.append("dir2");  //而且跨平台呢？win, linux下目录分隔符是不一样的；

std::filesystem::path dirPath("dir1");
dirPath /= "dir2";   // dir1/dir2 or dir1\dir2
```


## 文件操作




```cpp
void copy_file(const path& from, const path& to, copy_option option);
	//目标存在会抛出异常。
	//copy_file(source_path, destination_path, copy_option::overwrite_if_exists);
```



### 文件属性

```cpp
file_status status(const path& p);
	//不同于glibc.stat()这里面只有file_type & perms.
	//file_type type();
		//status_error, file_not_found, regular_file, fifo_file, .., type_unknown.
	//perms permissions();
		//no_perms, owner_read, .., symlink_perms.

uintmax_t file_size(const path& p);
bool is_directory(const path& p);
bool is_empty(const path& p);
bool is_regular_file(const path& p);
bool is_symlink(const path& p);
std::time_t last_write_time(const path& p);
void permissions(const path& p, perms prms);
```



### 文件内容

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



```cpp
//fstream;
boost::filesystem::fstream fstream(fpath, std::ios_base::out);
```





## 目录操作

方法
```cpp
bool is_directory(const path& p);
bool create_directory(const path& p);    
bool create_directories(const path& p);  //创建多级不存在的目录

bool remove(const path& p);  //删除文件p
uintmax_t remove_all(const path& p); //递归删除p

void copy_directory(const path& from, const path& to);
```

### Directory Traversal 
```cpp
;
```







