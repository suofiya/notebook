# PHP面试问题

## 初级

###### 1、PHP的整型溢出问题是怎样的

  PHP的整型数的字长和平台有关，对于32位的操作系统，最大的整型是有二十多亿，其实就是2的31次方，最小为-2的31次方，PHP不支持无符号的整数。 如果一个数超出了integer范围，将会被自动解释为float。如果执行的运算结果超出了 integer 范围，也会返回 float。（那在java、C中的整型整型溢出会怎样）
  
###### 2、如何理解OOP

  OOP，面向对象编程，包括三个方面，继承性、封装性、多态性，其中最根本的东西就是抽象。
  继承性，即扩展性，通过子类对已经存在的父类进行功能扩展。
  封装性，要求外部不能随意存取对象的内部数据，即对该类中的具体实现做封装，用户不必知道内部的具体实现，只有知道它是干什么的，怎么用就好了。
  多态性，就是类的抽象和接口，同一个类能够处理多种类型对象的能力。

###### 3、你对于设计模式和MVC的理解

Model-View-Controller，模型、视图、控制器，一想到MVC就会想到JAVA，因为JAVA是一个完全面向对象的语言，MVC最早出现在smalltalk中，其核心就是要将试图和数据模型分离，这样不同的程序就可以有不同的展示。

模型，即程序员写的功能、算法和数据模型，也就是我们说的系统业务逻辑层。
试图，即前端，图形界面。展示给用户看的。
控制器，主要负责对请求处理和转发。

设计模式，其实就是代码的设计经验的总结和归类，设计模式最早应用与建筑行业，编程的设计模式按最早的GoF所述，包括23种设计模式，主要用于面向对象的程序编程。遵循几个设计原则：开闭原则、单一职责原则、里氏替换原则、依赖注入、接口分离、迪米特原则、优先使用组合而不是继承等等。包括创建型模式、结构性模式、行为模式三类。

###### 4、MySQL的索引机制，复合索引的使用原则

（深入浅出MySQL一书中对索引的使用讲的比较细致）
一般都会用书本中的目录来介绍索引机制，其实有些书本会有专门的快速检索附录，就很类似于数据库的索引。
MySQL的索引包括4类：主键索引(primary key)、唯一索引(unique)、常规索引(index)、全文索引(fullindex)。 Show index from table_name; --查看表中的索引
Show status like 'Handler_read%'  --查看索引的使用情况

复合索引，一般遵循最左前缀原则，如table_a 的 a b c 三列建复合索引
create index ind_table_a on table_a(a,b,c);
那么，只有在条件中用到a,或者a、b,或者a、b、c这样的情况下，才会用到刚建的复合索引。

###### 5、MySQL的表类型及MyISAM与InnoDB的区别

MySQL常见的表类型(即存储引擎)show engines包 括:MyISAM/Innodb/Memory/Merge/NDB

其中，MyISAM和Innodb是最常用的两个表类型，各有优势，我们可以根据需求情况选择适合自己的表类型。
[MyISAM]
1）每个数据库存储包括3个文件：.frm(表定义)、MYD(数据文件)、MYI(索引文件)
2）数据文件或索引文件可以指向多个磁盘
3）Linux的默认引擎，win默认InnoDB
4）面向非事务类型，避免事务型额外的开销
5）适用于select、insert密集的表
6）MyISAM默认锁的调度机制是写优先，可以通过LOW_PRIORITY_UPDATES设置
7）MyISAM类型的数据文件可以在不同操作系统中COPY，这点很重要，布署的时候方便点。

[Innodb]
1）用于事务应用程序
2）适用于update、delete密集的操作。执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。
3）引入行级锁和外键的约束
4）InnoDB不支持FULLTEXT类型的索引

###### 6、简单说下快速排序算法


基本思想：通过一趟排序将待排序列分割成两部分，其中一部分比另一部分记录小，再分别对这两部分继续快速排序，以达到有序。
算法实现：设有两个指针low和high，初值为low=1，high=n，设基准值为key(通常选第一个)，则首先从high位置开始向前搜索，找到第一个比key小的记录与key交换，然后从low位置向后搜索，找到第一个比key大的记录与基准值交换，重复直至low=high为止。

第一趟排序结果，key之前的记录值比key之后的记录值小。

