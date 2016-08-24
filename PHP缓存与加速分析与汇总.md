php缓存与加速分析与汇总

 这篇博文笔者构思了很久，当然也写了很久，发现想到和写出来完全两码事，写下来会更深刻些弄清很多模糊概念。看张图


环境：win7 32 + apache2.2 + php5.28 +  5.1.49-community + chrome 22.0.1

- 一、    浏览器端缓存

要利用浏览器缓存则要先了解http协议内容，这里主要利用http协议头部header的一些头域名，主要“Expires”，“Etag”，“Last-Modified”；
先看张原理图：

当然还有其他http头域上图没说明，要详细点了解http协议和不同浏览器4个浏览器操作的解释可以移步：
http://www.phpben.com/?post=77
还有如何通过浏览器监听http请求（笔者喜欢用chrome）：
http://www.phpben.com/?post=76
ps:下面给出的http请求头部监听结果都是在chrome浏览器下的结果

1、先来了解几个问题
(1)在没任何缓存情况下。浏览器请求一个页面发送http请求不止一次，要根据页面中uri资源个数而定。
在http协议中，浏览器第一次处理服务器返回web内容的时候，也就是浏览器还没任何缓存的时候，向服务器发送http请求数是多个，也就是说浏览器不是一次处理完一个页面中所有uri资源。
百度首页部分http请求图：

你会发现status（状态行）都是200，size内容都是笫一次加载。
注意：测试时要确保没缓存，仅仅在浏览器清除是不彻底的，要到相应缓存库中删除所有，如笔者路径的：C:\Users\benwin\AppData\Local\Google\Chrome\User Data\Default\Cache ，否则你会看到如下（这是第二次刷新，浏览器已有缓存）

这也是一些前端强调写css的时候把一些图标，按钮图片合成一张用background-position定位坐标，这样加载时只请求一次则把所有图标加载完。
(2)apache对处理静态文件和动态文件返回头部信息有所不同。
Apache对于浏览器请求html、css、js、图片等文件时返回头部信息时会自动加上etag（apache默认打开），last-modified头域，而动态文件如php后缀文件则不会，除非php文件代码中加入header("Expires: …")、header("Last-modified:…")、header("Etag…")。
代码：

- <?php
- echo '<script src="test.js" type="text/javascript"></script>
- <link rel="Stylesheet" type="text/css" href="test.css" />
- <img src="test.jpg" />';
-  ?>

加载效果和监听结果图：

(3)浏览器利用expires头域和last-modified或etag的一个不同点。对于expires是先检查是否过期决定是否发请求；后两者是会发http请求给服务器，服务器判断是重新解析返回结果还是直接不用重新解析只返回304 Not Modified告诉浏览器请求内容没改变可以用原来的缓存。
Expires:浏览器在服务器首次返回http头中若有expires头域且有效则缓存内容，第二次发请求前会先根据expires是否过期来决定是否发请求，没过期则不会发请求，而是从浏览器缓存中拿数据，如下图没过期的http监听效果图：

上图status 200变白，和size显示from cache。过期则重新发http请求
last-modifie:第一次请求服务器返回last-modified，浏览器第二次发请求前向服务器发If-Modified-Since（值是last-modified返回的值），服务器根据If-Modified-Since来判断文件是否修改来做不同反应，如果If-Modified-Since相同则返回 304 Not Modified,否则服务器发新的last-modified回去。浏览器第i次对于last-modified的处理和第二次相同。

Etag:浏览器对etag处理和对last-modified处理一样，只不多（last-modified，If-Modified-Since）配对，（etag，If-None-Match）配对。

2、常见的缓存方法。
(1)使用http Expires头域的时间缓存（有个有趣的情况）
如下面告诉浏览器一小时过期代码：

- header('Expires:'.date('D, d M Y H:i:s \G\M\T', time()+3600));

