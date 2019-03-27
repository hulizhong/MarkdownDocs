[TOC]

## ReadMe
cmake输入文件：<-------   CMakeLists.txt （注意是Lists）
mkdir build; cd build; cmake ..;   生成Makefile；
make -j 4;



## Grammar

如下demo所示：

```bash
# comment use '#'.
# end the statement no ';'.

#define variable.
set(VarName VarValue)

#use variable. `${varName}`
include_directories(${PROJECT_SOURCE_DIR}) 

# log.
message("Will pring val = ${val}")
```



### flow control

包含if, while, foreach, ..



### 文件管理

```cmake
add_executable(obj f1.cpp f2.cpp)
	#最简单的方法，就是这样后缀。

aux_source_directory([Path] [Variable])
	#获取一个目录下的的所有源文件 然后储存在变量Variable中
	
FILE(GLOB var ./include/*.h)
	#GLOB 获取./include/目录下的*.h 储存在变量var中
FILE(GLOB_RECURSE var ./source/*.cpp)
	#GLOB_RECURSE 递归获取./include/目录和子目录下的*.cpp 储存在变量var中

```

备注：file的glob, glob_recurse选项；
GLOB选项将会为所有匹配查询表达式的文件生成一个文件list，并将该list存储进变量variable里。
GLOB_RECURSE选项将会生成一个类似于通常的GLOB选项的list，只是它会寻访所有那些匹配目录的子路径并同时匹配查询表达式的文件。作为符号链接的子路径只有在给定FOLLOW_SYMLINKS选项或者cmake策略CMP0009被设置为NEW时，才会被寻访到。



--------

**if**

Notice. 判断的变量不需要使用`${var}`格式，而是直接使用`var`.





--------

**while**



-----------

**foreach**






## finder
> https://blog.csdn.net/bytxl/article/details/50637277
> https://blog.csdn.net/dbzhang800/article/details/6329314

