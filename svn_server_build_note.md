# SVN 多项目管理（强烈建议每个项目建一个库）
Subversion的目录结构是很自由的，所有的规划都必须是你自己规定，考虑一个 subversion仓库的目录树，你可以把任何一个目录认定为一个项目，你可以只checkout这个目录下的所有文件进行编码，跟CVS不同，CVS显式指定一个个module。所以你可以在一个仓库内保存 多个项目，也可以一个仓库保存一个项目而使用多个仓库。我个人比较喜欢第二种，因为 Subversion的每次commit都会导致整个仓库 版本号增加一个，会使得 多个项目的 版本号出现断层。而且如果 多个项目参与人不同，就必须使用apache2进行细粒度的权限控制，不是太方便。一个仓库一个项目，显得更优雅一些。

# 安装subversion



# 配置SVN SERVER

在server端，新建一个目录用来存放所有的仓库。比如/Data/app/svnrepos。然后在这个目录下建立每个项目独立
	
	svnadmin create "/Data/app/svnrepos/rolex"
	svnadmin create "/Data/app/svnrepos/omega"

使用 `svnserve -d -r "/Data/app/svnrepos"` 启动。这样你的项目的url是：

	svn://IP/rolex
	svn://IP/omega

在客户端新建一个目录，作为import的内容，比如/tmp/svnimport/svn_tpl，然后在里面建立

	code目录
		branches，tags，trunk，docs子目录，

把你需要源代码管理的项目放入trunk目录，注意删除垃圾文件。在/tmp/svnimport/svn_tpl上点击Import...，选择url为 svn://IP/rolex，导入。你可以使用仓库浏览器查看导入的效果。

需要工作时，新建一个目录比如/tmp/svnclient/rolex/trunk，然后在trunk上checkout出svn://IP/rolex/trunk上的内容。

	Linux命令行：
	svn import svnimport_tpl/ svn://svn.fangfull.com:3691/qianjins_website/ -m'initial import'

#### 备注: 多项目管理时需修改每个项目的svnserve.conf文件
修改svnrepos/rolex/conf/svnserve.conf中authz与password路径:

	# 开启
	anon-access = none     (Note:read修改为none)
	auth-access = write
	# 修改
	password-db = ../../conf/passwd
	authz-db = ../../conf/authz


修改通用授权配置文件../../conf/authz

	# For hifang
	[ares_hifang:/]
	@zhAdminGroup = rw
	[ares_hifang:/code]
	@zhPHP = rw
	[ares_hifang:/code/trunk]
	@zhPHP = rw

增加hooks
	
	cp pre-commit ares_hifang/hooks/

重启svnserve服务

	# 查找
	ps -ef | grep svnserve
	# 停用
	ps -ef |grep svnserve |awk '{print $2}'|xargs kill -9
	# 启动
	svnserve -d -r /Data/app/svnrepos --listen-port 3691


# 分支开发

创建分支

	svn copy svn://svn.fangfull.com:3691/qianjins_website/trunk svn://svn.fangfull.com:3691/qianjins_website/branches/release -m"init release branch"


# 端口配置
svn服务器默认使用3690端口号，svn要使用非默认端口，可以在svnserve后面加一个 --listen-port 21 来修改svn使用的端口号，

操作如下：在命令提示符下输入：

	svnserve -d -r /home/declan/svnproject --listen-port 21

注：红色加粗部分为SVN根目录

同时，还可以为同一个svn服务器上不同的svn项目设定不同的端口号，比如在declan目录下还建有另一个项目，名为 svntest，那么可以启动

	svnserve -d -r /home/declan/svntest --listen-port 3690

则svntest项目监听3690（svn默认）端口号，这样在使用 netstat -ntlp 进程查看时会查看到另个svnserver，而在客户端，默认连接为3690端口，也可以在地址后加 “:21”，即 冒号+端口号 来设定访问端口

Example:
`svnserve -d -r /Data/app/svnrepos --listen-port 3691`

# 开放服务器端口

	vi /etc/sysconfig/iptables
加入3691端口开放规则

	-A INPUT -p tcp -m state --state NEW -m tcp --dport 3691 -j ACCEPT

重启服务

	service iptables restart


# 仓库迁移

假设将A仓库的数据转移到B仓库

	A位置：/svndata/A
	B位置：/svndata/B

**不能直接将A仓库重命名为B，或将A复制得到一个复本，再将复本命名为B**

需使用

	svnadmin dump & svnadmin load


1. 得到A仓库.dump文件A.dump
		
		svnadmin dump /svndata/A > A.dump

2. 创建B仓库(如果B不存在)

		svnadmin create /svndata/B

3. 将A.dump 加载到B仓库
		svnadmin load /svndata/B < A.dump


**Note**: 

使用dump&load方法只将A管理的文件复制到B中，但是A的配置信息（密码等）没有被复制到B中



# More...SVN实战

1. 服务器的确定

2. 配置管理工具的确定（SVN）               

