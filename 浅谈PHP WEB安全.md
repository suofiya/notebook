## 浅谈php web安全

### 前言：
	
首先，笔记不是web安全的专家，所以这不是web安全方面专家级文章，而是学习笔记、细心总结文章，里面有些是我们phper不易发现或者说不重视的东西。所以笔者写下来方便以后查阅。在大公司肯定有专门的web安全测试员，安全方面不是phper考虑的范围。但是作为一个phper对于安全知识是：“** 知道有这么一回事，编程时自然有所注意 **”。


#### --------------------概要---------------------

1、php一些安全配置
	
* 关闭php提示错误功能
* 关闭一些“坏功能”
* 严格配置文件权限。

2、严格的数据验证，你的用户不全是“好”人

* 为了确保程序的安全性，健壮性，数据验证应该包括内容。
* 程序员容易漏掉point或者说需要注意的事项

3、防注入
 
* 简单判断是否有注入漏洞以及原理
* 常见的mysql注入语句
  * (1)不用用户名和密码
  * (2)在不输入密码的情况下，利用某用户
  * (3)猜解某用户密码
  * (4)插入数据时提权
  * (5)更新提权和插入提权同理
  * (6)恶意更新和删除
  * (7)union、join等
  * (8)通配符号%、_
  * (9)还有很多猜测表信息的注入sql

* 防注入的一些方法
  * php可用于防注入的一些函数和注意事项。
  * 防注入字符优先级。

* 防注入代码
   * (1)参数是数字直接用intval()函数
   * (2)对于非文本参数的过滤
   * (3)文本数据防注入代码。
   * (4)当然还有其他与addslashes、mysql_escape_string结合的代码。

4、防止xss攻击

* Xss攻击过程
* 常见xss攻击地方
* 防XSS方法

5、CSRF

* 简单说明CSRF原理
* 防范方法

6、防盗链

7、防拒CC攻击


#### ---------------------正文开始---------------------


##### 1、php一些安全配置

###### (1)关闭php提示错误功能

在php.ini 中把display_errors改成

	display_errors = OFF

或在php文件前加入

	error_reporting(0)

1) 使用error_reporting(0);失败的例子：
A文件代码：

	<?
	error_reporting(0);
	echo 555
	echo 444;
	?>

错误：
Parse error: parse error, expecting ',' or ';' in E:\webphp\2.php on line 4

2) 使用error_reporting(0);成功的例子：
a文件代码：

	<?php
	error_reporting(0);
	include("b.php");
	?>

b文件代码：

	<?php
	echo 555
	echo 444;
	?>


这是很多phper说用error_reporting(0)不起作用。第一个例子A.php里面有致命错误，导致不能执行，不能执行服务器则不知有这个功能，所以一样报错。

第二个例子中a.php成功执行，那么服务器知道有抑制错误功能，所以就算b.php有错误也抑制了。

ps：抑制不了mysql错误。

######  (2)关闭一些“坏功能”

1)关闭magic quotes功能

	在php.ini 把magic_quotes_gpc = OFF 
    避免和addslashes等重复转义

2)关闭register_globals = Off

	在php.ini 把register_globals = OFF
	在register_globals = ON的情况下

地址栏目：http:www.phpben.com?bloger=benwin

	<?php
	//$bloger = $_GET['bloger']   //因为register_globals = ON 所以这步不用了直接可以用$bloger
	  echo $bloger;
	?>

  这种情况下会导致一些未初始化的变量很容易被修改，这也许是致命的。
  
  所以把register_globals = OFF关掉
  
###### (3)严格配置文件权限。
   
   为相应文件夹分配权限，比如包含上传图片的文件不能有执行权限，只能读取

##### 2、严格的数据验证，你的用户不全是“好”人。
记得笔者和一个朋友在讨论数据验证的时候，他说了一句话：你不要把你用户个个都想得那么坏！但笔者想说的这个问题不该出现在我们开发情景中，我们要做的是严格验证控制数据流，哪怕10000万用户中有一个是坏用户也足以致命，再说好的用户也有时在数据input框无意输入中文的时，他已经不经意变“坏”了。

###### 2.1为了确保程序的安全性，健壮性，数据验证应该包括

* (1)     关键数据是否存在。如删除数据id是否存在
* (2)     数据类型是否正确。如删除数据id是否是整数
* (3)     数据长度。如字段是char(10)类型则要strlen判断数据长度
* (4)     数据是否有危险字符

