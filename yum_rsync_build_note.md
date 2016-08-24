# yum 安装 rsync 参数

1. 安装rsync
	
		yum install rsync -y

2. 在远程服务器上创建配置文件和密码文件.

		mkdir /etc/rsyncd/
		cd /etc/rsyncd/
		vi rsyncd.conf

3. 然后粘贴下面的配置

		pid file = /var/run/rsyncd.pid
		port = 873
		uid = root
		gid = root
		use chroot = yes
		read only = yes
		max connections = 5 #This will give you a separate log file
		#log file = /var/log/rsync.log
		log format = %t %a %m %f %b
		syslog facility = local3
		timeout = 300
		[gosafe] #模块名,用来标识,我猜可以建立多个.
		path = /home/wwwroot/  #这个是你要下载的网站所在路径
		list=yes
		ignore errors
		auth users = gosafe #这个是下载用户
		secrets file = /etc/rsyncd/rsyncd.secrets #这个是用户密码文件
		comment = linuxsir home
		exclude = tmp/

4.	创建用户密码文件

		vi rsyncd.secrets
		
	键入 `gosafe：gosafepwd`
	
		chmod 600 rsyncd.secrets

5.	启动远程服务器的rsync服务.

		/usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf
 
6. 测试：本地运行
		
		rsync -avzP gosafe@IP::gosafe /backup
		
		
##########################################################		
		

# rsync配置(详细版)
#### 服务端配置
1. 通过yum安装
2. 确认是否安装了xinetd，未安装，通过yum安装
3. 在firewall中开放rsync需要的端口，修改/etc/sysconfig/iptables  
	
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 873 -j ACCEPT  
		  
4. 增加配置文件 /etc/rsyncd.conf
  
		touch /etc/rsyncd.conf  
		chmod 600 /etc/rsyncd.conf  
5. 编辑rsyncd.conf
  
		uid = nobody  
		gid = nobody  
		user chroot = no  
		max connections = 200  
		timeout = 600  
		pid file = /var/run/rsyncd.pid  
		lock file = /var/run/rsyncd.lock  
		motd file = /etc/rsyncd.motd   
		log file = /var/log/rsyncd.log  
		[data]  
		path = /data  
		ignore errors  
		read only = yes  
		list = no  
		hosts allow = 10.8.83.0/255.255.255.0 10.8.100.0/255.255.255.0  
		auth users = rsync  
		secrets file =/etc/rsyncd.pwd  

6. 创建欢迎文件 /etc/rsyncd.motd
 
		touch /etc/rsyncd.motd  
		chmod 600 /etc/rsyncd.motd  

7. 创建密码文件 /etc/rsyncd.pwd
  
		touch /etc/rsyncd.pwd  
		chmod 600 /etc/rsyncd.pwd  
8. 编辑密码文件/etc/rsyncd.pwd
  
		rsync:rsync  

9. 编辑启动文件/etc/xinetd.d/rsync中的disable改为no
 
		service rsync  
		{  
        		disable = no  
        		socket_type     = stream  
        		wait            = no  
        		user            = root  
        		server          = /usr/bin/rsync  
        		server_args     = --daemon  
        		log_on_failure  += USERID  
		}  
 
10. 配置成随系统启动
  
		chkconfig rsync on  

11. 启动
 
		service xinetd restart  
		/etc/rc.d/init.d/xinetd reload  
 
 
12. 测试是否已经正常启动
  
		netstat -a | grep rsync  

13. 直接检查文件
 
	rsync --list-only rsync@10.8.100.100::data  
 
#### 客户端使用方法
1. 安装，如果不做为服务器，yum安装即可
2. 如果不想输入密码，生成密码文件，可任意名 
  
		touch /etc/rsyncd.pwd  
		chmod 600 /etc/rsyncd.pwd  
		
3. 使用：
  
		#将远程数据拉取回来  
		rsync -vzrtopg --delete rsync@10.8.100.100::data /data --password-file=/etc/rsyncd.pwd  
		将本地数据备份至远程rsync Server  
		rsync -vzrtopg --delete /data rsync@10.8.100.100::data --password-file=/etc/rsyncd.pwd  
  
		#--delete表示完全一致，会执行目标文件夹的删除，  

4. 使用cron实现定时操作

（1）生成脚本 /usr/backup.sh  
		
		touch /usr/backup.sh  
		chmod /usr/backup.sh  
		
（2）增加内容
  
		#!/bin/sh  
		rsync -vzrtopg --delete rsync@10.8.100.100::data /data --password-file=/etc/rsyncd.pwd  

（3）加入至cron
  
		crontab -e  
  
		#添加  
		* * * * * sh /usr/backup.sh     /*表示每一分钟执行一次/usr/backup.sh文件  
		
（4）重启cron
  
		/etc/init.d/crond restart  