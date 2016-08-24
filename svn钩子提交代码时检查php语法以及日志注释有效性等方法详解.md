## SVN钩子提交代码时检查PHP语法以及日志注释有效性等方法详解

### 一. php-svn-hook

subversion本身有很好的扩展性，用户可以通过钩子（hook）实现一些自定义的功能。

所谓钩子实际上是一种事件机制，当系统执行到某个特殊事件时，会触发我们预定义的动作，这样的特殊事件在subversion里有很多，默认有如下模板可供选择：

```
shell> ls /path/to/repository/hooks
post-commit.tmpl
post-lock.tmpl
post-revprop-change.tmpl
post-unlock.tmpl
pre-commit.tmpl
pre-lock.tmpl
pre-revprop-change.tmpl
pre-unlock.tmpl
start-commit.tmpl
```

其中最常用的是pre-commit.tmpl 和 post-commit.tmpl，也就是提交前后的钩子.

** 注意： 
默认的是模板文件，正式使用的时候一定要 mv xxxx.tmpl  xxxx 为脚本，然后赋予可执行权限，钩子脚本才能正常工作 **


下面以pre-commit为例来说明一下如何使用SVN钩子。

##### * SVN钩子典型使用场景：

假设有一个PHP项目使用Subversion做版本控制，使用中发现了一些问题，比如程序员不写日志，或者提交的文件有BOM，或者提交的文件有语法错误，或者提交的文件不符合编码规范等等


##### * 问题场景

1. 提交代码时候，不能写空日志或者日志字符数不能少于N个，则拒绝提交
2. 提交代码时候，如果PHP文件语法有错误，则拒绝提交


##### * 解决方法：

利用pre-commit钩子来解决
解决工具：

一款老外基于PHP编写的工具 [php-svn-hook](http://jeanmonod.github.io/php-svn-hook/)

如果你shell玩的溜，那就在pre-commit脚本里直接编写对应的事件脚本即可，博主对PHP熟，所以很多类似场景都是穿透SHELL来调用PHP脚本达到同样的目的，效率很高，所以根据自己的情况选择就是啦，不纠结~

##### * 重点介绍php-svn-hook：

1. 此软件包面向对象开发
2. 先进的插件思想
3. 非常灵活，任意扩展定制所需的插件
4. 完美融合subversion钩子机制
5. 官方有详细说明文档

##### * 部署说明：

1.  默认的软件包只提供了日志注释插件
2.  默认的软件包有些BUG，之后我会在其他地方分享出博主修订后的代码，到时群里分享地址给大家
3.  软件包目录结构如下：4.  
	
	![image](http://www.blogdaren.com/content/uploadfile/201509/daa71441689335.jpg)
4.  博主扩展了下提交时的检查PHP语法插件，代码如下：
	
	![image](http://www.blogdaren.com/content/uploadfile/201509/98761441689554.jpg)
5.  接下来就是编写pre-commit钩子脚本啦，很简单，看图：
     ```
     vi  /path/to/svn_store/project/hooks/pre_commit
     ```
	
	![image](http://www.blogdaren.com/content/uploadfile/201509/86fd1441690358.jpg)
6. 效果图演示：

	![image](http://www.blogdaren.com/content/uploadfile/201509/28c01441690542.jpg)


### 二. pre-commit钩子内容(shell写法)

```
#!/bin/sh

REPOS="$1"
TXN="$2"
SVNLOOK=/usr/bin/svnlook
PHP="/usr/local/php5.5.1/bin/php"

# check that logmessage contains at least 5 alphanumeric characters
LOGMSG=`$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c`
if [ "$LOGMSG" -lt 5 ];
then
  echo -e "###### Empty log message not allowed. Commit aborted! ######" 1>&2
  exit 1
fi

# check PHP Syntax error
FILES=$($SVNLOOK changed -t "$TXN" "$REPOS" | awk '/^[AU]/ {print $NF}')
HASERROR=0

for FILE in $FILES; do
    CONTENT=$($SVNLOOK cat -t "$TXN" "$REPOS" "$FILE")

    if echo "$CONTENT" | grep -q $'var_dump'; then
        echo "######Debug Code found :$FILE#########" 1>&2
        echo "Please remove var_dump from $FILE" 1>&2
        HASERROR=1
    fi
    if [[ "$FILE" =~ \.(php)$ ]]; then
        MESSAGE=$(echo "$CONTENT" | $PHP -l 2>&1)

        if [ $? -ne 0 ]; then
            echo "###### PHP Error found: $FILE #########" 1>&2
            echo "$MESSAGE"  1>&2
            HASERROR=1
        fi
    fi
done
if [[ $HASERROR -eq 1 ]]; then
    exit 1
fi
```