11 25 9  3  16 2   //选择11为key
2  25 9  3  16 11
2  11 9  3  16 25
2  3  9  11 16 25


###### 7. 缓存

1、全页面静态化缓存

也就是将页面全部生成html静态页面，用户访问时直接访问的静态页面，而不会去走php服务器解析的流程。此种方式，在CMS系统中比较常见，比如dedecms；

一种比较常用的实现方式是用输出缓存：

```
Ob_start()
******要运行的代码*******
$content = Ob_get_contents();
****将缓存内容写入html文件*****
Ob_end_clean();
```

2、页面部分缓存

该种方式，是将一个页面中不经常变的部分进行静态缓存，而经常变化的块不缓存，最后组装在一起显示；可以使用类似于ob_get_contents的方式实现，也可以利用类似ESI之类的页面片段缓存策略，使其用来做动态页面中相对静态的片段部分的缓存(ESI技术，请baidu，此处不详讲)。

该种方式可以用于如商城中的商品页；

3、数据缓存

顾名思义，就是缓存数据的一种方式；比如，商城中的某个商品信息，当用商品id去请求时，就会得出包括店铺信息、商品信息等数据，此时就可以将这些数据缓存到一个php文件中，文件名包含商品id来建一个唯一标示；下一次有人想查看这个商品时，首先就直接调这个文件里面的信息，而不用再去数据库查询；其实缓存文件中缓存的就是一个php数组之类；

Ecmall商城系统里面就用了这种方式；

4、查询缓存

其实这跟数据缓存是一个思路，就是根据查询语句来缓存；将查询得到的数据缓存在一个文件中，下次遇到相同的查询时，就直接先从这个文件里面调数据，不会再去查数据库；但此处的缓存文件名可能就需要以查询语句为基点来建立唯一标示；

按时间变更进行缓存

其实，这一条不是真正的缓存方式；上面的2、3、4的缓存技术一般都用到了时间变更判断；就是对于缓存文件您需要设一个有效时间，在这个有效时间内，相同的访问才会先取缓存文件的内容，但是超过设定的缓存时间，就需要重新从数据库中获取数据，并生产最新的缓存文件；比如，我将我们商城的首页就是设置2个小时更新一次；

5、按内容变更进行缓存

这个也并非独立的缓存技术，需结合着用；就是当数据库内容被修改时，即刻更新缓存文件；
比如，一个人流量很大的商城，商品很多，商品表必然比较大，这表的压力也比较重；我们就可以对商品显示页进行页面缓存；

当商家在后台修改这个商品的信息时，点击保存，我们同时就更新缓存文件；那么，买家访问这个商品信息时，实际上访问的是一个静态页面，而不需要再去访问数据库；

试想，如果对商品页不缓存，那么每次访问一个商品就要去数据库查一次，如果有10万人在线浏览商品，那服务器压力就大了；

6、内存式缓存

提到这个，可能大家想到的首先就是Memcached；memcached是高性能的分布式内存缓存服务器。 一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、 提高可扩展性。

它就是将需要缓存的信息，缓存到系统内存中，需要获取信息时，直接到内存中取；比较常用的方式就是 key–>value方式；

```
<?php 
     $memcachehost = '192.168.6.191';
     $memcacheport = 11211;
     $memcachelife = 60;
     $memcache = new Memcache;
     $memcache->connect($memcachehost,$memcacheport) or die ("Could not connect");
     $memcache->set('key','缓存的内容');
     $get = $memcache->get($key);       //获取信息
?>
```

7、apache缓存模块

apache安装完以后，是不允许被cache的。如果外接了cache或squid服务器要求进行web加速的话，就需要在htttpd.conf里进行设置，当然前提是在安装apache的时候要激活mod_cache的模块。

安装apache时：./configure –enable-cache –enable-disk-cache –enable-mem-cache

8、php APC缓存扩展

Php有一个APC缓存扩展，windows下面为php_apc.dll，需要先加载这个模块，然后是在php.ini里面进行配置：


```
[apc] 
     extension=php_apc.dll 
     apc.rfc1867 = on 
     upload_max_filesize = 100M 
     post_max_size = 100M 
     apc.max_file_size = 200M 
     upload_max_filesize = 1000M 
     post_max_size = 1000M 
     max_execution_time = 600 ;   每个PHP页面运行的最大时间值(秒)，默认30秒 
     max_input_time = 600 ;       每个PHP页面接收数据所需的最大时间，默认60 
     memory_limit = 128M ;       每个PHP页面所吃掉的最大内存，默认8M
```     
     
     
9、Opcode缓存