3. 建版本库的根目录，如下图所示，svnroot根目录下有project1和project2两个库

   ![image](http://img.my.csdn.net/uploads/201210/23/1350971763_9191.jpg)

4. 创建第一个项目project1，命令：svnadmin create E:\svnroot\project1

5. 创建第二个项目project2，命令：svnadmin create E:\svnroot\project2

6. 为了便于管理，将所有版本库的密码和权限设置在同一个文件下面，操作步骤如下：

  6.1 取出project1下面conf文件夹下的authz和passwd两个文件到svnroot根目录下面

   6.2 修改每个版本库目录conf文件夹下面的svnserve.conf文件， 将

	anon-access = read ，#auth-access = write ，# password-db = passwd，#authz-db = authz 
	
	修改为：

     anon-access = none ，auth-access= write，password-db = ../../passwd，authz-db = ../../authz

   (password-db = ../../passwd，authz-db = ../../authz代表相对路径而非绝对路径)

7. 定义一下几个角色用来测试

  7.1 配置管理员（svnadmin），用来管理整个库

  7.2 项目经理（manage），用来相关管理文档

  7.3 开发人员 （dev），测试开发是否正常

8. 下面添加角色

   打开svnroot目录下的passwd文件，创建方法是在[user]下面添加 username = passwd，记得“=”前后的空格，如下图：（svnadmin控制所有项目，统一管理）


9. 为角色分配权限：假设（quxin是project1的项目经理，huzhixin是project2的项目经理，dev1、dev2是project1的开发人员，dev3、dev4是project2的开发人员，test1是project1的测试人员，test2是project2的测试人员）

库目录及具体权限如下图所示：

库目录                        权限分组：

 ![image](http://img.my.csdn.net/uploads/201210/23/1350971775_4187.jpg)  ![image](http://img.my.csdn.net/uploads/201210/23/1350971780_8205.jpg)           

具体权限：（根目录下，svnadmin拥有所有权限，其他人只有读权限，要设置子目录权限，

需设置子目录上级的权限方可，设置个别文件权限如下：）

![image](http://img.my.csdn.net/uploads/201210/23/1350971785_8482.jpg)

启动SVN服务，可在dos命令里启动，也可把SVN服务安装在服务管理里面

把服务在DOS命令里启动方法：svnserve –d –r E:\svnroot

如若把服务安装在服务管理里面，简单的办法，下载一个SVNService.exe文件，放到subversion安装目录的bin文件夹下面，然后在dos命令里运行，

运行方法如下： SVNService –install –d –rE:\svnroot

10. 安装客户端，连接服务器到要访问的库，假如访问project1：svn://172.16.26.28/project1 ，用同样的方法访问project2，依次类推到更多的版本库项目。

11. 工具

SVN 服务器端：Subversion 1.5 ，客户端 TrotoiseSVN 1.5

12. 下载地址：www.iusesvn.com ，你需要注册方可进入下载去下载相关版本的工具。

13. 相关角色的定义

配置管理员 CM

整个配置管理库由配置管理员管理。配置管理员负责分配和修改其他成员的权限，要维护所有目录和配置项。


项目经理

开发经理在本项目中负责主导完成需求分析和系统总体设计，对项目的总体进度负责。开发经理拥有对管理类文档的读取权限，可以对项目类文档进行读写操作；


开发组长

开发组长对本小组的工作负有组织和管理任务，同时开发组长也需要承担一定的开发任务。开发组长对管理类文档有读取权限，对本组负责的模块有读取权限，对自己负责的模块有读写的权限；

 
开发工程师
     开发工程师完成具体的开发任务，对自己负责的模块目录有读写权限，对管理类文档有读取权限；


测试组长

测试组长负责组织测试，给出测试计划和测试方案，并核定测试报告。测试组长对所有目录都有读取权限，对测试目录有读写权限；


测试工程师

测试工程师负责完成测试工作，包括测试用例开发和测试执行，测试报告编写。测试工程师对自己负责的模块有读取权限，对测试用例目录有读写权限。

 
QA工程师
    QA工程师拥有对所有目录的读取权限，拥有对QA类文档目录的读写权限。


高层经理

高层经理负责部门及各个项目的协调工作。对部门公共库PUB有读写权限，对各项目有读取权限

# 参考资料

[SVN:多版本库环境的搭建](http://blog.csdn.net/subkiller/article/details/8102566)

[Linux下安装配置SVN独立服务器svnserve](http://www.ttlsa.com/svn/install-svnserve-on-linux/)

[使用svn——项目的目录布局](http://www.cnitblog.com/stomic/archive/2008/03/17/41043.html)


[使用svn钩子(hook)来检查PHP提交时的语法](http://blog.shuaizhu.com/2016/03/25/%E4%BD%BF%E7%94%A8svn%E9%92%A9%E5%AD%90hook%E6%9D%A5%E6%A3%80%E6%9F%A5php%E6%8F%90%E4%BA%A4%E6%97%B6%E7%9A%84%E8%AF%AD%E6%B3%95/)