# php autoload 机制探索


autoload机制是php的一个容错机制,提供了一种在没有预先加载相关类的情况下,不让程序出错,继续执行的机制.



下面我们分别分几方面来探索php的autoload机制


1. 最基本的__autoload方法

2. 普遍的__autoload的实现逻辑

3. 如何自定义自己的autoload方法

4. 多个项目中的autoload方法是如何协调工作的



##### 1. 最基本的__autoload方法


以下文字摘自php官方的说明文档:

	很多开发者写面向对象的应用程序时对每个类的定义建立一个 PHP 源文件。一个很大的烦恼是不得不在每个脚本开头写一个长长的包含文件列表（每个类一个文件）。

	在 PHP 5 中，不再需要这样了。可以定义一个 __autoload() 函数，它会在试图使用尚未被定义的类时自动调用。通过调用此函数，脚本引擎在 PHP 出错失败前有了最后一个机会加载所需的类。



a.php是主访问程序


```
<?php
$a = new testAuto();
     
function __autoload($class)
{
    require('b.php');
}
```


b.php是 testAuto类的定义文件, 构造方法会输出 this is from b.php

```
<?php
class testAuto {
    function __construct (){
        echo "this is from b.php";
    }
}
```



访问a.php会有如下输出



this is from b.php


##### 2. 普遍的__autoload的实现逻辑

在实现__autoload的时候,绝大多数的实现方式都是根据类名去找对应的文件,这个大家可以参考各种开源软件的实现,估计90%都是这种模式



项目的所有类都是按照   ClassName.class.php来命名的

那么__autoload的实现可以如下


```
<?php   
function __autoload($class)
{
    require(CLASS_PATH.'/'.$class.'.class.php');
}
```


##### 3. 如何自定义自己的autoload方法


       在php官方文档中也提到了,spl_autoload_register() 提供了一种更加灵活的方式来实现类的自动加载。因此，不再建议使用 __autoload()函数，在以后的版本中它可能被弃用。



为了演示自定义autoload方法,我们新添加如下两个文件,与b.php相似,只是输出的内容有变化




c.php是 testAuto类的定义文件, 构造方法会输出 this is from c.php


```
<?php
class testAuto {
    function __construct (){
        echo "this is from c.php";
    }
}
```



d.php是 testAuto类的定义文件, 构造方法会输出 this is from d.php


```
<?php
class testAuto {
    function __construct (){
        echo "this is from d.php";
    }
}
```



主访问程序 a.php的内容变为如下所示:

```
<?php
spl_autoload_register('myautoload');
$a = new testAuto();
   
function myautoload($class)
{
    echo "myautoload"."<hr>";
    include('c.php');
}
   
function __autoload($class)
{
    require('b.php');
}
```

输出:

myautoload

this is from c.php

这样我们就通过spl_autoload_register成功的将autoload的方法变为了 myautoload



##### 4. 项目中多个autoload方法是如何工作的
如下情况是大家经常遇见的:

项目中引用了多个第三方的库,比如 smarty等等, 如果你仔细阅读他们的源码,就会发现,其实每个类库里都有他自己的autoload方法,那么这些第三方库的autoload会对我们项目本身的autoload会有什么影响呢?



下面先来做一个最简单的例子, 在a.php里我们多次调用spl_autoload_register看看,代码变为如下所示:


```
<?php
spl_autoload_register('myautoload');
spl_autoload_register('myautoload1');
$a = new testAuto();
   
function myautoload($class)
{
    echo "myautoload"."<hr>";
    include('c.php');
}
function myautoload1($class)
{
    echo "myautoload1"."<hr>";
    include('d.php');
}
   
function __autoload($class)
{
    require('b.php');
}
```

输出内容会变成如下所示:

myautoload

this is from c.php



因此,**最先spl_autoload_register的方法最先调用 ( 先注册先调用)**



那么问题又来了 **如果最先调用的方法没有正确加载到相应的类,那么会怎么样呢?**



将a.php代码改为如下所示:

```
<?php
spl_autoload_register('myautoload');
spl_autoload_register('myautoload1');
   
$a = new testAuto();
   
function myautoload($class)
{
    echo "myautoload"."<hr>";
    include('1.php'); // 此文件是一个不存在的文件
}
function myautoload1($class)
{
    echo "myautoload1"."<hr>";
    include('d.php');
}
   
function __autoload($class)
{
    require('b.php');
}
```

页面输出如下:

myautoload

myautoload1

this is from d.php



所以,**多次spl_autoload_register会按照注册顺序来依次调用方法,找到则退出 ( 先注册先调用)**





O(∩_∩)O~,如果你再深入想一下,**如果所有spl_autoload_register的方法,都找不到该类,会发生什么?  会调用到默认的__autoload方法吗?**

再将a.php改为如下所示

```
<?php
spl_autoload_register('myautoload');
spl_autoload_register('myautoload1');
//spl_autoload_register('__autoload');
  
$a = new testAuto();
  
function myautoload($class)
{
    echo "myautoload"."<hr>";
    include('1.php');  // 此文件不存在
}
function myautoload1($class)
{
    echo "myautoload1"."<hr>";
    include('2.php'); // 此文件不存在
}
  
function __autoload($class)
{
    require('b.php');
}
```

这时候输出:

myautoload

myautoload1



php会报错,找不到对应的类



PHP Fatal error:  Class 'testAuto' not found



结论就是: **调用spl_autoload_register之后,默认的__autoload就会失效,**如果想继续回朔__autoload方法,需要在代码最开始加入如下代码

spl_autoload_register('__autoload');