数据验证有些人主张是把功能完成后再慢慢去写安全验证，也有些是边开发边写验证。笔者偏向后者，这两种笔者都试过，然后发现后者写的验证相对健壮些，主要原因是刚开发时想到的安全问题比较齐全，等开发完功能再写时有两个问题，一个phper急于完成指标草草完事，二是确实漏掉某些point。

###### 2.2程序员容易漏掉point或者说需要注意的事项：

* (1)     进库数据一定要安全验证，笔者在广州某家公司参与一个公司内部系统开发的时候，见过直接把$_POST数据传给类函数classFunctionName($_POST)，理由竟然是公司内部使用的，不用那么严格。暂且不说逻辑操作与数据操控耦合高低问题，连判断都没判断的操作是致命的。安全验证必须，没任何理由推脱。
* (2)     数据长度问题，如数据库建表字段char(25)，大多phper考虑到是否为空、数据类型是否正确，却忽略字符长度，忽略还好更多是懒于再去判断长度。（这个更多出现在新手当中，笔者曾经也有这样的思想）
* (3)     以为前端用js判断验证过了，后台不需要判断验证。这也是致命，要知道伪造一个表单就几分钟的事，js判断只是为了减少用户提交次数从而提高用户体验、减少http请求减少服务器压力，在安全情况下不能防“小人”，当然如果合法用户在js验证控制下是完美的，但作为phper我们不能只有js验证而抛弃再一次安全验证。
* (4)     缺少对表单某些属性比如select、checkbox、radio、button等的验证，这些属性在web页面上开发者已经设置定其值和值域（白名单值），这些属性值在js验证方面一般不会验证，因为合法用户只有选择权没修改权，然后phper就在后端接受数据处理验证数据的时候不会验证这些数据，这是一个惯性思维，安全问题也就有了，小人一个伪表单足矣致命。
* (5)     表单相应元素name和数据表的字段名一致，如用户表用户名的字段是user_name，然后表单中的用户名输入框也是user_name ，这和暴库没什么区别。
* (6)     过滤危险字符方面如防注入下面会独立讲解。


##### 3、防注入

###### 3.1简单判断是否有注入漏洞以及原理。
网址：http:www.phpben.com/benwin.php?id=1 运行正常，sql语句如：select  *  from phpben where id = 1

	(1) 网址：http:www.phpben.com/benwin.php?id=1’   sql语句如：select  *  from phpben where id = 1’  然后运行异常 这能说明benwin.php文件没有对id的值进行“’” 过滤和intval()整形转换，当然想知道有没有对其他字符如“%”，“/*”等都可以用类似的方法穷举测试（很多测试软件使用）

	(2)网址：http:www.phpben.com/benwin.php?id=1 and 1=1  则sql语句可能是 select  *  from phpben where id = 1 and 1=1，运行正常且结果和http:www.phpben.com/benwin.php?id=1结果一样，则说明benwin.php可能没有对空格“ ”、和“and”过滤（这里是可能，所以要看下一点）

	(3)网址：http:www.phpben.com/benwin.php?id=1 and 1=2则sql语句可能是 select  *  from phpben where id = 1 and 1=2 如果运行结果异常说明sql语句中“and 1=2”起作用，所以能3个条件都满足都则很确定的benwin.php存在注入漏洞。

ps：这里用get方法验证，post也可以，只要把值按上面的输入，可以一一验证。
这说明

###### 3.2常见的mysql注入语句。

* (1)不用用户名和密码

```
	//正常语句
	$sql ="select * from phpben where user_name='admin' and pwd ='123'";
	//在用户名框输入’or’=’or’或 ’or 1=’1 然后sql如下
	$sql ="select * from phpben where user_name=' 'or'='or'' and pwd ='' ";
	$sql ="select * from phpben where user_name=' 'or 1='1' and pwd ='' ";
```

这样不用输入密码。话说笔者见到登录框都有尝试的冲动。

* (2)在不输入密码的情况下，利用某用户。

```
	//正常语句
	$sql ="select * from phpben where user_name='$username' and pwd ='$pwd'";
	//利用的用户名是benwin 则用户名框输入benwin’#  密码有无都可,则$sql变成
	$sql ="select * from phpben where user_name=' benwin'#' and pwd ='$pwd'";
```

这是因为mysql中其中的一个注悉是“#”，上面语句中#已经把后面的内容给注悉掉，所以密码可以不输入或任意输入。网上有些人介绍说用“/*”来注悉，笔者想提的是只有开始注悉没结束注悉“*/”时，mysql会报错，也不是说“/**/”不能注悉，而是这里很难添加上“*/”来结束注悉，还有“-”也是可以注悉mysql 但要注意“--”后至少有一个空格也就是“-	”，当然防注入代码要把三种都考虑进来，值得一提的是很多防注入代码中没把“-	”考虑进防注入范围。

