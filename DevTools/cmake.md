[toc]

## cmake

### 常用
日志、调试
- message("Will pring val = ${val}")


-----
添加搜索路径
- set(CMAKE_INCLUDE_PATH "include_path")
	- list(APPEND CMAKE_INCLUDE_PATH "include_path")
- set(CMAKE_LIBRARY_PATH "lib_path")
	- list(APPEND CMAKE_LIBRARY_PATH "lib_path") 


### finder
> https://blog.csdn.net/bytxl/article/details/50637277
> https://blog.csdn.net/dbzhang800/article/details/6329314


使用（整体框架）
- 在顶层CMakeLists.txt
	- 定义${CMAKE_MODULE_PATH}变量
	- 使用find_package(xxyy)
- 在CMAKE_MODULE_PATH定义的目录中添加
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


--- 
find_package()
在编译过程中使用了外部库，但并不知道头文件、库的位置，可以用此命令来解决；

查找finder.cmake文件并执行它
- 模块模式（在以下目录查找 Find<xx>.cmake 文件并执行它）
	- CMAKE_MODULE_PATH
	- cmake_install_path\share\cmake-3.11\Modules
- 配置模式（查找 <Name>Config.cmake 或者 <lower-case-name>-config.cmake 文件并执行它）
	- 这些.cmake配置文件是安装库时安装上的；（安装在各自的目录里）
	- 但目前很少有库安装这些配置的；


> FIND_PACKAGE(
> 	&lt;name&gt; 
>   [version] [EXACT]
>>  版本格式：major[.minor[.patch[.tweak]]]
>>  EXACT版本精确匹配
>
>   [QUIET]
>>  不会告警（未找到包时），对应于Findxxyy.cmake中的NAME_FIND_QUIETLY变量；
>
>   [NO_MODULE] 
>>  跳过模块模式；
>
>   [ REQUIRED|COMPONENTS [componets list... ]]
>>  REQUIRED：包是必须的，如未找到那么会退出cmake，对应.cmake模块中的DNAME_FIND_REQUIRED；
>>  COMPONENTS：其后跟包的依赖、相关部件（有些库不是一个整体，还包含一些依赖的库或者组件）；
>
> )



----
finder.cmake的编写
- 一般定义如下变量
	- <NAME>_INCLUDE_DIRS
		- <NAME>_INCLUDES
	- <NAME>_LIBRARIES
		- <NAME>_LIBS
	- <NAME>_DEFINITIONS
	- &lt;NAME&gt;_FOUND
- find_path() 搜索包含某个文件的路径
	- find_path(<VAR> name1 [path1 path2 ...])
- find_library()
	- 命令 FIND_LIBRARY 同 FIND_PATH 类似,用于查找链接库并将结果保存在变量中。
- find_package_handle_standard_args()
	- 设置<name>_FOUND变量，并打印一条成功或者失败的消息

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

### 打包cpack

