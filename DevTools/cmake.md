[toc]

## 常用
日志、调试

- message("Will pring val = ${val}")


-----
添加头文件搜索路径CMAKE\_INCLUDE\_PATH
添加头文件搜索路径CMAKE\_LIBRARY\_PATH

- set(CMAKE\_INCLUDE\_PATH "include\_path")
	- list(APPEND CMAKE\_INCLUDE\_PATH "include\_path")
- set(CMAKE\_LIBRARY\_PATH "lib\_path")
	- list(APPEND CMAKE\_LIBRARY\_PATH "lib\_path") 


## finder
> https://blog.csdn.net/bytxl/article/details/50637277
> https://blog.csdn.net/dbzhang800/article/details/6329314


### 使用（整体框架）
一般使用模块配置的方法，其使用框架如下：

- 在顶层CMakeLists.txt
	- 定义${CMAKE\_MODULE\_PATH}，用以限定模块配置的寻找路径；
	- 使用find\_package(xxyy)，开始寻找Findxxyy.cmake；
- 在CMAKE\_MODULE\_PATH定义的目录中添加
	- Findxxyy.cmake
- 在Findxxyy.cmake中实现
	- 头文件查找，并设置变量_INCLUDE_DIRS；
	- 库查找，并设置变量_LIBRARIES；
- 在顶层CMakeLists.txt
	- 判断xxyy_FOUND是找到了包；
	- 如果找到（也可以在子工程里面进行include, link）
		- include_directories(${xxyy_INCLUDE_DIRS})
		- target_link_libraries(${xxyy_LIBRARIES})
	- 如果未找到
		- 看find_package设置的动作而定（告警输出、退出、。。）


### find\_package()
在编译过程中使用了外部库，但并不知道头文件、库的位置，可以用此命令来解决；

查找finder.cmake文件并执行它

- 模块模式（在以下目录查找 Find<xx>.cmake 文件并执行它）
	- CMAKE\_MODULE\_PATH
	- cmake\_install\_path\share\cmake-3.11\Modules
- 配置模式（查找 <Name>Config.cmake 或者 <lower-case-name>-config.cmake 文件并执行它）
	- 这些.cmake配置文件是安装软件、开发库时自带安装上的；（安装在各自的目录里）
	- 但目前很少有开发库安装这些配置的；


> FIND\_PACKAGE(
> 	&lt;name&gt; 
>   [version] [EXACT]
>>  版本格式：major[.minor[.patch[.tweak]]]
>>  EXACT版本精确匹配
>
>   [QUIET]
>>  不会告警（未找到包时），对应于Findxxyy.cmake中的NAME\_FIND\_QUIETLY变量；
>
>   [NO\_MODULE] 
>>  跳过模块模式；
>
>   [ REQUIRED|COMPONENTS [componets list... ]]
>>  REQUIRED：包是必须的，如未找到那么会退出cmake，对应.cmake模块中的DNAME\_FIND\_REQUIRED；
>>  COMPONENTS：其后跟包的依赖、相关部件（有些库不是一个整体，还包含一些依赖的库或者组件）；
>
> )



### finder.cmake的编写

finder.cmake文件编写流程如下：

- 一般定义如下变量
	- &lt;NAME&gt;_INCLUDE_DIRS
		- &lt;NAME&gt;_INCLUDES，与上面的变量有何区别，rwhy
	- &lt;NAME&gt;_LIBRARIES
		- &lt;NAME&gt;_LIBS，与上面的变量有何区别，rwhy
	- &lt;NAME&gt;_DEFINITIONS
	- &lt;NAME&gt;_FOUND
- find\_path() 搜索包含某个头文件的路径
	- find\_path(&lt;VAR&gt; name1 [path1 path2 ...])在各path中寻找name1
		- 如果找到，将path赋值到变量var中；
		- 如果没找到，var的值为&lt;VAR&gt;-NOTFOUND
	- PATHS/HINTS参数，寻找的路径列表；
		- hints指由系统内省时计算得出的路径，比如由其它已经发现的项目提供的线索。
	- ENV var参数，是将环境变量var转成cmake的一个list
	- PATH\_SUFFIXES参数，指定了每个搜索路径下的附加子路径；
- find\_library()
	- 命令 FIND\_LIBRARY 同 FIND\_PATH 类似,只是用于查找库并将结果保存在变量中。
- find\_package\_handle\_standard\_args()
	- 设置<name>_FOUND变量，并打印一条成功或者失败的消息。

---
且看如下这个FindLibxtml.cmake是如何写的；
```bash
find_package(PkgConfig)
pkg_check_modules(PC_LIBXML QUIET libxml-2.0)
set(LIBXML2_DEFINITIONS ${PC_LIBXML_CFLAGS_OTHER})

find_path(LIBXML2_INCLUDE_DIR libxml/xpath.h
	  HINTS ${PC_LIBXML_INCLUDEDIR} ${PC_LIBXML_INCLUDE_DIRS}
	  PATH_SUFFIXES libxml2 )

find_library(LIBXML2_LIBRARY NAMES xml2 libxml2
		 HINTS ${PC_LIBXML_LIBDIR} ${PC_LIBXML_LIBRARY_DIRS} )

set(LIBXML2_LIBRARIES ${LIBXML2_LIBRARY} )
set(LIBXML2_INCLUDE_DIRS ${LIBXML2_INCLUDE_DIR} )

include(FindPackageHandleStandardArgs)
# handle the QUIETLY and REQUIRED arguments and set LIBXML2_FOUND to TRUE
# if all listed variables are TRUE
find_package_handle_standard_args(LibXml2  DEFAULT_MSG
							  LIBXML2_LIBRARY LIBXML2_INCLUDE_DIR)

#将缓存的变量标记为高级变量。其中，高级变量指的是那些在cmake GUI中，只有当“显示高级选项”被打开时才会被显示的变量
mark_as_advanced(LIBXML2_INCLUDE_DIR LIBXML2_LIBRARY )
```

## cpack打包

### 打成deb包
可以打成debian系列的.deb包；

### 打成rpm包
可以打成redhat系列的.rpm包；