* (3)猜解某用户密码

```
	//正常语句
	$sql ="select * from phpben.com where user_name='$username' and pwd ='$pwd'";
	//在密码输入框中输入“benwin’ and left(pwd,1)='p'#”，则$sql是
	$sql ="select * from phpben.com where user_name=' benwin' and left(pwd,1)='p'#' and pwd ='$pwd'";
```

如果运行正常则密码的密码第一个字符是p，同理猜解剩下字符。

* (4)插入数据时提权

```
	//正常语句，等级为1
	$sql = "insert into phpben.com (`user_name`,`pwd`,`level`) values(‘benwin','iampwd',1) ";
	//通过修改密码字符串把语句变成
	$sql = "insert into phpben.com (`user_name`,`pwd`,`level`) values(‘benwin','iampwd',5)#',1) ";
	$sql = "insert into phpben.com (`user_name`,`pwd`,`level`) values(‘benwin','iampwd',5)-	 ',1) ";这样就把一个权限为1的用户提权到等级5
```

* (5)更新提权和插入提权同理

```
	//正常语句
	$sql = "update phpben set  `user_name` ='benwin', level=1";
	//通过输入用户名值最终得到的$sql
	$sql = "update phpben set  `user_name` ='benwin',level=5#', level=1";
	$sql = "update phpben set  `user_name` ='benwin',level=5-	 ', level=1";
```

* (6)恶意更新和删除

```
	//正常语句
	$sql = "update phpben set `user_name` = ‘benwin' where id =1";
	//注入后,恶意代码是“1 or id>0”
	$sql = "update phpben set `user_name` = ‘benwin' where id =1 or id>0";
	//正常语句
	$sql = "update phpben set  `user_name` =’benwin’ where id=1";
	//注入后
	$sql = "update phpben set  `user_name` ='benwin' where id>0#' where id=1";
	$sql = "update phpben set  `user_name` ='benwin' where id>0-			' where id=1";
```

* (7)union、join等

```
	//正常语句
	$sql ="select * from phpben1 where `user_name`=’benwin’ ";
	//注入后
	$sql ="select * from phpben1 where`user_name`=’benwin’ uninon select * from phpben2#’ ";
	$sql ="select * from phpben1 where`user_name`=’benwin’ left join……#’ ";
```

* (8)通配符号%、_

```
	//正常语句
	$sql ="select * from phpben where `user_name`=’benwin’ ";
	//注入通配符号%匹配多个字符，而一个_匹配一个字符，如__则匹配两个字符
	$sql ="select * from phpben where `user_name` like ’%b’ ";
	$sql ="select * from phpben where `user_name` like ’_b_’ ";
```

这样只要有一个用户名字是b开头的都能正常运行,“ _b_”是匹配三个字符，且这三个字符中间一个字符时b。这也是为什么有关addslashes()函数介绍时提示注意没有转义%和_(其实这个是很多phper不知问什么要过滤%和_下划线，只是一味的跟着网上代码走)

* (9)还有很多猜测表信息的注入sql

```
	//正常语句
	$sql ="select * from phpben1 where`user_name`='benwin'";
	//猜表名，运行正常则说明存在phpben2表
	$sql ="select * from phpben1 where`user_name`='benwin' and (select count(*) from phpben2 )>0#' ";
	//猜表字段，运行正常则说明phpben2表中有字段colum1

	$sql ="select * from phpben1 where`user_name`='benwin' and (select count(colum1) from phpben2 )>0#'";
	//猜字段值
	$sql ="select * from phpben1 where`user_name`='benwin' and left(pwd,1)='p'#’'";
```

当然还有很多，笔者也没研究到专业人士那种水平，这里提出这些都是比较常见的，也是phper应该知道并掌握的，而不是一味的在网上复制粘贴一些防注入代码，知然而不解其然。

