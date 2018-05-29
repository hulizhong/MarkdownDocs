[toc]

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
