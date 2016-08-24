# Linux“Bash”漏洞大爆发 360发布测试方法

9月25日，国外安全专家发现Linux系统中一个频繁使用的片段“Bash”存在漏洞，影响目前主流的Linux系统。利用该漏洞，黑客可以远程窃取服务器上的信息，并进一步控制服务器。360安全中心进一步分析发现，该漏洞会被黑客制作成自动化攻击工具。

该漏洞编号为CVE-2014-6271，主要存在于bash 1.14 - 4.3版本中，受影响的linux系统包括：Red Hat企业Linux (versions 4 ~7) 、Fedora distribution、CentOS (versions 5 ~7)、Ubuntu 10.04 LTS,12.04 LTS和14.04 LTS、Debian等。

经过360安全中心进一步分析发现，该“Bash”漏洞暂未对安卓系统造成影响，但会被黑客制作成自动化攻击工具，从而针对企业网络设备、安全产品等的远程控制台、串口控制台发动规模化攻击。

目前，360安全中心应发布“Bash”漏洞测试方法，同时提醒广大网站和企业及时更新服务务器安全补丁，避免造成重大危险。

“Bash”漏洞测试方法

1）、本地测试

env x='() { :;}; echo vulnerable' bash -c "echo this is a test"

![image](http://www.cctime.com/upLoadFile/2014/9/25/2014925183913734.jpg)

2）、远程测试

首先用BASH写一个CGI 

root@kali:/usr/lib/cgi-bin# cat bug.sh 

	#!/bin/bash

	echo "Content-type: text/html"

	echo ""

	echo '<html>'

	echo '<head>'

	echo '<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">'

	echo '<title>PoC</title>'

	echo '</head>'

	echo '<body>'

	echo '<pre>'

	/usr/bin/env

	echo '</pre>'

	echo '</body>'

	echo '</html>'

	exit 0

放到/usr/lib/cgi-bin里，然后用curl访问

![image](http://www.cctime.com/upLoadFile/2014/9/25/2014925183913580.jpg)

能打印出环境变量了。说明能够正常访问了。下面反弹一个SHELL

![image](http://www.cctime.com/upLoadFile/2014/9/25/2014925183913621.jpg)

访问看结果：

![image](http://www.cctime.com/upLoadFile/2014/9/25/2014925183913360.jpg)


# 阿里云[分享]Linux Bash严重漏洞修复紧急通知

尊敬的阿里云ECS用户： 
           您好，日前Linux官方内置Bash中新发现一个非常严重安全漏洞（漏洞参考https://access.redhat.com/security/cve/CVE-2014-6271  ），黑客可以利用该Bash漏洞完全控制目标系统并发起攻击，** 为了避免您Linux服务器受影响，建议您尽快完成漏洞修补，修复方法如下，请了解！ **
 
【已确认被成功利用的软件及系统】  
所有安装GNU bash 版本小于或者等于4.3的Linux操作系统。  
 
 
【漏洞描述】  
该漏洞源于你调用的bash shell之前创建的特殊的环境变量，这些变量可以包含代码，同时会被bash执行。  
 
 
【漏洞检测方法】  
 
** 修复前 **
输出:    
	
	vulnerable   
	this is a test    
 
 
** 使用修补方案修复后 **
	
	bash: warning: x: ignoring function definition attempt 
	bash: error importing function definition for `x' 
	this is a test 

特别提示：该修复不会有任何影响，如果您的脚本使用以上方式定义环境变量，修复后您的脚本执行会报错。 
 
 
【建议修补方案 】  
 
请您根据Linux版本选择您需要修复的命令， 为了防止意外情况发生，建议您执行命令前先对Linux服务器系统盘打个快照，如果万一出现升级影响您服务器使用情况，可以通过回滚系统盘快照解决。  
 
**centos: **

	yum -y update bash 
 
**ubuntu: **
14.04 64bit 

	wget http://mirrors.aliyun.com/fix_stuff/bash_4.3-7ubuntu1.1_amd64.deb && dpkg -i bash_4.3-7ubuntu1.1_amd64.deb 
 
14.04 32bit 
	
	wget http://mirrors.aliyun.com/fix_stuff/bash_4.3-7ubuntu1.1_i386.deb && dpkg -i  bash_4.3-7ubuntu1.1_i386.deb 
 
 
12.04 64bit 
	
	wget http://mirrors.aliyun.com/fix_stuff/bash_4.2-2ubuntu2.2_amd64.deb && dpkg -i  bash_4.2-2ubuntu2.2_amd64.deb 
 
12.04 32bit 

	wget http://mirrors.aliyun.com/fix_stuff/bash_4.2-2ubuntu2.2_i386.deb && dpkg -i  bash_4.2-2ubuntu2.2_i386.deb 
 
10.10 64bit 

	wget http://mirrors.aliyun.com/fix_stuff/bash_4.1-2ubuntu3.1_amd64.deb && dpkg -i bash_4.1-2ubuntu3.1_amd64.deb 
 
10.10 32bit 
	
	wget http://mirrors.aliyun.com/fix_stuff/bash_4.1-2ubuntu3.1_i386.deb && dpkg -i bash_4.1-2ubuntu3.1_i386.deb 
 
 
** debian: **
7.5 64bit && 32bit 
	
	apt-get -y install --only-upgrade bash 
 
6.0.x 64bit 

	wget http://mirrors.aliyun.com/debian/pool/main/b/bash/bash_4.1-3%2bdeb6u1_amd64.deb &&  dpkg -i bash_4.1-3+deb6u1_amd64.deb 
 
6.0.x 32bit 

	wget http://mirrors.aliyun.com/debian/pool/main/b/bash/bash_4.1-3%2bdeb6u1_i386.deb &&  dpkg -i bash_4.1-3+deb6u1_i386.deb 
 
** opensuse: **
13.1 64bit 

	wget http://mirrors.aliyun.com/fix_stuff/bash-4.2-68.4.1.x86_64.rpm && rpm -Uvh bash-4.2-68.4.1.x86_64.rpm 
 
 
13.1 32bit 

	wget http://mirrors.aliyun.com/fix_stuff/bash-4.2-68.4.1.i586.rpm && rpm -Uvh bash-4.2-68.4.1.i586.rpm 
 
** aliyun linux: **
5.x 64bit 
	
	wget http://mirrors.aliyun.com/centos/5/updates/x86_64/RPMS/bash-3.2-33.el5.1.x86_64.rpm && rpm -Uvh bash-3.2-33.el5.1.x86_64.rpm 
 
5.x 32bit 

	wget http://mirrors.aliyun.com/centos/5/updates/i386/RPMS/bash-3.2-33.el5.1.i386.rpm && rpm -Uvh bash-3.2-33.el5.1.i386.rpm 
 
 
如果有问题，请随时提交工单或者电话联系我们。 
 
阿里云计算 

2014年9月25日  


