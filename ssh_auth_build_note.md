# 快速方式
 
	ssh-keygen 	   #会在你用户的home下的.ssh文件夹下生成id_rsa和id_rsa.pub文件，生成的时候不选passphrase
	ssh-copy-id -i ~/.ssh/id_rsa.pub userb@host    #这个会把你的id_rsa.pub公钥copy到目标主机userb用户home下的.ssh，并自动改名为authorized_keys
	ssh host -l userb     #以userb登录远程主机

	
Note:
	
	id_ras私钥机器去连接authorized_keys公钥机器
 
 
这样就行了，不需要两台机器有相同的用户，如果你用了selinux请检查一下相关设置，另外在/etc/ssh_cinfig下有个是否是用密码的选项，改动一下看看。

另外你可以描述一下系统提示的是输入什么密码，用passphrase的话是提示输入key的密码，不是类似"userb@host"这样的提示
 
# CentOS 配置双机SSH信任
 
 一、实现原理
 
使用一种被称为"公私钥"认证的方式来进行ssh登录。"公私钥"认证方式简单的解释是：
首先在客户端上创建一对公私钥（公钥文件：~/.ssh/id_rsa.pub；私钥文件：~/.ssh/id_rsa），然后把公钥放到服务器上（~/.ssh/authorized_keys），自己保留好私钥。当ssh登录时，ssh程序会发送私钥去和服务器上的公钥做匹配。如果匹配成功就可以登录了。

二、实验环境

A机：TS-DEV/10.0.0.163
B机：CS-DEV/10.0.0.188

三、Linux/Unix双机建立信任

3.1 在A机生成证书
在A机root用户下执行ssh-keygen命令，在需要输入的地方，直接回车，生成建立安全信任关系的证书。

	ssh-keygen  -t  rsa
 
 ![image](http://www.centoscn.com/uploads/allimg/130820/133U4A21-0.jpg)
 
注意：在程序提示输入passphrase时直接输入回车，表示无证书密码。
　　　上述命令将生成私钥证书id_rsa和公钥证书id_rsa.pub，存放在用户家目录的.ssh子目录中。
　　　
3.2 查看~/.ssh生成密钥的文件

	cd ~/.ssh
	ll

![image](http://www.centoscn.com/uploads/allimg/130820/133U415G-1.jpg)

3.3 A对B建立信任关系
将公钥证书id_rsa.pub复制到机器B的root家目录的.ssh子目录中，同时将文件名更换为authorized_keys，此时需要输入B机的root用户密码（还未建立信任关系）。建立了客户端到服务器端的信任关系后，客户端就可以不用再输入密码，就可以从服务器端拷贝数据了。

	scp -r id_rsa.pub 10.0.0.188:/root/.ssh/authorized_keys

![image](http://www.centoscn.com/uploads/allimg/130820/133U45212-2.jpg)

3.4 B对A建立信任关系
在B机上执行同样的操作，建立B对A的信任关系。

	ssh-keygen -t rsa
 
![image](http://www.centoscn.com/uploads/allimg/130820/133U41061-3.jpg)
 
	cd ~/.ssh/
	ll

![image](http://www.centoscn.com/uploads/allimg/130820/133U45B9-4.jpg)

	scp -r id_rsa.pub 10.0.0.163:/root/.ssh/authorized_keys

![image](http://www.centoscn.com/uploads/allimg/130820/133U44256-5.jpg)

四、测试

在A机上：

	scp -r 10201_database_linux_x86_64.cpio 10.0.0.188:/tmp/david/

![image](http://www.centoscn.com/uploads/allimg/130820/133U41421-6.jpg)

在B机上：

![image](http://www.centoscn.com/uploads/allimg/130820/133U44207-7.jpg)

注：如果想让B，C同时可以scp不输入密码，传输A中的数据；
则要把B、C的公钥都给 A；
操作步骤：把两机器的id_rsa.pub中的数据都拷贝到A的/root/.ssh/authorized_keys文件中，一行表示一条；

五、远程执行命令

命令格式：ssh 远程用户名@远程主机IP地址 '远程命令或者脚本'

	ssh root@10.0.0.188 'hostname'

![image](http://www.centoscn.com/uploads/allimg/130820/133U46133-8.jpg)

上述命令执行后，终端输出的是对端主机的主机名，而不是当前登录的主机的主机名。说明 hostname 这个命令其实是在对端主机上运行的。
双机信任关系已经建立！