适用范围：静态页面html如一些新闻网内容页
笔者发现个有趣的事情：
代码1：

- header('Expires:'.date('D, d M Y H:i:s \G\M\T', time()+60));
- session_start();
- $_SESSION['url']='www.phpben.com';

http监听结果1：

- HTTP/1.1 200 OK
- Expires: Thu, 19 Nov 1981 08:52:00 GMT
- Content-Type: text/html
- ……

代码2：

- session_start();
- header('Expires:'.date('D, d M Y H:i:s \G\M\T', time()+60));
- $_SESSION['url']='www.phpben.com';

http监听结果2：

- HTTP/1.1 200 OK
- Expires: Mon, 29 Oct 2012 13:49:19 GMT
- Content-Type: text/html
- ……

笔者结论：
两个Expires结果因为session_start();的顺序不同结果不同。
header('Expires:'.date('D, d M Y H:i:s \G\M\T', time()+60));当后面有session_start();的时候，Expires会被改写成Expires:Thu, 19 Nov 1981 08:52:00 GMT，也就是Expires会被重写，也可以说是优先级别高，所以要在有session情况使用Expires所以要注意顺序问题，这里也说明expires方法在某种程度上不是很适合于动态页面-----话说笔者在这里吃了不少苦头
(2)使用http last-modified和if-modified-since 头域的时间缓存
实现代码：

- if(isset($_SERVER['HTTP_IF_MODIFIED_SINCE']) && (time()-strtotime($_SERVER['HTTP_IF_MODIFIED_SINCE']) < 10))
- {
-     header("HTTP/1.1 304 Not Modified");
-     exit;
- }else
- {
-  //网上很多代码都没考虑齐全，都是下面两个版本之一而已
-  if (date_default_timezone_get()=='UTC')
-  {
-    header("Last-Modified:" . date('D, d M Y H:i:s \G\M\T', time()));
-  }else //这是针对时区设为PRC、Asia/ShangHai等中国时区
-  {
-    header("Last-Modified:" . date('r', time()));
-  }
- }

为什么要考虑两种情况（话说笔者以为是php一个bug后来检测了不是）。
-------------
环境：apche2.2+php5.2.8  和apache2.2+php5.3.3 和apache2.2+php5.4.7
测试(1)
代码：

- $a= date('D, d M Y H:i:s \G\M\T', time());
- echo $a,'<br/>';
- echo date('D, d M Y H:i:s \G\M\T', strtotime($a));

怎么多8小时了呢？
utc时间=gmt时间
utc时间+8小时=prc时间=北京时间（是不是忘记了？）
而的结果也是类似多8小时
所以如果你服务器time.zone设置utc没问题，但如果不是，则像以下代码是不起作用的

- if(isset($_SERVER['HTTP_IF_MODIFIED_SINCE']) && (time()-strtotime($_SERVER['HTTP_IF_MODIFIED_SINCE']) < 10))
- {
-     header("HTTP/1.1 304 Not Modified");
-     exit;
- }
- header("Last-Modified:" . date('D, d M Y H:i:s \G\M\T', time()));

