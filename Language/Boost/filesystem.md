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
#include <boost/filesystem.hpp>     // g++ src -lboost_filesystem
namespace bfs = boost::filesystem;  //一般用简写的bfs代替；

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

`path, wpath`表示路径的信息，并提供了处理路径的方法。

**T.构造路径的选择。**

注意点：

- path的构造字符串，不会进行任何校验。
- 构造字符串同时支持可移植路径（用`/`分隔）、平台相关路径。

```cpp
typedef boost::filesystem::basic_path<std::string> boost::filesystem::path;
typedef boost::filesystem::basic_path<std::wstring> boost::filesystem::wpath;

boost::filesystem::path p("dir1/dir2/filename.doc");   //可移植路径；
boost::filesystem::path p("dir1\\dir2\\filename.doc"); //平台相关路径；
	//不具有可移植性，'\\'为windows专用，不能在posix兼容的系统上正常work.但仅能在win平台上work.
		//p.filename() 在windows-OS会得到 'filename.doc'
		//p.filename() 在POSIX兼容的OS会得到 'dir1\dir2\filename.doc'
```



**T.path的成员方法**

如下：

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
// .append(string), /= 是等效的，但不等于+=.
bfs::path p("dir1");

p.append("dir2");  // dir1/dir2
p /= "dir2";   // dir1/dir2

p.concat("dir2");  //dir1dir2
p += "dir2";  // dir1dir2
```



### native vs generic path

`Native pathname format` VS `Generic pathname format`.

> 通用path只能用`/`做分隔符。
> 本地path与通用path相似，只是目录分隔符可以是`/`, `\`斜杠。（注意反斜杠有着转义的意思，所以需要俩）



```cpp
// ------------------------ win下native.（注意很多返回只是个观察者，并不能改变原有的path.）
std::wstring = p.native();  // /root/temp/file.pdf
	//win平台下只是用，wstring来装，但目录结构并不是win特有的。
bfs::path pp = p.make_preferred(); // \root\temp\file.pdf
	//path& make_preferred(); 操作pp会更改原变量p.
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
bool create_directory(const path& p);    //不可创建多级不存在的目录（异常）
bool create_directories(const path& p);  //可创建多级不存在的目录

bool remove(const path& p);  //删除文件p
uintmax_t remove_all(const path& p); //递归删除p，返回删除文件的数量

boost::rename(const Path1& from_p, const Path2& to_p);      //重命名
boost::copy_file(const Path1& from_fp, const Path2& to_fp); //拷贝文件
void copy_directory(const path& from, const path& to);
```

### Directory Traversal 
```cpp
;
```







