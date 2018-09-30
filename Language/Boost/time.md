[TOC]



## ReadMe

https://blog.csdn.net/hongjunbj/article/details/9271177

ptime - ptime = time_duration

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
 
        // Get current time from the clock, using microseconds resolution
       const boost::posix_time::ptime now = boost::posix_time::microsec_clock::local_time();
       // Get the time offset in current day
       const boost::posix_time::time_duration td = now.time_of_day();
       int hh = td.hours();
       int mm = td.minutes();
       int ss = td.seconds();
       int ms = td.total_milliseconds() - ((hh * 3600 + mm * 60 + ss) * 1000);


```