下面一些防注入方法回看可能更容易理解。

##### 3.3防注入的一些方法

###### 3.3.1 php可用于防注入的一些函数和注意事项。

* (1)addslashes 和stripslashes。
	Addslashes给这些 “’”、“””、“\”,“NULL” 添加斜杆“\’”、“\””、“\\”,“\NULL”, 
stripslashes则相反，这里要注意的是php.ini是否开启了magic_quotes_gpc=ON，开启若使用addslashes会出现重复。所以使用的时候要先get_magic_quotes_gpc()检查
一般代码类似：

	if(!get_magic_quotes_gpc())
	{
	         $abc = addslashes($abc);
	}

其实这个稍微学习php一下的人都知道了，只不过笔者想系统点介绍(前面都说不是专家级文章)，所以也顺便写上了。addslashes

* (2)mysql_escape_string()和mysql_ real _escape_string()
	mysql_real_escape_string 必须在(PHP 4 >= 4.3.0, PHP 5)的情况下才能使用。否则只能用 mysql_escape_string

```
	if (PHP_VERSION >= '4.3')
	{
	$string  =  mysql_real_escape_string($string);
	}else
	{
	$string  =  mysql_escape_string($string );
	}
```

mysql_escape_string()和mysql_ real _escape_string()却别在于后者会判断当前数据库连接字符集，换句话说在没有连接数据库的前提下会出现类似错误：

Warning: mysql_real_escape_string() [function.mysql-real-escape-string]: Access denied for user 'ODBC'@'localhost' (using password: NO) in E:\webphp\test.php on line 11

* (3)字符代替函数和匹配函数
	str_replace() 、perg_replace()这些函数之所以也在这里提是因为这些函数可以用于过滤或替代一些敏感、致命的字符。

###### 3.3.2防注入字符优先级。

防注入则要先知道有哪些注入字符或关键字，常见的mysql注入字符有字符界定符号如“'”、“"”；逻辑关键字如“and”、“or”；mysql注悉字符如“#”，“-	”，“/**/”；mysql通配符“%”，“_”；mysql关键字“select|insert|update|delete|*|union|join|into|load_file|outfile”

* (1)对于一些有规定格式的参数来说，防注入优先级最高的是空格” ”。

	如一些银行卡号，身份证号，邮箱，电话号码，，生日，邮政编码等这些有自己规定的格式且格式规定不能有空格符号的参数，在过滤的时候一般最先过滤掉空格（包括一些空格“变种”），因为其他字符界定符号，逻辑关键字，mysql注悉，注意下图可以看出重要的是“'”,“ ”

	ps：空格字符的变种有：“%20”，“\n”，“\r”，“\r\n”，“\n\r”，“chr("32")” 这也是为什么mysql_escape_string()和mysql_real_escape_string() 两个函数转义“\n”，“\r”。其实很多phper只知道转义\n,\r而不知原因，在mysql解析\n,\r时把它们当成空格处理，笔者测试验证过，这里就不贴代码了。

* (2)“and”，“or”，“\”，“#”，“-	”
	逻辑关键可以组合很多注入代码；mysql注悉则把固有sql代码后面的字符全部给注悉掉从而让注入后的sql语句能正常运行；“\”也是能组合很多注入字符\x00,\x1a。
	ps:sql解析“#”，“-	”是大多数mysql防注入代码没有考虑到的，也是很多phper忽略。还有因为一些phper给参数赋值的时候会有用“-”来隔开，所以笔者建议不要这样写参数，当然也可以再过滤参数的时候“-	”（注意有空格的，没空格不解析为注悉）当一个整体过滤而不是过滤“-” ，这样就避免过多过滤参数。

* (3)“null”，“%”，“_”
	这几个不能独立，都不要在特定情况下，比如通配字符“%，_”都要在mysql like子句的前提下。所以“%”，“_”的过滤一般在搜索相关才过滤，不能把它们纳入通常过滤队列，因为有些如邮箱就可以有”_”字符

* (4)关键字“select|insert|update|delete|*|union|join|into|load_file|outfile”
	也许你会问怎么这些重要关键字却优先级这么低。笔者想说的是因为这些关键字在没有“'”,“"”，“ ”，“and”，“or”等情况下购不成伤害。换句话说这些关键字不够“独立”，“依赖性”特别大。当然优先级低，不代表不要过滤。

