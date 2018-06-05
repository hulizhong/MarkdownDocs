[TOC]

## ReadMe
JavaScript 是一种轻量级的编程语言，是属于网络的脚本语言！
JavaScript 是可插入 HTML 页面的编程代码。
```bash
//在html中插入js
	<script type="text/javascript"> alert("My First JavaScript"); </script> //老版本
	<script> alert("My First JavaScript"); </script> //新版本

//变量：
var name = value;
var name;  //声明和赋值可分开
name = value;
var name; //重声明，值还是value
name2 = value2; //如果您把值赋给尚未声明的变量，该变量将被自动作为全局变量声明。

//函数
function myFunction(var1,var2)
{
	//这里是要执行的代码
}

//for循环
for (var i=0; ..) //这是常见的一种；
for (item in varName) //这在js里面也是可以的；

//异常机制
try {
  //throw();
}
catch(err){
  //在这里处理错误
}
```

- 入门语法墙
	- 注释// /**/
	- 行尾分号;可选
	- 大小写敏感
	- 代码折行'\'
	- 变量拥有动态类型；（同于Python）
		- Number：只有一种类型（可带小数点）
		- String
		- Boolean： true/false
		- Array： var cars = new Arrary(); cars[0] = "本田";
		- Object：{key:value, ...}
		- Undefined：表示变量现在无值；
		- null：可用于清空变量值；
		- 变量生命周期
			- 局部变量：函数内声明；
			- 全局变量：函数外声明，从声明处到页面关闭；
	- 一切皆对象（包含变量）：可用.访问其属性、方法；
	- 运算符号
		- ++/==/+=/-=/..
		- +可以用于数字、字符串、数据和字符串；
		- 判等：==/===全等（值和类型）
	
	
## JQuery	
- jQuery
	- 是一个 JavaScript 库。jQuery 极大地简化了 JavaScript 编程。