因为time()-strtotime($_SERVER['HTTP_IF_MODIFIED_SINCE'])已经是负数
根本原因：strtotime()函数转化成时检测到“GMT”然后在time.zone=prc配置下 所以加上八小时，所以出现多八小时的情况。
所以说不要随便复制网上的代码。
原来以为是php的一个bug所以检测几个版本，后来发现去掉“GMT”后正常了.
-----------
提问：为什么要在代码加上这个，apache或其他服务器本来不是有判断自己返回？
回答：前面提到apache服务器对待动态页面和html的不一样，不会自动加上Last-Modified返回给浏览器，加上用户不停刷新（或按F5），那apache则只解析运行到上面段代码后停止，返回304给浏览器，浏览器用回缓存。
适合范围：一般在php文件里这段代码和expires头域一起用，这样既能在浏览器端减少发请求，也能在服务器端减少运行代码量从而提高性能。
值得一提：此方法在服务器集群的时候仍然适合使用，这个是Etag和if-none-match的鸡肋来的。
(3)使用Etag和if-none-match头域的缓存
Apache对于etag头部是默认开启的，且会给html、js、css等静态文件自动返回，动态如php文件默认没有，像Last-Modified一样需要加上代码。
不同服务器返回的etag格式不同。
Apche默认FileETag INode Size MTime来设置etag包含的内容。
1)INode表示文件的索引节数被包含在内
2)Size表示文件的字节数被包含在内
3)MTime表示文件的最后修改时间包含在内
4)All表示包含INode Size MTime
5)None表示如果一个文档是基于文件的，不在回应中包含任何ETag域
如“50000000536da-311-488a45c786210” 50000000536da是INode，311是字节数，488a45c786210是MTime
注意：
如果包含INode Size MTime三者，不管“FileETag INode Size MTime”语句3个顺序如何，apache生成的顺序都是按照 INode-Size-MTime顺序
常见代码：

- if(isset($_SERVER['HTTP_IF_NONE_MATCH']) && ($_SERVER['HTTP_IF_NONE_MATCH']==date('Y-m-d H:i',time())))
- {
-   header("HTTP/1.1 304 Not Modified");
-   exit();
- }
- header('Etag:'.date('' Y-m-d H:i '', time()));

以上代码就是last-modified的复制，last-modified能用到的时候上面代码可以用。
适用范围：动静态页面，单服务器模式。不适合集群服务器.
是否矛盾：前面说last-modified在集群的时候可以用，又说不适合集群服务器，岂不是矛盾，那就要看etag的鸡肋
如一个站有两台server1和server2两台服务器，假如php文件中没上面代码（有的话就是last-modified的复制了）
1)在负载均衡的情况下首先server1处理静态文件是否自动返回了etag值10000000536db-44c2-4cd59ff9c752c
2)然后第二次请求时候负载均衡到server2，因为第一次的时候server2并没有记录到10000000536db-44c2-4cd59ff9c752c这个值，所以不能判断而是生成属于自己的etag值，这样etag值10000000536db-44c2-4cd59ff9c752c起不了作用了。如果服务器集群很庞大的时候就成了鸡肋了。
但是如果自动加上上面的代码，不过那台服务器处理，还是要执行代码，所以起作用了。

- 二、    利用apache服务器端缓存和加速

(1)启用apache缓存模块 mod_expires
启用动态加载模块mod_expires.so，启用该模块apache会根据你设定的配置规则返回相应expires和cache-control头域，相当于代码: header('Expires:' ...); header('Cache-Control: max-age=second ');
优点：
1)比起last-modified和etag头域减少http请求,从而减服务器和宽带负担
2)可以适用服务器集群，并没有etag鸡肋问题
示例代码例说：
在apache配置文件httpd.conf中加入：

- LoadModule expires_module modules/mod_expires.so
- ExpiresActive On
- ExpiresByType text/css "access plus 1 seconds"
- ExpiresByType tex/html "access plus 1 minutes"
- ExpiresByType application/javascript "access plus 1 hours"
- ExpiresByType application/x-javascript "access plus 1 days"
- ExpiresByType image/gif "acess plus 1 months"
- ExpiresByType image/png "acess plus 1 months"
- ExpiresByType image/jpeg "acess plus 1 months"
- ExpiresByType image/x-jpeg "acess plus 1 months"
- ExpiresByType application/x-shockwave-flash "acess plus 1 months"

其中
LoadModule expires_module modules/mod_expires.so 动态加载缓存模块mod_expires.so；ExpiresActive On表示激活缓存
access表示允许缓存
plus表示后面必须是整数
seconds、minutes、days、months、years就不用说了，还有单位单复数要求不严格
还有3种语法：
1)ExpiresByType text/css "a60"相当于ExpiresByType text/css  "access plus 1 minute"
上面代码按照这种语法如下：

