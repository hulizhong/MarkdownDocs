

## RAII

### boost\_scope\_exit

```cpp
#include <boost/scope_exit.hpp>
int fun()
{
	int fd = open("tst.md", "rb");
	//1. fd此时为按值传递（捕获），亦可按引用传递（捕获）；多个参数之间用逗号分隔；
	//2. 引用捕获时有些编译器不支持指针的引用捕获；
	//3. 类成员函数捕获类对象本身用this_；
	BOOST_SCOPE_EXIT(fd){
		//当程序离开fun()函数时，执行exit,exit_end之间的代码；
		close(fd);
	}BOOST_SCOPE_EXIT_END
}
```


