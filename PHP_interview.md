PHP面试题级

### 一般有三年以上php开发经验去百度、腾讯面试，常会接触的面试题小总结一下：

0.简单做一下自我介绍,?  然后谈一下近三年来你的得意之作?

1.面试官看过你的简历，会问一些你做的项目的用户量、pv、吞吐量、相关难点和解决方法等

2.数据库设计经验,为什么进行分表? 分库?
   一般多少数据量开始分表? 分库? 分库分表的目的? 什么是数据库垂直拆分? 水平拆分? 分区等等？可以举例说明

3.数据库优化有哪些? 分别需要注意什么?

4.web开发方面会遇到哪些缓存? 分别如何优化?

5.给你256M的内存,对10G的文件进行排序(文件每行1个数字),如何实现？
   对10G的文件进行查找如何实现？
   统计10G文件每个关键字出现的次数如何实现？

6.假如你现在是12306火车订票的设计师,你该如何设计满足全国人民订票?

7.假如有1亿用户的访问量,你的服务器架构是怎样的? 用户信息的存储方案如何设计?

8.如果你是技术组长,所带团队任务进度无法完成你该如何解决?
   如果在进度排满的前提下插入任务,你该如何保证总进度不延期?
   如果有的工程师今天预定任务没有完成,你该如何解决?
   
9.从你的经验方面谈一下如何构建高性能web站点? 需要哪些环节? 步骤? 每个步骤需要注意什么如何优化等?

10. 为什么要对数据库进行主从分离? 

11. 如何处理多服务器共享session?

12. 一个10G的表,你用php程序统计某个字段出现的次数,思路是?

13. 会告诉你一个nginx日志例子,用你认为最佳的编程语言统计一下http响应时间超过1秒的前10个url?

14. 给你一个mysql配置文件,用你认为最佳的编程语言解析该文件?

15. 给你两个路径a和b,写一个算法或思路计算a和b差距几层并显示a和b的交集?

16. 给你一个url,在nginx配置一下rewrite指定到某个具体路径?

17. 一个php文件的解释过程是? 一般加速php有哪些?  提高php整体性能会用到哪些技术?

18. session和cookie生存周期区别? 存储位置区别?

19. require、include、require_once、include_once区别? 加载区别? 如果程序按需加载某个php文件你如何实现?

20. chrome号称为多线程的,所以多线程和多进程的区别为?

21. php在2011年底出现hash碰撞,hash碰撞原理为? 如何进行修复?

22. web不安全因素有哪些? 分别如何防范?

23. 假如两个单链表相交,写一个最优算法计算交点位置,说思路也可以?

24. 假如你是技术组长? 如何提高团队效率?

25. nginx负载均衡有哪些? 如果其中一台服务器挂掉,报警机制如何实现?

26. 不优化前提下,apache一般最大连接数为? nginx一般最大连接数为? mysql 每秒insert ? select ? update ? delete?

27. mysql 数据类型有哪些 ? 分别占用多少存储空间 ?

28. nginx设置缓存js、css、图片等信息,缓存的实现原理是?

29. 如何提高缓存命中率? 如何对缓存进行颗粒化?

30. php的内存回收机制是?

31. 我的所有问题都问完了,你有什么问题问我没有？


### 二、算法题
1. 使用PHP描述冒泡排序和快速排序算法，对象可以是一个数组
2. 使用PHP描述顺序查找和二分查找（也叫做折半查找）算法，顺序查找必须考虑效率，对象可以是一个有序数组
3. 写一个二维数组排序算法函数，能够具有通用性，可以调用php内置函数



####### 1. 使用PHP描述冒泡排序和快速排序算法，对象可以是一个数组

//冒泡排序（数组排序）

```
<?php
function bubble_sort($array){
    $count = count($array);
    if ($count <= 0) return false;
    for($i=0; $i<$count; $i++){
        for($j=$i; $j<$count-1; $j++){
            if ($array[$i] > $array[$j]){
                $tmp = $array[$i];
                $array[$i] = $array[$j];
                $array[$j] = $tmp;
            }
        }
    }
    return $array;
}
```

//快速排序（数组排序）

```
<?php
function quick_sort($array) {
    if (count($array) <= 1) return $array;
    $key = $array[0];
    $left_arr = array();
    $right_arr = array();
    for ($i=1; $i<count($array); $i++){
        if ($array[$i] <= $key)
            $left_arr[] = $array[$i];
        else
            $right_arr[] = $array[$i];
    }
    $left_arr = quick_sort($left_arr);
    $right_arr = quick_sort($right_arr);
    return array_merge($left_arr, array($key), $right_arr);
}
```

####### 2. 使用PHP描述顺序查找和二分查找（也叫做折半查找）算法，顺序查找必须考虑效率，对象可以是一个有序数组

//二分查找（数组里查找某个元素）