- LoadModule expires_module modules/mod_expires.so
- ExpiresActive On
- ExpiresByType text/css a1
- ExpiresByType tex/html a60
- ExpiresByType application/javascript a3600
- ExpiresByType application/x-javascript a86400
- ExpiresByType image/gif a2678400
- ExpiresByType image/png a2678400
- ExpiresByType image/jpeg a2678400
- ExpiresByType image/x-jpeg a2678400
- ExpiresByType application/x-shockwave-flash a2678400

2)ExpiresByType text/css "m60"表示文件最后修改的60秒过期，m等同于modification

- ExpiresByType image/jpeg "modification plus 1 months"

3)ExpiresDefault这个表示默认文档类型文件缓存

- ExpiresDefault "access plus 1 days"

这种写法主要补上其他非文档类型的缓存
服务器生成效果
语句：

- ExpiresByType text/css "access plus 1 hours"

服务器处理mime类型是text/css也就是css文件时，效果等同于php语句：

- header("Cache-Control:max-age=3600");
- header("Expires:". date('D, d M Y H:i:s \G\M\T', time()+3600);

浏览器端显示头域如下：

- Cache-Control: max-age=3600
- Expires: Fri, 09 Nov 2012 05:37:22 GMT

注意事项：
1)Cache-Control: max-age=3600级别比Expires: Fri, 09 Nov 2012 05:37:22 GMT高，也就是浏览器检查到有max-age且有效会忽略Expires
2) ExpiresByType text/js "access plus 1 days"对于js文件是不起作用的，其中text/js改成application/javascript
3) ExpiresByType application/x-javascript "access plus 1 days"这个类型是mime非正规类型，有些网站用到，考虑代码移植性加上去
4) ExpiresByType image/jpg "acess plus 1 hours"这样不起作用，其中的image/jpg应该改成image/jpeg，这里就不贴测试内容了。
更多mod_expires 语法、作用域等信息查看手册：
http://www.phpchina.com/resource/manual/apache/mod/mod_expires.html
--------------------------------------------------------
(2)启用apache header模块 mod_headers
mod_expires设置缓存是为每重mime文件设置缓存时间，多且烦，要简短的话则可以启用apache mod_headers模块，而且mod_expires设置效果只能等效于http头cache-control和expires头域，mod_headers则可以设置、更改、删除http头所有头域，范围更广。这里只讲mod_headers模块的header指令
header指令格式：
header set 头域 value
value的的格式根据头域而变化，请看下面一个例子
在httpd.conf里加入

- LoadModule headers_module modules/mod_headers.so
- <FilesMatch "\.(pdf|swf|js|css|gif|jpeg|jpg|png|flv|ico|html)$">
- Header set Cache-Control "max-age=604800"
- </FilesMatch>

上面代码第一行表示服务器匹配客户端请求文件的后缀是pdf|swf|js|css|gif|jpeg|jpg|png|flv|ico|html则设置cache-control max-age=604800 让浏览器缓存这些文件1天
当然还可以设置其他头域如：
Header set Last-Modified  "Thu, 10 Jun 2010 03:10:14 GMT"
Header set Expires  "Thu, 10 Jun 2012 03:10:14 GMT "
优点：
1)可以任意增删改header头部
2）设置代码简洁
3）因为可以根据后缀设置，所以可以为动态页面设置头域名
4）前面介绍到的etag、last-modified、和mod-Expires模块功能都可以涵盖（v5）
缺点：
 Last-Modified, Expires不能动态变化，只能设定某个值，则需要人为的根据项目需求不断修改相应的值，没代码header('Expires:'.date('D, d M Y H:i:s \G\M\T', time()+60));那么灵活。