######  3.3.3防注入代码。
* (1)参数是数字直接用intval()函数
	注意：现在很多网上流行的防注入代码都只是只是用addslashes()、mysql_escape_string()、mysql_real_escape_string()或三者任意组合过滤，但phper以为过滤了，一不小心一样有漏洞，那就是在参数为数字的时候：

```
	$id = addslashes($_POST['id']); //正确是$id = intval($_POST['id']);
	$sql =" select * from phpben.com where id =$id";
	$sql =" select * from phpben.com where id =1 or 1=1";
```

对比容易发现，post过来的数据通过addslashes过滤后的确很多注入已经不起作用，但是$id并没有intval，导致漏洞的存在，这是个小细节，不小心则导致漏洞。

* (2)对于非文本参数的过滤

文本参数是指标题、留言、内容等可能有“’”，“’”等内容，过滤时不可能全部转义或代替。
但非文本数据可以。

	function _str_replace($str )
	{
	     $str = str_replace(" ","",$str);
	     $str = str_replace("\n","",$str);
	     $str = str_replace("\r","",$str);
	     $str = str_replace("'","",$str);
	     $str = str_replace('"',"",$str);
	     $str = str_replace("or","",$str);
	     $str = str_replace("and","",$str);
	     $str = str_replace("#","",$str);
	     $str = str_replace("\\","",$str);
	     $str = str_replace("-	","",$str);
	     $str = str_replace("null","",$str);
	     $str = str_replace("%","",$str);
	     //$str = str_replace("_","",$str);
	     $str = str_replace(">","",$str);
	     $str = str_replace("<","",$str);
	     $str = str_replace("=","",$str);
	     $str = str_replace("char","",$str);
	     $str = str_replace("declare","",$str);
	     $str = str_replace("select","",$str);
	     $str = str_replace("create","",$str);
	     $str = str_replace("delete","",$str);
	     $str = str_replace("insert","",$str);
	     $str = str_replace("execute","",$str);
	     $str = str_replace("update","",$str);
	     $str = str_replace("count","",$str);
	     return $str;
	}


ps：还有一些从列表页操作过来的一般href是”phpben.php?action=delete&id=1”,这时候就注意啦，_str_replace($_GET['action'])会把参数过滤掉，笔者一般不用敏感关键作为参数，比如delete会写成del，update写成edite，只要不影响可读性即可;
还有上面代码过滤下划线的笔者注悉掉了，因为有些参数可以使用下划线，自己权衡怎么过滤;
有些代码把关键字当重点过滤对象，其实关键字的str_replace很容易“蒙过关”，str_replace(“ininsertsert”)过滤后的字符还是insert，所以关键的是其他字符而不是mysql关键字。

* (3)文本数据防注入代码。
	文本参数是指标题、留言、内容等这些数据不可能也用str_replace()过滤掉，这样就导致数据的完整性，这是很不可取的。
代码：

```
	function no_inject($str)
	{
	         if(is_array($str))
	         {
	                   foreach($str as $key =>$val)
	                   {
	                           $str[$key]=no_inject($val);
	                   }
	         }else
	         {
	        //把一些敏感关键字的第一个字母代替掉，如or  则用"&#111;r"代替
	                   $str = str_replace(" ","&#32;",$str);
	                   $str = str_replace("\\","&#92;",$str);
	                   $str = str_replace("'"," &#39; ",$str);
	                   $str = str_replace('"'," &#34; ",$str);
	                   $str = str_replace("or"," &#111; r",$str);
	                   $str = str_replace("and"," &#97;nd",$str);
	                   $str = str_replace("#","&#35; ",$str);
	                   $str = str_replace("-	","&#45;	",$str);
	                   $str = str_replace("null","&#110;ull",$str);
	                   $str = str_replace("%","&#37;",$str);
	                   //$str = str_replace("_","&#95;",$str);
	                   $str = str_replace(">","&#888;",$str);
	                   $str = str_replace("<","&#62;",$str);
	                   $str = str_replace("=","&#61;",$str);
	                   $str = str_replace("char","&#99;har",$str);
	                   $str = str_replace("declare","&#100;eclare",$str);
	                   $str = str_replace("select","&#115;elect",$str);
	                  $str = str_replace("create","&#99;reate",$str);
	                  $str = str_replace("delete","&#100;elete",$str);
	                  $str = str_replace("insert","&#105;nsert",$str);
	                 $str = str_replace("execute","&#101;xecute",$str);
	                 $str = str_replace("update","&#117;pdate",$str);
	                 $str = str_replace("count","&#99;ount",$str);
	         }
	    return $str;
	}
```


