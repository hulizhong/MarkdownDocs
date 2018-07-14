[TOC]

## ReadMe
> http://www.runoob.com/php/php-tutorial.html
> http://php.net/manual/zh/language.oop5.properties.php

## 一个demo引进门
四种php的标记风格
```php
<?php
	echo "这是xml风格的标记";
?>

<script languange="php">
	echo "这是脚本风格的标记";
</script>

<?这是简短风格的标记;?>
#需要在php.ini中配置short_open_tag=on或者php编译时加入--enable-short-tags选项；

<%
	echo "这是asp风格的标记";
%>
#需要在php.ini中进行配置asp_tags = on;
```


demo如下
```php
<?php
// PHP 代码
// end with ;


//变量
$var1 = "dafaf" //变量名前加个$。（有点怪，按说定义的时候不用加的，而只是用的时候才加）


//输出语句
echo $var1 . " connect " . $var2 . "<br>" //echo可以接多个参数
print "$var1 <br>"  //不能接多个参数


//函数
function writeName()
{
    echo "Kai Jim Refsnes";
}
writeName();


//类
class SimpleClass
{
    // property declaration
    public $var = 'a default value';

    // method declaration
    public function displayVar() {
        echo $this->var;
    }
}
$instance = new SimpleClass();  //new一定要；
$instance->displayVar(); 

?>
```


## 变量
### 数组
- 数值数组，以数字为下标
	- 遍历时：for, foreach均可
- 关联数组，
	- 遍历只有foreach方法（不以数字为下标）；
	
		```php
		foreach ($colors as $key => $color) {
	    	$colors[$key] = strtoupper($color);
		}
		```


## 库
### jpgraph