注意：因为apache匹配后缀，则像jpeg、jpe这些同mime类型的不同后缀文件都要填如filematch里面
更多mod_expires 语法、作用域等信息查看手册：
http://www.phpchina.com/resource/manual/apache/mod/mod_headers.html
---------------------------------------------------------
(3)启用apache压缩模块 mod_deflate
在httpd.conf加入

- LoadModule deflate_module modules/mod_deflate.so
- <Location />
- # 插入过滤器,启用压缩
- SetOutputFilter DEFLATE
- # Netscape 4.x 有一些问题...
- BrowserMatch ^Mozilla/4 gzip-only-text/html
- # Netscape 4.06-4.08 有更多的问题，取消压缩
- BrowserMatch ^Mozilla/4\.0[678] no-gzip
- # MSIE 会伪装成 Netscape ，但是事实上它没有问题，所以这里是修复
- BrowserMatch \bMSIE !no-gzip !gzip-only-text/html
- # 不压缩图片
- SetEnvIfNoCase Request_URI .(?:gif|jpe?g|png)$ no-gzip dont-vary
- </Location>

监听http头会多一个头域：
Accept-Encoding: gzip,deflate,sdch
说明压缩有效了。
优点：
减少宽带加快速度
更多mod_deflate 语法、指令、作用域等信息查看手册：
http://man.chinaunix.net/newsoft/Apache2.2_chinese_manual/mod/mod_deflate.html
--------------------------------------------------------------
(4)启用apache模块 mod_cache
前面提到的mod_expires和mod_headers（缓存部分）都是apache根据http协议返回相应的缓存头域（cache-control、Expires、etag、last-modified）给服务器，让客户端的浏览器使用缓存达到加速优化的效果，而mod_cache模块则是在apache端缓存数据从而达到优化的目的。前两者从客户端缓存考虑，后者是服务器端考虑。
相比起来后者可以把同一页面user one 产生的缓存返回给user two
mod_cache一般结合模块mod_mem_cache(基于内存缓存管理模块，把数据缓存进主存)或mod_disk_cache(基于磁盘的缓冲管理器，把数据缓存进磁盘)
1) mod_mem_cache

- <IfModule mod_cache.c>
- #加载mod_mem_cache
- LoadModule mem_cache_module modules/mod_mem_cache.so
- <IfModule mod_mem_cache.c>
- #缓存folderName文件内容
- CacheEnable mem /folderName
- #内存缓冲大小这里是4G
- MCacheSize 4096
- #用最近最少LRU算法，简单说内存满了，则找出最近最少使用的缓存块写入新数据。memcache也是用该算法
- MCacheRemovalAlgorithm LRU
- #最大缓存对象个数，这里1000,如果缓存对象超过此值则根据相应算法（这里是LRU）移除一个来储存最新对象，此值必须大MCacheMinObjectSize
- MCacheMaxObjectCount 1000
- #最小缓存对象个数
- MCacheMinObjectSize 1
- #每个对象最大容量，这里是2G，大于此值的对象会被丢弃，此值该根据/foldername下
- MCacheMaxObjectSize 2048
- #每个对象最大缓存时间
- CacheMaxExpire 864000
- #每个对象默认缓存时间
- CacheDefaultExpire 86400
- #指定哪些内容不缓存，这里表示noCacheFolder文件夹内内容不缓存
- CacheDisable /noCacheFolder
- </IfModule>
- </IfModule>

注意：

- l  MCacheMaxObjectCount > MCacheMinObjectSize
- l  MCacheMaxObjectSize和MCacheMinObjectSize值的确定根据folderName里内容确定
- l  CacheDefaultExpire值需要根据web访问强度，峰值等影响因素，一般需要不断测试

2) mod_disk_cache