* (4)当然还有其他与addslashes、mysql_escape_string结合的代码。
	防注入的代码其实来来去去都是那些组合，然后根据自己程序代码变通，笔者这些代码也是没考虑全的，不如cookes、session、request都没全过滤。重要是知道其中原理，为什么过滤这些字符，字符有什么危害。当然还有一些笔者没考虑也没能力考虑到的方面比如还有哪些关键字之类，欢迎mailto：chen_bin_wen@163.com/445235728@qq.com

######  4、防止xss攻击

XSS：cross site script 跨站脚本，为什么不叫css，为了不和div+css混淆。

* 4.1Xss攻击过程：

	(1)发现A站有xss漏洞。
	
	(2)注入xss漏洞代码。可以js代码，木马，脚本文件等等，这里假如A站的benwin.php这个文件有漏洞。
	
	(3)通过一些方法欺骗A站相关人员运行benwin.php，其中利用相关人员一些会员信息如cookies，权限等。

	相关人员：
	管理员（如贴吧版主），管理员一般有一定权限。目的是借用管理员的权限或进行提权，添或加管理员，或添加后门，或上传木马，或进一步渗透等相关操作。
	A站会员：会员运行A站的benwin.php。目的一般是偷取会员在A站的信息资料。
	方法：

		* 1)       在A站发诱骗相关人到benwin.php的信息，比如网址，这种是本地诱骗
		* 2)       在其他网站发诱骗信息或者发邮件等等信息。

	一般通过伪装网址骗取A站相关人员点击进benwin.php

	(4)第三步一般已经是一次xss攻击，如果要更进一步攻击，那不断重复执行(2)、(3)步以达到目的。


简单例说xss攻击
代码：
benwin.php文件

	<html>
	<head>
	<title>简单xss攻击例子</title></head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	<dody>
	<form action="phpben.com?user_name=<?php echo $user_name; ?>">
	<input type="submit" value="提交" >
	</form>
	</body>
	</html>

当用户名$user_name的值是“benwin" onSubmit="alert('这是xss攻击的例子');" class= "”（这里）

	<form action="phpben.com?user_name=benwin" onSubmit="alert('这是xss攻击的例子');" class= "" >
	<input type="submit" value="提交" >
	</form>

当提交表单的时候就会弹出提示框。

	* (1)     很明显$user_name在保存进数据库的时候没有过滤xss字符（和防注入很像，这里举例说明）==>发现漏洞
	* (2)     构造xss代码：benwin" onSubmit="alert('这是xss攻击的例子');" class= "" 传入数据库
	* (3)     骗相关人员进来点击“提交”按钮


###### 4.2常见xss攻击地方

* (1)Js地方

```
	<script language="javascript">
	var testname =" <?php echo $testname;?>";
	</script>
```

$testname的值只要符合js闭合关系：“";alert("test xss ");”(以下同理)

* (2)form表单里面

```
	<input type="text" name="##" value="<?php echo $val; ?>" />
```

* (3)a标签

```
	<a href="benwin.php?id= <?php echo $id; ?>">a标签可以隐藏xss攻击</a>
```

* (4)用得很多的img标签

```
	<img src="<?php echo $picPath; ?>" />
```

甚至一些文本中插入整个img标签并且用width、 height、css等隐藏的很隐蔽

* (5)地址栏目

总之，有输出数据的地方，更准确的说是有输出用户提交的数据的地方，都有可能是XSS攻击的地方。

###### 4.3防XSS方法

防xss方法其实和防注入很相似，都是一些过滤、代替、实体化等方法

* (1)过滤或移除特殊的Html标签。
 例如:< 、>、&lt;,、&gt; ’、”、<script>、 <iframe> 、&lt;,、&gt;、&quot

* (2)过滤触发JavaScript 事件的标签。例如 onload、onclick、onfocus、onblur、onmouseover等等。
* (3)php一些相关函数，strip_tags()、htmlspecialchars()、htmlentities()等函数可以起作用

##### 5、CSRF

CSRF跨站请求伪造cross site request forgery。

5.1简单说明CSRF原理

