[TOC]



## ReadMe

安装

```bash
apt-get install liblog4cxx10-dev
```



demo

```cpp
LOG4CXX_DEBUG(logger, message);
	//…INFO …
LOG4CXX_DEBUG(logger, “one ..” << “two ..”);
	//用<<连接多个日志字段
```





## About Log4cxx

三个重要的概念：logger, appender, layout.

### Logger

logger是具有层次的，顶层的rootlogger一直会存在，如下；

```cpp
log4cxx::Logger::getRootLogger(); //获得rootLogger.
log4cxx::Logger::getLogger(); //获得其它命令logger.
```

logger中带有日志级别，如果没有设置那么继承自上层logger；

```cpp
log4cxx::Level::getDebug();
	//TRACE, DEBUG, INFO, WARN, ERROR, FATAL.
```





### Appender

日志输出目标，可以是多个目标。(console, file, db, syslog ..)
可以是同步输出、异步输出。

下面是fileAppender

```cpp
FileAppender
RollingFileAppender
DailyRollingFileAppender
```



往logger中指定appender

```cpp
//Logger与appender绑定
log4cxx::LoggerPtr Log::logger = log4cxx::Logger::getLogger("rabin");
log4cxx::helpers::Properties props;
props.put("log4j.logger.rabin", "DEBUG, console");
props.put("log4j.appender.console", "org.apache.log4j.ConsoleAppender"); //console
//…
log4cxx::PropertyConfigurator::configure(props);


//Root logger与appender绑定：
log4cxx::LoggerPtr Log::logger = log4cxx::Logger::getRootLogger();
log4cxx::helpers::Properties props;
props.put("log4j.rootLogger", "DEBUG, file");
props.put("log4j.appender.file", "org.apache.log4j.FileAppender"); //file
//….
log4cxx::PropertyConfigurator::configure(props);
```



appender的属性

```cpp
log4cxx::helpers::Properties props;
props.put("log4j.appender.xx", "org.apapche.log4j.RollingFileAppender");
	//通过key, value的形式添加
	//File 文件名
	//MaxFileSize 文件大小B/KB/MB/GB
	//Append 追加模式
```





### Layout

指定日志输出格式，如下demo

```cpp
log4cxx::helpers::Properties props;
std::string logPattern = "%d [%t] %-5p %.32c - %m%n";
props.put("log4j.appender.file.layout", "org.apache.log4j.Patternlayout");
props.put("log4j.appender.file.layout.ConversionPattern", logPattern);
```

| 转换字符 | 表示的意思                                                   |
| -------- | ------------------------------------------------------------ |
| c        | 用于输出的记录事件的类别。例如，对于类别名称"a.b.c" 模式  %c{2} 会输出 "b.c" |
| C        | 用于输出呼叫者发出日志请求的完全限定类名。例如，对于类名 "org.apache.xyz.SomeClass", 模式 %C{1} 会输出 "SomeClass". |
| d        | <font color=red>用于输出的记录事件的日期</font>。例如， %d{HH:mm:ss,SSS} 或 %d{dd MMM yyyy HH:mm:ss,SSS}. |
| F        | 用于输出被发出日志记录请求，其中的文件名                     |
| l        | 用于将产生的日志事件调用者输出位置信息                       |
| L        | 用于输出从被发出日志记录请求的行号                           |
| m        | 用于输出使用日志事件相关联的应用程序提供的消息，即<font color=red>日志消息</font>。 |
| M        | 用于输出发出日志请求所在的方法名称                           |
| n        | 输出平台相关的行分隔符或文字                                 |
| p        | 用于输出的记录事件的优先级                                   |
| r        | 用于输出毫秒从布局的结构经过直到创建日志记录事件的数目       |
| t        | <font color=red>用于输出生成的日志记录事件的线程的名称</font> |
| x        | 用于与产生该日志事件的线程相关联输出的NDC（嵌套诊断上下文）  |
| X        | 在X转换字符后面是键为的MDC。例如  X{clientIP} 将打印存储在MDC对键clientIP的信息 |
| %        | 文字百分号 %%将打印％标志                                    |

格式控制

| Format modifier | left justify | minimum width | maximum width | 注释                                                         |
| --------------- | ------------ | ------------- | ------------- | ------------------------------------------------------------ |
| %20c            | false        | 20            | none          | 用空格左垫，如果类别名称少于20个字符长                       |
| %-20c           | true         | 20            | none          | 用空格右垫，如果类别名称少于20个字符长                       |
| %.30c           | NA           | none          | 30            | 从开始截断，如果类别名称超过30个字符长                       |
| %20.30c         | false        | 20            | 30            | 用空格左侧垫，如果类别名称短于20个字符。但是，如果类别名称长度超过30个字符，那么从开始截断。 |
| %-20.30c        | true         | 20            | 30            | 用空格右侧垫，如果类别名称短于20个字符。但是，如果类别名称长度超过30个字符，那么从开始截断。 |



### Async Appender

/home/pro/helloserver/util/Log.cpp







因为异步写日志会先写到buffer中，然后再会触发到磁盘的过程。
所以验证的时候可以构造buffer中日志没来得及输出的情况。（同步输出日志的话不会有这个情况，即日志输出语句调用返回意味着日志内容已经落地了）具体如下：

```cpp
log_info("1");
std::shared_ptr<int> pi;
log_info("2");
*pi = 10;  //程序崩溃，异步日志的情况下1, 2是没有写到日志文件中的；
log_info("3");
```



## Demo





## Experiences

**各独立运行模块间的log4cxx是独立的！**

A.exe 调用 b.lib，a.exe用了初始化了log4cxx，库b.lib中也初始化了log4cxx，他俩是各管各的不冲突；





