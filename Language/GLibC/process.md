[toc]

## ReadMe

## fork
```cpp
#include <unistd.h>
pid_t fork(void);
```

## 警惕
如果下面三个分支，不用{}限定的话，很有可能A跑完跑B，B跑完跑C；
因为父、子进程都是从04行开始执行的；
```cpp
01 void fun()
02 {
03 	res = fork();
04 	if (res == 0) {
05 		//A
06 	}
07 	else if(res > 0) {
08 		//B
09 	}
10 	else {
11 		//C
12 	}
13 }
```