* (1)A登录Site1（如现在网民常上的淘宝、微博、QQ等），产生一些信息，session、cookies等等，且一直保持没退出。
* (2)A再登录Site2（如一些成人网等，至于怎么跑到Site2，多数是Site通过些手段，邮件欺骗等），打开site2的浏览器和打开site1的一样，否则无效
* (3)Site2站中伪造了Site1的http请求（如修改密码，买东西，转账等），Site1的服务器误以为A在site1的正常操作（因为同浏览器且A还没登出），然后就运行了请求，那么csrf已成功操作。

csrf和xss很相似。
笔者开始也一时混淆，一位热心网友提醒下，修改csrf和xss的区别

xss：注重于修改、获取页面元素和值，而且可以执行任意html代码或javascript，能执行代码那肯定能偷取cookies等网站信息

csrf：不能修改、获取页面元素和值，而且可以执行任意html代码或javascript，注重利用客户已在安全网站（比如淘宝）登录且产生给非安全网站利用的cookies等信息，然后用户在非安全网站点击连接等一些触发非安全网站向安全网站发送请求。

伪造的请求可以很多方面，发邮件、改密码、返回用户信息、交易等等，所以相对与xss攻击来说csrf危害更严重。

###### 5.2防范方法。

对于phper

* (1)严密操控执行入口
执行一些敏感操作比如改密码这些操作前判断请求来源，只有本站服务器发的请求才可以执行。判断方法可以判断ip来源。非本站服务器ip不会执行。

* (2)本站有外链的话做些必要操作
一般site2的hacker会在site1（比如论坛里）里发欺骗连接，因为在site1诱骗的相关人员一般都登录site1了，满足csrf气体条件之一。

如当你点击QQ邮件里面的长外链时候，回跳转到一个页面提示“有风险”之类，这样不仅可以减低跳出率，一些不懂的人看到这样的提示，若不是非必要而是处于好奇点击的连接一般不会继续点击访问；还有是QQ邮件正文里的图片在加载内容时是不加载图片的，要点击“显示图片”按钮才显示图片，这里一个原因之一就是避免攻击。

当然对于用户体验来说这是不可取的，可以优化的是判断到一些网址（如QQ本身网址）是安全直接可以显示（不用提示），而可疑的才提示或禁止。

* (3)防止csrf也可以用防xss的方法。

##### 6、防盗链
盗链问题增加服务器的负担。盗链就是盗链网站盗取被盗链网站资源来实现一些功能。盗链方面主要是图片、视频、以及其他资源下载文件。

方法：判断ip，只有本站服务器才能使用站点资源，否则不能使用。

代码：
(1)在Apache htaccess添加

	RewriteEngine on
	RewriteCond %{HTTP_REFERER} !^$ [NC]
	RewriteCond %{HTTP_REFERER} !phpben.com [NC]
	RewriteCond %{HTTP_REFERER} !google.com [NC]
	RewriteCond %{HTTP_REFERER} !baidu.com [NC]
	RewriteCond %{HTTP_REFERER} !zhuaxia.com [NC]
	RewriteRule .(jpg|gif|png|bmp|swf|jpeg) /image/replace.gif [R,NC,L]
	RewriteRule ^(.*)$ http:\/\/phpben.com\/image\/$1 [L]

这样，凡是不是phpben.com google.com baidu.com zhuaxia.com 域名请求的都返回replace.gif代替返回

##### 7、防CC攻击

CC攻击：是利用不断对网站发送连接请求致使形成拒绝服务的目的。

详细百度百科：http://baike.baidu.com/view/662394.htm

代码：

	session_start();
	$ll_nowtime = $timestamp ;
	if (session_is_registered('ll_lasttime')){
	$ll_lasttime = $_SESSION['ll_lasttime'];
	$ll_times = $_SESSION['ll_times'] + 1;
	$_SESSION['ll_times'] = $ll_times;
	}else{
	$ll_lasttime = $ll_nowtime;
	$ll_times = 1;
	$_SESSION['ll_times'] = $ll_times;
	$_SESSION['ll_lasttime'] = $ll_lasttime;
	}
	if (($ll_nowtime 	$ll_lasttime)<3){
	if ($ll_times>=5){
	header(sprintf("Location: %s",'http://127.0.0.1'));
	exit;
	}
	}else{
	$ll_times = 0;
	$_SESSION['ll_lasttime'] = $ll_nowtime;
	$_SESSION['ll_times'] = $ll_times;
	}