```
<?php
function bin_sch($array, $low, $high, $k){
    if ($low <= $high){
    $mid = intval(($low+$high)/2);
    if ($array[$mid] == $k){
    return $mid;
    }elseif ($k < $array[$mid]){
    return bin_sch($array, $low, $mid-1, $k);
    }else{
    return bin_sch($array, $mid+1, $high, $k);
    }
    }
    return -1;
}
```


//顺序查找（数组里查找某个元素）

```
<?php
function seq_sch($array, $n, $k){
    $array[$n] = $k;
    for($i=0; $i<$n; $i++){
        if($array[$i]==$k){
            break;
        }
    }
    if ($i<$n){
        return $i;
    }else{
        return -1;
    }
}
```


###### 3. 写一个二维数组排序算法函数，能够具有通用性，可以调用php内置函数

//二维数组排序， $arr是数据，$keys是排序的健值，$order是排序规则，1是升序，0是降序

```
<?php
function array_sort($arr, $keys, $order=0) {
    if (!is_array($arr)) {
        return false;
    }
    $keysvalue = array();
    foreach($arr as $key => $val) {
        $keysvalue[$key] = $val[$keys];
    }
    if($order == 0){
        asort($keysvalue);
    }else {
        arsort($keysvalue);
    }
    reset($keysvalue);
    foreach($keysvalue as $key => $vals) {
        $keysort[$key] = $key;
    }
    $new_array = array();
    foreach($keysort as $key => $val) {
        $new_array[$key] = $arr[$val];
    }
    return $new_array;
}
```


## 三、 设计一个缓存系统。写出思路。画出图。考虑命中，生存期等多种要素。


1. 确定缓存接口: (这里只列举set和get来说明, 可以增加delete等等...)


```
<?php
interface Icache {
    function get($_key);
 
    function set($_key, $_val, $_time);
}
?>
```

2. 各种缓存实现:

(1). 文件缓存:  按照一个文件一个数据存储, 依据key分区下可以很好的适应增长的数据量. 可以参考: php文件缓存设计

```
<?php
class FileCache implements ICache {
    function get( $_key ) {
        //get实现
    }
 
    function set( $_key, $_value, $_time ) {
        //set实现
    }
}
?>
```

(2). 基于memcached或者redis等NoSQL数据库(应该不是叫你设计一个类似于memcached类的东西):   

```
<?php
//基于redis实现缓存
class RedisCache implements ICache {
    function get( $_key ) {
 
    }
 
    function set( $_key, $_val, $_time ) {
 
    }
}
?>
```

选择一个memcached或者redis的php客户端, 实现上面的接口即可.



3. 来一个缓存工厂:

用于方便创建各种缓存, 从API层隐藏内部实现:

```
<?php
define('_CACHE_HOME_', dirname(__FILE__));
class CacheFactory {
 
    private static $_classes = NULL;
     
    public static function create( $_class, $_args = NULL ) {
        if ( self::$_classes == NULL ) {
            self::$_classes = array();
            //require the common interface
            require _CACHE_HOME_.'/ICache.class.php';
        }
         
        $_class = ucfirst( $_class ).'Cache';
        if ( ! isset( self::$_classes[$_class] ) ) {
            require _CACHE_HOME_.'/'.$_class.'.class.php';
            self::$_classes[$_class] = true;
        }
         
        //return the newly created object
        return new $_class($_args);
     }
}
?>
```

4. 缓存调用:

```
<?php
require '缓存工厂类';
 
$_cfg = array();   //缓存配置信息, 例如: redis的配置信息
//或者文件缓存的存放目录信息等等...
$_cache = CacheFactory::create('redis', $_cfg);
 
//添加缓存
$_cache->set("foo", "value", 3600);
 
//获取缓存
$_cache->get($_key);
?>
```

5. 考虑扩展/分布式:

(1). 对于文件缓存的实现, 本人的博客中讲述到分区存储(分文件夹)

(2). 对于redis或者memcached: 

->普通hash散列 (分布效果不可控制, 依据数据和hash算法而定分布情况, 而且不可增加机器)

->一致性hash散列 (哈哈, 这个是最常用的了, 可以随意的增加服务器, 增加服务器后缓存的命中率会得到最小的下降).

这个只要在redis.class.php的缓存类中实现一致性hash算法, 在CacheFactory::create("redis", $_cfg)的第二个参数$_cfg中将全部的缓存服务器配置信息传递进去就可以了.

```
<?php
$_cfg = array(
    array(
        'host' => '',   //主机地址
        'port' => '',   //主机端口
        //其他信息
    ),    //缓存服务器1
    array(),    //缓存服务器2
 
    //更多缓存服务器.
);
$_cache = CacheFactory::create('redis', $_cfg);
 
//API调用
?>
```
这一步的难点: 一致性hash的实现, 其实也挺简单的, google 一下有很多的.



优点: 可扩展性好, 调用方便, 调用代码只要更改一处就可以切换缓存的存储介质

