# PHP常见错误收集


##### Fatal error: Non-static method Conn::__construct() cannot be called statically in  /file.php

- 没有静态的方法(里面这个指方法参数，字符串类型),不能从静态上下文引用。

##### Fatal error:  operator not supported for strings in  /file.php

- 当一个变量已设为非数组类型的时候，就不能再次使用[]让同名变量增加数据键值

- 解决方法：

	1.改变变量名称、

	2.使用$var = array(...)

	举例：

```
<?php
//这里为字符串类型 
$err = $e->getMessage();
//当执行到这里的时候会报错 
$err[] = array ( 
	'gid' => $this->_get['id'],
	'url' => $new, 
	'log' => $err, 
	'time' => time() 
);
?>
````

##### Fatal error: Declaration of Listing::content() must be compatible with that of InewsList::content() in file\List_1.phpon line 7

-  统一接口所有类方法都必须和接口规定的一致：作用域声明、方法名、参数数量


##### Parse error: syntax error, unexpected T_NAMESPACE, expecting T_STRING in file\List_1.php on line 42

- 检查语句是否闭合，例如：()、""
- 检查是否有命名冲突，例如：namespace

##### Fatal error: Allowed memory size of 134217728 bytes exhausted (tried to allocate 6144 bytes) in file\List_1.php on line 8

- 内存超过PHP默认设置的128M

##### Notice: Undefined property: News::$_matches in file\List_1.php on line 57

- 没有找到类中的方法

##### Warning: preg_match_all() [function.preg-match-all]: Empty regular expression in file\List_1.php on line 57

-  正则表达式为空

##### Warning: preg_match_all() [function.preg-match-all]: Delimiter must not be alphanumeric or backslash infile\List_1.php on line 57

-  判断为第一个参数的正则表达式写法有问题 记得在前面和后面加上 /  符号。

##### Warning: mysqli::query() [mysqli.query]: Couldn't fetch Insert in /file.php

- 必须使用mysqli链接数据库后返回的结果集去执行操作。

##### Warning: 1064_You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near...

- 网上搜索是MYSQL兼容问题，实际操作上是语法错误，需检查SQL语句写的是否正确。

##### Warning: array_shift() expects parameter 1 to be array, integer given

- 函数第一个参数必须是一个数组。

##### Fatal error: Unsupported operand types in file\List_1.php on line 60

- 致命错误：不支持的操作数据类型
产生原因，将不符合数据类型的数据传送给了某些函数。比如我就不小心将一个数组传给了我的一个自定义函数，而这个函数接受的参数应该是数字。

##### Fatal error: Only variables can be passed by reference in ......

- 在PHP里，如果运行以下代码：

```
<?php
function eee(&$t) { 
	$w = 'hello '.$t; return $w; 
} 
echo eee('World');
?>
```

意思是“只有变量能通过‘引用’”。



##### Warning: Illegal offset type in isset or empty in

- 前几天写程序的时候碰到一个这种错误提示
如果你使用这样的表示方法如下：

```
<?php
$arr = array();
class a {} 
$o = new a;
echo $arr[$o];
?>
```

- 就会出现上面的错误提示，因为不能使用实例化的对象来作为数组的索引，或者在使用isset empty检测这样的数据的时候也会出现第二种情况。
如果在做实际中碰到这样的错误提示，查看数组变量的键名是否使用了实例化的对象变量作为键名

##### #1366 - Incorrect integer value: '' for column 'ID' at row 1

- mysql版本为msyql 5.1.14 WIN32版本,出现错误的原因是没有给自增ID赋值,尽管之前的版本可以不赋值,自动增加,但是在新版本的msyql中需要为其赋值NULL

##### #1136:Column count doesn't match value count at row 1

- 检查一下有没有序号自增加的字段。
- 所存储的数据与数据库表的字段类型定义不相匹配.
- 字段类型是否正确, 是否越界, 有无把一种类型的数据存储到另一种数据类型中.
- SQL语句里列的数目和后面的值的数目不一致

##### #1062_Duplicate entry '...' for key 'map'

- 关键字重复、可能是主键ID、也可能是唯一字段。