[TOC]

## 入门

```bash
标签关键字：大小写不敏感、一般用小写；

<!--> 注释 <-->

- 标签元素
	- <a href="http://www.w3school.com.cn">This is a link</a>  链接
		- 链接由<a>定义
		- href为属性，属性一般为key=value形式给出；
		- This is a link为<a>标签的内容
	- <h1>This is a heading</h1> 标题，1-6级
	- <p>This is a paragraph.</p> 段落
	- <img src="w3school.jpg" width="104" height="142" />  图片
	- <html> ... </html>
	- <body> ... </body> 
	- <table> .. </table> 表格
	- <div>, <span> 块
	
	- 属性
		- style 样式
			- 一般为内部、内联样式；
			- 外部样式像这样 <link rel="stylesheet" type="text/css" href="mystyle.css">
		- href
			- 绝对 URL - 指向另一个站点（比如 href="http://www.example.com/index.htm"）
			- 相对 URL - 指向站点内的某个文件（href="index.htm"）
			- 锚 URL - 指向页面中的锚（href="#top"）
		
	- 一些小意思
		- <hr /> 水平线
		- <br /> 换行
```



## html中嵌入js
在Html中嵌入javascript脚本，不需要额外include头；
只需要在添加的位置加个Script标签，并在其内部实现逻辑、调用；
注意：在script外部调用script内部的函数是没有意义的，会被当作一个普通字符串；
```javascript
//<body>
	<Script language="javascript">
		function GetFileName() { //函数定义；
			var href = location.href.split("/");
			var name = href[href.length - 1];
			document.title = name;
		}
		GetFileName(); //调用一定在script标签内部，方可有意义；
	</Script>
//</body>
```