- LoadModule cache_module modules/mod_cache.so
- <IfModule mod_cache.c>
- LoadModule disk_cache_module modules/mod_disk_cache.so
- <IfModule mod_disk_cache.c>
- #缓存数据目录
- CacheRoot D:/
- #缓存folderName文件内容
- CacheEnable disk /folderName
- #缓存目录的子目录层数，这里是4
- CacheDirLevels 4
- #缓存文件最大大小
- CacheMaxFileSize 333
- #缓存文件最小大小
- CacheMinFileSize 1
- #缓存文件名长度
- CacheDirLength 3
- </IfModule>
- </IfModule>

更多查看手册：
http://man.chinaunix.net/newsoft/Apache2.2_chinese_manual/mod/mod_cache.html

- 三、    Php代码入手

(1)css 背景图片尽量合成一张图，用background-image and background-position定位图片，从而减少http请求数，且避免css表达式尽量避免。
(2)css文件放在页面的前面，js文件放在后面。很容想到css是布局用得，让用户最先看到网页的内容js一般是内容齐全后才用到。这样用户觉得速度快很多，加载js的时候网页的布局已经好了，虽然事实上不是。
(3)ajax如果不是大数据，多cookies不用用post而是用get，因为post会发送两次tcp包，第一个发送头部，第二个才是内容。
(4)$bloger=$benwin . 'phpben.com ';代替$bloger= "$benwin phpben.com";
(5) if ("" == $var)代替if ($var == "")
(6)explode()代替split()
(7)$len=count('benwin');for($i=0;$i<$len;$i++)代替for($i=0;$i< count('benwin');$i++)
(8)正则匹配尽量少用“*”
(9)$i+=1;代替$i=$i+1;
(10)if(!isset($bloger{5}))代替if(strlen($bloger)<5)
(11)echo $bloger, ':', 'benwin';代替echo $bloger. ':'. 'benwin';
(12)避免重复代码，写成一个函数调用
(13)手动unset释放内存，特别是大数组，大对象，及时unset,而不是等脚本执行完自动回收
(14)数据库连接用完记得关闭
(15)抑制错误符号@尽量少用
更多可以看看：
雅虎优化35条：http://developer.yahoo.com/performance/rules.html  （非永久）
Php性能测试：http://maettig.com/code/php/php-performance-benchmarks.php （非永久）

- 四、    建立文件缓存


