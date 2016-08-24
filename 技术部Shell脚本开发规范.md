# 一、命名规范

## 1.1 文件/模块

* Shell脚本使用统一后缀 .sh
	
		Example：foo.sh

	`通过.sh后缀可以直观的反映出文件类型，并且能跟二进制程序区别开。`
		
* 模块命名应该代表其特性和功能，禁止使用个人名字缩写等形式命名

* 配置文件使用{模块名}_conf.sh命名规则
	
		Example：foo_conf.sh


## 1.2 函数

* 函数名不做强制要求，但整个模块内部应该统一命名风格。

	推荐命名风格：
		
	`1）首字母大写（CreateFile）`
	
	`2）下划线分隔（create_file）`


* 在多个模块互相调用的项目里，应该在每个函数名称前加上模块缩写（opt_create_file 或 optCreateFile）

		Example：db跟log模块分别有create_file函数，当main.sh要一起引用两个模块时，就会出现不必要的麻烦。可以通过为函数名加模块前缀解决这类问题db_create_file，log_create_log，也能在后续改动或复查main.sh时能更直观的知道使用的是某个模块的create_file.
		
## 1.3 全局变量

全局变量使用全小写加下划线并使用g_前缀

	Example：g_foo_path
	直观看出某个变量是否是全局变量，并能在使用这类变量的时候更加慎重。

## 1.4 全局常量

全局常量命名使用全大写加下划线
	
	Example：MAX_BUF
	全大写的考虑是区别变量、函数命名。同C语言宏定义。


## 1.5 临时文件

临时文件命名使用模块名加tmp中缀加pid后缀

	Example： module_name.tmp.${pid}
	模块名标明临时文件由谁产生，tmp中缀标明该文件为临时文件，后缀的pid用来避免临时文件重名，并且可以通过pid确认当前临时文件是否是当前脚本进程所产生。

# 二．注释规范
2.1 文件/模块说明

说明模块主要用途，版本信息，输入输出文件，依赖工具及其版本信息, 前后流程脚本(可选)


	Example：
	##! @TODO:		url analyse
	##! @VERSION:	1.0
	##! @AUTHOR:	MM;BB
	##! @FILEIN:		data/url.crawl 	##!		由dedup_crawl.sh生成, 格式为...
	##! @FILEOUT:	result/GOOD_GRP 	##!		通过检测的无问题alias组 	##! @FILEOUT:	result/WRONG_GRP 	##!		未通过检测的有问题alias组	
	##! @DEP:		wget 1.10.2
	##! @DEP:		lftp 3.0.6
	##! @PREV:		dedup_crawl.sh
	##! @NEXT:		dedup_update.sh