包含如下章节：
[使用（整体框架）](#使用（整体框架）)
[find\_package()](#find_package指令)
[finder.cmake的编写](#finder.cmake的编写)

### 使用（整体框架）
一般使用模块配置的方法，其使用框架如下：

- 在顶层CMakeLists.txt
	- 定义${CMAKE\_MODULE\_PATH}，用以限定模块配置的寻找路径；
	- 使用find\_package(xxyy)，开始寻找Findxxyy.cmake；
- 在CMAKE\_MODULE\_PATH定义的目录中添加
	- Findxxyy.cmake
- 在Findxxyy.cmake中实现
	- 头文件查找，并设置`变量_INCLUDE_DIRS`；
	- 库查找，并设置`变量_LIBRARIES`；
- 在顶层CMakeLists.txt
	- 判断xxyy_FOUND是找到了包；
	- 如果找到（也可以在子工程里面进行include, link）
		- include_directories(${xxyy_INCLUDE_DIRS})
		- target_link_libraries(${xxyy_LIBRARIES})
	- 如果未找到
		- 看find_package设置的动作而定（告警输出、退出、。。）


### find\_package指令
在编译过程中使用了外部库，但并不知道头文件、库的位置，可以用此命令来解决；

查找finder.cmake文件并执行它

- 模块模式（在以下目录查找 Find<xx>.cmake 文件并执行它）
	- CMAKE\_MODULE\_PATH
	- cmake\_install\_path\share\cmake-3.11\Modules
- 配置模式（查找 <Name>Config.cmake 或者 <lower-case-name>-config.cmake 文件并执行它）
	- 这些.cmake配置文件是安装软件、开发库时自带安装上的；（安装在各自的目录里）
	- 但目前很少有开发库安装这些配置的；


> FIND\_PACKAGE(
> ​	&lt;name&gt; 
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
	- 设置<name>\_FOUND变量，并打印一条成功或者失败的消息。

---
且看如下这个FindLibxtml.cmake是如何写的；
```cmake
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

## 指令集

Todo. https://blog.csdn.net/z_h_s/article/details/50699905

指令如下：

| key                              | subkey       | value                                                        |
| -------------------------------- | ------------ | ------------------------------------------------------------ |
| project(XX)                      |              | 项目名称，${XX_SOURCE_DIR}则为项目根目录。                   |
| cmake_mini**m**um_required(..)   |              | cmake最低版本设置。(VERSION 2.8 FATAL_ERROR)                 |
| include(xx)                      |              | 引入cmake模块xx.                                             |
|                                  |              |                                                              |
| add_subdirectory(dir)            |              | 包含子目录dir中的CmakeLists.txt.                             |
| aux_source_directory(dir var)    |              | 发现dir下面所有源码文件、并将文件列表赋值给var.              |
| add_executable(obj codefilelist) |              | 生成二进制目标。                                             |
| add_library(obj codefilelist)    |              | 生成库目标。                                                 |
|                                  |              |                                                              |
| include_directories(xx)          |              | -I参数                                                       |
| link_directories(xx)             |              | -L参数                                                       |
| target_link_libraries(obj a b)   |              | 为obj指定-l库名，可以放在任何地址。                          |
| link_libraries(a b c)            |              | 为后续add_executable/add_library目标指定-l库，只能放在前面。 |
|                                  |              |                                                              |
| add_definitions(-DXX)            |              | 向c/c++编译器添加-DXX定义。                                  |
| cmake_c_flags                    |              | 预置变量：c编译器选项设置；                                  |
| cmake_cxx_flags                  |              | 预置变量：c++编译器选项设置；                                |
|                                  |              |                                                              |
| file(subkey ..)                  |              | 文件操作指令                                                 |
| install(subkey ..)               |              |                                                              |
| find_xx(..)                      |              | find_系列指令                                                |
|                                  | find_file    |                                                              |
|                                  | find_library |                                                              |
|                                  | find_path    |                                                              |
|                                  | find_program |                                                              |
|                                  | find_package |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |
|                                  |              |                                                              |



### find



### file



### install



### add\_definitions
option选项，让用户可以根据选项值进行条件编译。（可在`cmake .. -DUSE\_A=ON/OFF`进行功能开启、关闭）
这个option命令和你本地是否存在编译缓存的关系很大。所以，如果你有关于 option 的改变，那么请你务必清理 CMakeCache.txt 和 CMakeFiles 文件夹。

```cmake
OPTION(USE_A "just for test" ON)
OPTION(USE_A "just for test")  #如果没有提供初始化值，使用OFF。（同于下一句）
OPTION(USE_A "just for test" OFF)
message("the use_a value = ${USE_A}")  #要么打OFF，要么打ON


if (DEFINED USE_A)  #是否定义过use_a变量，不关心值的内容；
	#defined必须大写；
    add_definitions("-DUSE_MACRO")  #在gcc/g++时使用-Duse_macro宏定义；
endif()
if (USE_A STREQUAL "ON")  #use_a变量的值是否为on；（未定义此变量不会引起错误！）
	#strequal必须大写；
	#use_a也可用${USE_A}; 
endif()
 
add_executable(multimap ${MULTIMAP_SRC})
```
在以上例子中，实际应用中为use\_a, use\_macro是一样的名称；

```cmake
#ifdef USE_MACRO
    std::cout << "Code With USE_MACRO" << std::endl;
#endif
```





### Add header/lib Search Path.

添加头文件、库的搜索路径

```bash
set(CMAKE_INCLUDE_PATH "include_path")
list(APPEND CMAKE_INCLUDE_PATH "include_path")

set(CMAKE_LIBRARY_PATH "lib_path")
list(APPEND CMAKE_LIBRARY_PATH "lib_path") 
```

```bash
include_directories(dir1 dir2)  #找头文件

link_directories(dir1 dir2)  #找库文件
target_link_libraries(obj lib1 lib2)  #链接库
```



### output path

EXECUTABLE_OUTPUT_PATH

executable_output_path()

LIBRARY_OUTPUT_PATH

library_output_path()





## 宏

探测OS类型的宏
```cmake
if (NOT WIN32)
	不是win32
if (WIN32 OR APPLE)
	win32 或者 mac
else()
endif()
```



## With Visual Studio

**Q. cmake生成`projectname.sln` ？**这个sln文件就是visual studio的工程文件，可以用vs来跑。
CMakeLists.txt要用`VS2012 x86 Native Tools Command Prompt`来跑，这样才能跑出来工程的sln。
   step 1. mkdir build; cd build;
   step 2 with nmake.      cmake .. -G "NMake Makefiles";  nmake;
   step 2 with msbuild.    cmake ..; msbuild xx.sln;



### msbuild

MSBuild 是 Microsoft 和 Visual Studio 的生成平台。
msbuild的项目文件是xml定义的。也可以以参数的形式给到msbuild。

```bash
#--------------------------------sln方法
msbuild projectname.sln

#------------------------------------vcxproj方法
msbuild ALL_BUILD.vcxproj /p:Configuration=Release /p:Platform=x64 /p:MultiProcessorCompilation=true /p:PlatformToolset=v140_xp

msbuild ALL_BUILD.vcxproj /p:Configuration=Release /p:Platform=Win32 /p:MultiProcessorCompilation=true /p:PlatformToolset=v140_xp
```

参数

- /p, /property 后接属性；
- /t, /target 生成目标；

属性值

- Configuration
- Platform
- MultiprocessorCompilation
- PlatformToolset
- ...



### visual studio usage

