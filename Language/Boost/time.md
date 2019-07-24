[TOC]



## ReadMe

boost.date, time, timer.



## date_time

T. second_clock, microsec_time_clock

```cpp
#include <boost/date_time/microsec_time_clock.hpp> //microsec_clock 纳秒级时钟；
#include <boost/date_time/time_clock.hpp> //second_clock 提供秒级时钟
```



### posix time

posix_time是与时区无关的时间。

T. ptime时间点

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
using boost::posix_time;

ptime pt(second_clock::local_time());     //机器的当前时间；
	//boost::date_time::second_clock<ptime>
ptime pt(second_clock::universal_time()); //机器的GMT时间；

to_simple_string(pt);       //2019-Jul-23 03:59:52
to_iso_string(pt);          //20190723T035952
to_iso_extended_string(pt); //2019-07-23T03:59:52
ptime pt = from_iso_string("20121101T202020");
ptime pt = time_from_string("2012-3-5 01:00:00");

//boost/date_time/posix_time/conversion.hpp
std::time_t to_time_t(ptime pt);
std::tm to_tm(const boost::posix_time::ptime& t);
std::tm to_tm(const boost::posix_time::time_duration& td);
ptime ptime_from_tm(const std::tm& timetm);
```

T.time_duration时间长度

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
//time_duration = ptime - ptime;
//ptime = ptime +|- time_duration;

boost::posix_time::time_duration td(1, 2, 3, 4); //1h,2min,3s,4micros
to_simple_string(td);
to_iso_string(td);

const boost::posix_time::time_duration td = now.time_of_day();
int hh = td.hours();
int mm = td.minutes();
int ss = td.seconds();
td.total_seconds();
td.total_milliseconds();
td.total_microseconds(); // == td.ticks()
td.totao_nanoseconds();

boost::posix_time::hours h(1);
boost::posix_time::minutes m(10);    
boost::posix_time::seconds s(30);    //s
boost::posix_time::millisec ms(1);   //毫s = 000
boost::posix_time::microsec mcs(1);  //微s = 000000
boost::posix_time::nanosec ns(1);    //纳s = 000000000，这个得开编译选项；
```



### local time

local_time是于时区相关的时间；

```cpp
#include <boost/date_time/local_time/local_time.hpp>
```





## origianl timers

包含如下几个timer，<font color=red>但都已被弃用</font>，请用最新的cpu timers；

```cpp
timer; //timer.hpp
progress_timer;   //progress.hpp
progress_display; //progress.hpp
```



## cpu timers



### cpu_timer

```cpp
#include <boost/timer/timer.hpp>

boost::timer::cpu_timer t;
t.start(); //开始计时
//...
t.stop(); //停止计时
std::cout << t.format() << std::endl;
if (t.is_stopped()) {
    //是否调用了stop();
    t.resume();  //如果is_stopped为true,则为恢复累计额外计时；为false则无效；
}
t.elapsed();  //从start至今的时差；（或者start到stop间的时长）
t.elapsed().wall;   //总的时长；
t.elapsed().user;   //用户时间；
t.elapsed().system; //系统时间；
```



### auto_cpu_timer

继承自`cpu_timer`, 更为精细的控制。
对象创建即开始计时，被销毁时自动报告花费时长。

```cpp
#include <boost/timer/timer.hpp>
//auto_cpu_timer(..); //参数places(精度)，format(格式)，ostream(输出流)

boost::timer::auto_cpu_timer t(4, "%w");
t.report(); //从start()到现在的时长报告，stop()之后调用无效；
```



## async timer

### steady_timer 

参见：asio.md/steady_timer.