文件缓存在某种程度上有点想apache mod_disk_cache模块，一般都包缓存写在磁盘文件里，等用户访问的时候先判断是否存在缓存，存在者直接读取缓存返回，否则则写入新的缓存。不同的mod_disk_cache是这些事情读取操作apache帮你完成，而这里说的文件缓存是需要程序员按照自己根据程序功能，以自己的经验，熟悉的算法规划出一定的缓存机制。
现在不同公司都会自己写自己的文件缓存机制，包括一些大开源php系统比如phpcms、ecshop、phpwind、dedecms等，导致在php文件缓存机制方面没什么统一（smarty算统一一个）如ecshop改写smarty的缓存机制，phpcms只要三个类库pc_base 和cache_factory cache_file_class.php
(1)文件缓存机制一般需要考虑的几个环节。
写（write_cache），读（read_cache），删（delete_cache），重写（rewrite_cache）
(2)要考虑的几个问题：读写涉及的文件唯一，数据锁定，缓存时间
1)单目录文件缓存，网站中所有静态页面（html），动态页面（sort.php?id=11）都保存在一个页面。为了方便读取，相互缓存文件之间的文件名必须唯一，第一时间想到md5(url)加密后作为文件名，或则自己写一个hash算法生成文件名。文件名是否有点长？
2)多层目录文件缓存
这种比较常见，一般根据不同模块，不同分类来建立缓存目录，然后这里肯定涉及到配置问题，读写的时候根据哪个目录，这时你的缓存类就需要配置、路径转换等问题了。
Module1/sort.php?id=11这样的路径则可以把缓存写入Module1文件夹的sort_11.php
3)锁定一般用到互斥锁，读不能写，写不能读
4)设置缓存时间。
在读的时候判断缓存文件什么时候创建（或最后一次修改），和当前时间相比如过期则重新生成缓存。还有一种方法是把缓存过期时间写入缓存文件的第一行或最后一行，读取内容是截取出来判断
5)数据完整性
笔者再做ecshop开发的时候曾经遇到缓存文件在写入的时候没完整写入（系统崩溃、断电等原因），读出来的数据肯定是不完整。解决这问题一种方法可以加入类型mysql redo的机制，专门用一个文件保存写入的信息包括写入时间，路径，内容，缓存时间，是否完整写入，否则重写入。还有一种笔者较喜欢的是，可以在写入文件末尾加上一个“判断量”比如加上“[phpben.com]”，在读取的时候先判断文件是否有该变量，有则是完整的，否者则删除或重写。后一种方法没那么复杂，容易实现。
(3)实现环节简述
这里是简单说下流程，没具体实现，具体实现可以googel以下或看开源系统（一般穿插很多内容）
1)配置环节，set_config，主要是一些缓存路径path，默认缓存时间，是否开启互斥锁等操作 function set_config (path=defaultPath,expire=defaultTime,isLock=ture,…)
2)have_cache(fil_name)用于调用时判断文件中是否有缓存，有则通过read_cache读取缓存，否则执行文件且写入缓存write_cache
3)写环节（write_cache），判断文件是否存在，不存在则创建（注意权限），根据规则生成缓存文件的名字（同目录下必须唯一），具体算法可以md5(url)，sort.php?id=11改成sort_11.php等等。内容中是加入缓存时间，加入完整写入判断，当然肯定写入缓存内容。锁是否锁定等
function write_cache(filName,filContent,expire,…)
4)读环节 read_cache，按照规则生成文件名和路径，锁定文件，读取文件，判断文件是否过期，过期则删除，否则继续判断文件是否是完整写入文件（很多没这个功能），完整则返回数据,否则返回
functions read_cache(filName,…)
5)重写环节，更新了数据则需要重写，和写环节一样，一般触发此环节都是人为触发。
上面说的是实现环节
ps:具体缓存机制时函数名肯定是花样百出，还有许多细节东西是写不出来的，这也是问什么文件缓存并没有设计模式哪有有统一的模式的原因。

- 五、    优化Mysql加速

关于mysql可以看笔者另外一篇文章：http://www.phpben.com/?post=74


- 六、    Memcache和Redis

这两个篇幅太大了且很重要的缓存模块，笔者以后会独立总结和介绍。
至于那个比较好和趋势：
引用黑夜路人前辈的一句话（此话鸟哥也转发）：
“在互联网开发领域，Redis + MongoDB (MySQL) 的模式正在逐步取代 Memcached + MySQL 的模式，就像曾经 Apache + PHP(mod_php) 的模式逐步已经被 Nginx + PHP(php-fpm) 在取代一样，这个是技术历史发展的逐步行进”

- 七、    Opcode加速器

有了解过php源码的会知道，Zend引擎执行php代码经过以下阶段

而php每次执行都要经过这些步骤，然后opcode加速器(缓存)就产生了，从而加快速度。
老实说，笔者本着专研的精神，以后在这方面可是要加把劲了（^_^）。ps：图片来自鸟哥博客。
现在比较多的加速器有：apc、Xcache、eAccelerater、facebook开发的HipHop VM
八、    LVS、TFS、BFS、Hadoop
自己搜索一下这些关于调度、集群、分布式的资料。
参考资料：
http://man.chinaunix.net/newsoft/Apache2.2_chinese_manual/mod/mod_cache.html
http://man.chinaunix.net/newsoft/Apache2.2_chinese_manual/mod/mod_deflate.html
http://www.phpchina.com/resource/manual/apache/mod/mod_expires.html
http://maettig.com/code/php/php-performance-benchmarks.php
http://developer.yahoo.com/performance/rules.html
