[TOC]


要用的python的两个库
- Requests 向webserver发http请求
- pyquery 像jquery一样解析html


## Requests

手册： http://docs.python-requests.org/zh_CN/latest/user/quickstart.html




单个请求完全没必要用 Session。直接 requests.get(xxx) 就可以了。


问题：Resp.text() vs resp.content() ？？？
> resp.text()返回的是处理过的Unicode型的数据（库自己检测出来的）。  
>> 如果Requests 检测不到正确的编码，那么你告诉它正确的是什么  
>> response.encoding = 'gbk'

> resp.content()返回的是bytes型的原始数据。




## pyquery

手册： http://pythonhosted.org/pyquery/attributes.html

三种加载方法
```python
v_source=pq("")   ---直接加载一个html串
v_source=pq(filename=path_to_html_file)  ---加载位于指定路径下的html文件
v_source=pq(url='http://yunvs.com/list/mai_1.html')    ---加载url地址直接进行解析
```

特定标签的查找方法
```python
v_source=pq(url='http://yunvs.com/list/mai_1.html')  
for data in v_source('tr'):
    v_code = pq(data).find('td').eq(0).text() #在已有的data进行二次查找，查找属性td=0的dom。

for data in v_source('li')
	if pq(data).attr('class') == 'class attributes value':
		break
```

问题：Html vs xml
> HTML是超文本标记语言的因为缩写，XML是扩展标记语言的缩写。
他们都是标记语言，是一种特殊的文本标记，用途当然是用于传输数据和现实信息了。
HTML专用于浏览器，而xml则不同，它是信息交换的标准语言，他可以跨平台进行信息的交流。
XML比HTML更加的灵活。


## Scrapy

是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 