* 采用##！@注释前缀是让以后文档化工作更加方便
* TODO应该简单介绍该模块完成的功能，比如URL分析
* VERSION应该在每次发生变更时应该修改
* AUTHOR应该包含作者和修改者
* FILEIN 标明了本模块使用的输入文件，可以注释该输入文件是由某个程序生成，以及格式
* FILEOUT标明了本模块使用的输出文件，可以注释该输出文件的用途，以及格式
* DEP 包含了脚本需要依赖特殊版本工具的清单，对于通用工具或者使用的功能在该工具各版本之间无差异的不用做此声明。 （可选）
* PREV 里注明流程的上一个脚本（可选）
* NEXT 里注明流程的下一个脚本（可选）`

## 2.2 重要函数说明

对于重要函数，需说明函数用途，参数，返回值，作者，版本

	Example：
	##! @TODO: 	get hostname
	##! @AUTHOR: 	somebody
	##! @VERSION: 	1.0
	##! @IN:		$1 => ip
	##! @IN:		$2 => port
	##! @OUT:	0 => success; 1 => failure

* 采用##！@注释前缀是让以后文档化工作更加方便
* TODO应该简单介绍该函数完成的功能，比如获取主机名
* VERSION应该在每次发生变更时应该修改
* AUTHOR应该包含作者和修改者
* IN提供函数的参数和对应意义，格式为每个参数一行，每行为 $1 => ip这样的格式，该注释说明第一个参数是IP地址。Shell的函数使用的是$n这样的默认参数，不加注释的话在函数较长或者参数较多时将很难区分各个参数代表的意义。当然也可以在函数起始使用local ip=$1把参数赋值到命名更有意义的变量里
* OUT 提供该函数的返回值和对应意义， 0 => success 这样的格式，该注释说明返回0表示函数执行成功。

## 2.3 特殊说明
对代码逻辑里出现的直接量做注释。(尽可能使用常量来替代直接量，参见3.4节)

	Example：
	for ((i = 0; i < 100; i++))		# max port: 100

	对于直接量做注释在某些情况下是不必要的，这条规范主要是让大家在使用直接量的时候能更多的考虑一下是否可以使用常量来替换直接量，达到更佳的阅读性和可维护性。对于没必要使用常量的场合，对直接量做注释能更清楚的让大家知道它是从哪里冒出来的。一个100大家可能有不同的理解，是100分？100块？还是100个人？

# 三．代码风格

## 3.1 代码框架

* 解释器声明				#!/bin/bash
* 模块说明				参见2.1节
* 配置文件/库函数的引用	source foo_conf.sh
* 全局常量，变量定义		参见1.3节
* 子函数说明				参见2.2 节
* 子函数
* 重复e和f
* 主过程注释（起分隔作用）如果是模块，则该节不存在
* 主过程					如果是模块，则该节不存在

### 3.1.1 主过程只实现程序主干，功能实现应该封装在子函数里。

### 3.1.2 对于能独立执行的脚本需要有usage和version函数，可以输出脚本用法和版本信息。


	Example:
	#!/bin/bash

	##! @TODO:	hello world
	##! @VERSION:	1.0
	##! @AUTHOR:	wengjianyi
	##! @FILEIN:	data/sth_in
	##! @FILEOUT:	result/sth_out
	##! @PREV:	foo.sh
	##! @NEXT:	bar.sh

	#------------------- library ------------------
	source lib/log.sh
	source lib/color_print.sh

	#--------------- global variable --------------
	typeset -r PROGRAM_NAME="hello world"
	typeset -r VERSION="1.0"

	g_pid=0
	g_level=0

	#------------------ function ------------------
	function usage()
	{
    	:
    	return 0
	}

	function version()
	{
    	:
    	return 0
	}

	##! @TODO:       show message	
	##! @AUTHOR:     somebody
	##! @VERSION:    1.0
	##! @IN:         $1 => msg
	##! @OUT:        0 => success; 1 => failure
	function show_msg()
	{
    	local msg="$1"

    	echo "$msg"
    	return 0
	}

	#------------------- main -------------------
	#...
	#...
	#...
	show_msg "hello world!"


## 3.2 语句风格
代码需要通过一定的缩进、空格、空行来提高代码的可读性。对于缩进、空行数量，换行、空格位置不做特别要求。

## 3.3 函数
* 函数定义时在函数名前加上function保留字

		Example:
		function foobar()
		{
			…
		}
		
		加function 保留字主要考虑方面定位函数（包括在vi里或者在外部grep出函数列表），

* 局部变量使用local限定符

		Example： local bar
		
		首先限定变量的作用域，其次是让作者再考虑一下哪些变量应该拥有什么作用域，跟全局变量g_一样。在函数内部使用的每个变量作者应该都要清楚的知道是该全局还是局部。

* 显示函数返回
	
		在函数的每个分支都包含显示的return语句，并跟上返回值。即使是不关心返回值的函数，也可能在后续调用时无意的去判断它的返回值并进行一系列动作，这种不必要的麻烦我们在一开始就应该注意，显示的写return语句并不会带来多少负担，相反的，它能让函数逻辑更加清晰和严谨。

## 3.4 常量
* 尽量使用常量替代直接量
	
		对于使用多次的直接量应该使用常量来替代，提供代码的可读性并可减低后续修改成本。对于一次使用的直接量可考虑采用直接量加注释的方式，参见 2.3节。

* 全局常量在文件起始处或在外部配置文件里定义
	
		参见3.1节

* 使用typedef –r或readonly定义不做强制要求
		
		Example：typeset –r MAX_BUF=10


# 四．错误处理
## 4.1 返回值处理

对于可能失败的命令就要对其返回值进行处理, 处理的方式这里不做规定。可以考虑做日志，重试，函数返回或脚本终止等动作。
	
	Example：grep “something” file > /dev/null 2>&1 || echo “sorry!”