我们知道，php的执行流程可以用下图来展示：

![image](http://www.php100.com/uploadfile/2015/0919/20150919024343438.jpg)

首先php代码被解析为Tokens，然后再编译为Opcode码，最后执行Opcode码，返回结果；所以，对于相同的php文件，第一次运行时可以缓存其Opcode码，下次再执行这个页面时，直接会去找到缓存下的opcode码，直接执行最后一步，而不再需要中间的步骤了。

比较知名的是XCache、Turck MM Cache、PHP Accelerator等。


## 高级

###### 1. 网站运行一段时间后，随着访问量的增加，系统会越来越慢，如何通过优化，提高网站的性能？如果数据库已经达到最优化，请设计出持续升级的方案。
类似问题：
	
	如何提高网页的加载速度？
	对于大流量网站，你将从哪几个方面解决访问量的问题？
	
###### 2. SESSION与COOKIE的区别和关系？禁用cookie后，session能否继续使用？如何设置一个严格30分钟过期的Session？

cookie在客户端保存状态，session在服务器端保存状态。但是由于在服务器端保存状态的时候，在客户端也需要一个标识，所以session也可能要借助cookie来实现保存标识位的作用。
cookie包括名字，值，域，路径，过期时间。路径和域构成cookie的作用范围。cookie如果不设置过期时间，则这个cookie在浏览器进程存在时有效，关闭时销毁。如果设置了过期时间，则cookie存储在本地硬盘上，在各浏览器进程间可以共享。
session存储在服务器端，服务器用一种散列表类型的结构存储信息。当一个连接建立的时候，服务器首先搜索有没有存储的session id，如果没有，则建立一个新的session，将session id返回给客户端，客户端可以选择使用cookie来存储session id。也可以用其他的方法，比如服务器端将session id附在URL上。
区别：

(1).cookie在本地，session在服务器端。
(2).cookie不安全，容易被欺骗，session相对安全。
(3).session在服务器端，访问多了会影响服务器性能。
(4). cookie有大小限制，为3K
多服务器共享session可以尝试将session存储在memcache中。

第一种回答

那么, 最常见的一种回答是: 设置Session的过期时间, 也就是session.gc_maxlifetime, 这种回答是不正确的, 原因如下:

1. 首先, 这个PHP是用一定的概率来运行session的gc的, 也就是session.gc_probability和session.gc_divisor(介绍参看 深入理解PHP原理之Session Gc的一个小概率Notice), 这个默认的值分别是1和100, 也就是有1%的机会, PHP会在一个Session启动时, 运行Session gc. 不能保证到30分钟的时候一定会过期.

2. 那设置一个大概率的清理机会呢? 还是不妥, 为什么? 因为PHP使用stat Session文件的修改时间来判断是否过期, 如果增大这个概率一来会降低性能, 二来, PHP使用”一个”文件来保存和一个会话相关的Session变量, 假设我5分钟前设置了一个a=1的Session变量, 5分钟后又设置了一个b=2的Seesion变量, 那么这个Session文件的修改时间为添加b时刻的时间, 那么a就不能在30分钟的时候, 被清理了. 另外还有下面第三个原因.

3. PHP默认的(Linux为例), 是使用/tmp 作为Session的默认存储目录, 并且手册中也有如下的描述:

Note: 如果不同的脚本具有不同的 session.gc_maxlifetime 数值但是共享了同一个地方存储会话数据，则具有最小数值的脚本会清理数据。此情况下，与 session.save_path 一起使用本指令。

也就是说, 如果有俩个应用都没有指定自己独立的save_path, 一个设置了过期时间为2分钟(假设为A), 一个设置为30分钟(假设为B), 那么每次当A的Session gc运行的时候, 就会同时删除属于应用B的Session files.

所以, 第一种答案是不”完全严格”正确的.

第二种答案

还有一种常见的答案是: 设置Session ID的载体, Cookie的过期时间, 也就是session.cookie_lifetime. 这种回答也是不正确的, 原因如下:

这个过期只是Cookie过期, 换个说法这点就考察Cookie和Session的区别, Session过期是服务器过期, 而Cookie过期是客户端(浏览器)来保证的, 即使你设置了Cookie过期, 这个只能保证标准浏览器到期的时候, 不会发送这个Cookie(包含着Session ID), 而如果通过构造请求, 还是可以使用这个Session ID的值.

第三种答案

使用memcache, redis等, okey, 这种答案是一种正确答案. 不过, 很显然出题者肯定还会接着问你, 如果只是使用PHP呢?

第四种答案

当然, 面试不是为了难道你, 而是为了考察思考的周密性. 在这个过程中我会提示出这些陷阱, 所以一般来说, 符合题意的做法是:

1. 设置Cookie过期时间30分钟, 并设置Session的lifetime也为30分钟.

2. 自己为每一个Session值增加Time stamp.

3. 每次访问之前, 判断时间戳.

最后, 有同学问, 为什么要设置30分钟的过期时间: 这个, 首先这是为了面试, 第二, 实际使用场景的话, 比如30分钟就过期的优惠劵?

thanks

###### PHP autoload机制？

2. 普遍的__autoload的实现逻辑

在实现__autoload的时候,绝大多数的实现方式都是根据类名去找对应的文件,这个大家可以参考各种开源软件的实现,估计90%都是这种模式

3. 如何自定义自己的autoload方法

在php官方文档中也提到了,spl_autoload_register() 提供了一种更加灵活的方式来实现类的自动加载。因此，不再建议使用 __autoload()函数，在以后的版本中它可能被弃用。

4. 多个项目中的autoload方法是如何协调工作的

项目中引用了多个第三方的库,比如 smarty等等, 如果你仔细阅读他们的源码,就会发现,其实每个类库里都有他自己的autoload方法,那么这些第三方库的autoload会对我们项目本身的autoload会有什么影响呢?

所以,**多次spl_autoload_register会按照注册顺序来依次调用方法,找到则退出 ( 先注册先调用)**
因此,**最先spl_autoload_register的方法最先调用 ( 先注册先调用)**

结论就是: **调用spl_autoload_register之后,默认的__autoload就会失效,**如果想继续回朔__autoload方法,需要在代码最开始加入如下代码

spl_autoload_register('__autoload');


###### Memcache与Redis？
##### Mysql存储引擎MyIsam与Innnodb差别？
##### 为什么要对数据库进行主从分离? 一般多少数据量开始分表? 分库? 分库分表的目的? 什么是数据库垂直拆分? 水平拆分? 分区等等？可以举例说明

##### 你自己完成缓存系统，你将如何进行设计？
##### web开发方面会遇到哪些缓存? 分别如何优化?
##### 常见的web攻击有哪些？日常编码时你是如何处理的？

1、命令注入(Command Injection)

2、eval注入(Eval Injection)

3、客户端脚本攻击(Script Insertion)

4、跨网站脚本攻击(Cross Site Scripting, XSS)

5、SQL注入攻击(SQL injection)

6、跨网站请求伪造攻击(Cross Site Request Forgeries, CSRF)

7、Session 会话劫持(Session Hijacking)

8、Session 固定攻击(Session Fixation)

9、HTTP响应拆分攻击(HTTP Response Splitting)

10、文件上传漏洞(File Upload Attack)

11、目录穿越漏洞(Directory Traversal)

12、远程文件包含攻击(Remote Inclusion)

13、动态函数注入攻击(Dynamic Variable Evaluation)

14、URL攻击(URL attack)

15、表单提交欺骗攻击(Spoofed Form Submissions)

16、HTTP请求欺骗攻击(Spoofed HTTP Requests)


1、php一些安全配置
(1)关闭php提示错误功能
(2)关闭一些“坏功能”
(3)严格配置文件权限。
2、严格的数据验证，你的用户不全是“好”人
2.1为了确保程序的安全性，健壮性，数据验证应该包括内容。
2.2程序员容易漏掉point或者说需要注意的事项
3、防注入
   3.1简单判断是否有注入漏洞以及原理
   3.2常见的mysql注入语句
       (1)不用用户名和密码
       (2)在不输入密码的情况下，利用某用户
       (3)猜解某用户密码
(4)插入数据时提权
(5)更新提权和插入提权同理
(6)恶意更新和删除
(7)union、join等
(8)通配符号%、_
(9)还有很多猜测表信息的注入sql
   33防注入的一些方法
       2.3.1 php可用于防注入的一些函数和注意事项。
       2.3.2防注入字符优先级。
2.3.3防注入代码
    (1)参数是数字直接用intval()函数
    (2)对于非文本参数的过滤
(3)文本数据防注入代码。
(4)当然还有其他与addslashes、mysql_escape_string结合的代码。
4、防止xss攻击
4.1Xss攻击过程
4.2常见xss攻击地方
4.3防XSS方法
5、CSRF
5.1简单说明CSRF原理
5.2防范方法
6、防盗链
7、防拒CC攻击


###### Redis使用总结之与Memcached异同

这是一篇转帖文，写得很好很详细，让人快速了解了redis.

Redis是什么？两句话可以做下概括：
1. 是一个完全开源免费的key-value内存数据库
2. 通常被认为是一个数据结构服务器，主要是因为其有着丰富的数据结构 strings、map、 list、sets、 sorted sets

Redis不是什么？同样从两个方面来做下对比：
1. 不是sql server、mySQL等关系型数据库，主要原因是：
. redis目前还只能作为小数据量存储（全部数据能够加载在内存中） ，海量数据存储方面并不是redis所擅长的领域
. 设计、实现方法很不一样.关系型数据库通过表来存储数据，通过SQL来查询数据。而Redis通上述五种数据结构来存储数据，通过命令 来查询数据
2. 不是Memcached等缓存系统，主要原因有以下几个：
.网络IO模型方面：Memcached是多线程，分为监听线程、worker线程，引入锁，带来了性能损耗。Redis使用单线程的IO复用模型，将速度优势发挥到最大，也提供了较简单的计算功能
.内存管理方面：Memcached使用预分配的内存池的方式，带来一定程度的空间浪费 并且在内存仍然有很大空间时，新的数据也可能会被剔除，而Redis使用现场申请内存的方式来存储数据，不会剔除任何非临时数据 Redis更适合作为存储而不是cache
.数据的一致性方面：Memcached提供了cas命令来保证.而Redis提供了事务的功能，可以保证一串 命令的原子性，中间不会被任何操作打断
. 存储方式方面：Memcached只支持简单的key-value存储，不支持枚举，不支持持久化和复制等功能

一句话小结一下：Redis是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部分场合可以对关系数据库起到很好的补充作用。

Redis有什么用？只有了解了它有哪些特性，我们在用的时候才能扬长避短，为我们所用：
1. 速度快：使用标准C写，所有数据都在内存中完成，读写速度分别达到10万/20万
2. 持久化：对数据的更新采用Copy-on-write技术，可以异步地保存到磁盘上，主要有两种策略，一是根据时间，更新次数的快照（save 300 10 ）二是基于语句追加方式(Append-only file，aof)
3. 自动操作：对不同数据类型的操作都是自动的，很安全
4. 快速的主–从复制，官方提供了一个数据，Slave在21秒即完成了对Amazon网站10G key set的复制。
5. Sharding技术： 很容易将数据分布到多个Redis实例中，数据库的扩展是个永恒的话题，在关系型数据库中，主要是以添加硬件、以分区为主要技术形式的纵向扩展解决了很多的应用场景，但随着web2.0、移动互联网、云计算等应用的兴起，这种扩展模式已经不太适合了，所以近年来，像采用主从配置、数据库复制形式的，Sharding这种技术把负载分布到多个特理节点上去的横向扩展方式用处越来越多。

这里对Redis数据库做下小结：
1. 提高了DB的可扩展性，只需要将新加的数据放到新加的服务器上就可以了
2. 提高了DB的可用性，只影响到需要访问的shard服务器上的数据的用户
3. 提高了DB的可维护性，对系统的升级和配置可以按shard一个个来搞，对服务产生的影响较小
4. 小的数据库存的查询压力小，查询更快，性能更好

写到这里，可能就会有人急不可待地想用它了，那怎么用呢？可以直接到官方文档，里面帮我们整理好了各个语言环境下的客户端，主要有Ruby、Python、 PHP、Perl、Lua、Java、C#….有几种语言，我也没见过，所以就不多说了，你懂的….

最后，把我使用过程中的一些 经验与教训，做个小结：
1. 要进行Master-slave配置，出现服务故障时可以支持切换。
2. 在master侧禁用数据持久化，只需在slave上配置数据持久化。
3. 物理内存+虚拟内存不足，这个时候dump一直死着，时间久了机器挂掉。这个情况就是灾难！
4. 当Redis物理内存使用超过内存总容量的3/5时就会开始比较危险了，就开始做swap,内存碎片大
5. 当达到最大内存时，会清空带有过期时间的key，即使key未到过期时间.
6. redis与DB同步写的问题，先写DB，后写redis，因为写内存基本上没有问题


### MySQL优化方式

第一方面：30种mysql优化sql语句查询的方法

1.对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。

2.应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。

3.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

```
　　select id from t where num is null
　　可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：
　　select id from t where num=0
```

4.应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：

```
　　select id from t where num=10 or num=20
　　可以这样查询：
　　select id from t where num=10
　　union all
　　select id from t where num=20
```


5.下面的查询也将导致全表扫描：

```
　　select id from t where name like '%abc%'
```
　　
　　若要提高效率，可以考虑全文检索。
　

6.in 和 not in 也要慎用，否则会导致全表扫描，如：

```
	select id from t where num in(1,2,3)
　　对于连续的数值，能用 between 就不要用 in 了：
　　select id from t where num between 1 and 3
```

7.如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时;它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```
　　select id from t where num=@num
　　可以改为强制查询使用索引：
　　select id from t with(index(索引名)) where num=@num
```


8.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：

　
```
	select id from t where num/2=100
　　 应改为:
　　 select id from t where num=100*2
```


9.应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：
　　
```
　　select id from t where substring(name,1,3)='abc'--name以abc开头的id
　　select id from t where datediff(day,createdate,'2005-11-30')=0--'2005-11-30'生成的id
　　应改为:
　　select id from t where name like 'abc%'
　　select id from t where createdate>='2005-11-30' and createdate<'2005-12-1'
```　　
　　
　　10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
　　
　　11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
　　
12.不要写一些没有意义的查询，如需要生成一个空表结构：

```
　　select col1,col2 into #t from t where 1=0
　　这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：
　　create table #t(...)
```

13.很多时候用 exists 代替 in 是一个好的选择：
```　　
　　select num from a where num in(select num from b)
　　用下面的语句替换：
　　select num from a where exists(select 1 from b where num=a.num)
```　　
　　
　　14.并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。
　　15.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
　　16.应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。
　　17.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
　　18.尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
　　19.任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。
　　20.尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限(只有主键索引)。
　　21.避免频繁创建和删除临时表，以减少系统表资源的消耗。
　　22.临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使用导出表。
　　23.在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度;如果数据量不大，为了缓和系统表的资源，应先create table，然后insert。
　　24.如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。
　　25.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写。
　　26.使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。
　　27.与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。
　　28.在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。
　　29.尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。
　　30.尽量避免大事务操作，提高系统并发能力。

上面有几句写的有问题。

第二方面：
select Count (*)和Select Count(1)以及Select Count(column)区别
一般情况下，Select Count (*)和Select Count(1)两着返回结果是一样的
    假如表沒有主键(Primary key), 那么count(1)比count(*)快，
    如果有主键的話，那主键作为count的条件时候count(主键)最快
    如果你的表只有一个字段的话那count(*)就是最快的
   count(*) 跟 count(1) 的结果一样，都包括对NULL的统计，而count(column) 是不包括NULL的统计

第三方面：
索引列上计算引起的索引失效及优化措施以及注意事项

创建索引、优化查询以便达到更好的查询优化效果。但实际上，MySQL有时并不按我们设计的那样执行查询。MySQL是根据统计信息来生成执行计划的，这就涉及索引及索引的刷选率，表数据量，还有一些额外的因素。
Each table index is queried, and the best index is used unless the optimizer believes that it is more efficient to use a table scan. At one time, a scan was used based on whether the best index spanned more than 30% of the table, but a fixed percentage no longer determines the choice between using an index or a scan. The optimizer now is more complex and bases its estimate on additional factors such as table size, number of rows, and I/O block size.
简而言之，当MYSQL认为符合条件的记录在30%以上，它就不会再使用索引，因为mysql认为走索引的代价比不用索引代价大，所以优化器选择了自己认为代价最小的方式。事实也的确如此

是MYSQL认为记录是30%以上，而不是实际MYSQL去查完再决定的。都查完了，还用什么索引啊？！
MYSQL会先估算，然后决定是否使用索引。