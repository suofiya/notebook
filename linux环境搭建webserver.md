## Linux环境搭建WebServer

1. 搭建LNMP环境
   * 拷贝所需软件包：
   			
   			cp lnmp_software.tgz /home/xxx
   	
   * 启动一键安装脚本 
   		
   			/bin/bash LNMP_install.sh
   			
   			
2. 搭建WebServer
   * 创建虚拟主机
  		
  			mkdir -p /var/www/vhosts/test.jikehaiwai.com/
  			
  		
   * 创建DocumentDir和创建访问LogDir
   			
   			cd /var/www/vhosts/test.jikehaiwai.com/
   			mkdir httpdocs
   			mkdir logs
   			
   	* 创建应用日志Dir
   		
   			mkdir -p /var/log/www/test.jikehaiwai.com/admin
   			mkdir -p /var/log/www/test.jikehaiwai.com/api
   			mkdir -p /var/log/www/test.jikehaiwai.com/mobile
   			mkdir -p /var/log/www/test.jikehaiwai.com/pc
   			
   			## 增加写入权限
   			chmod -R 777 /var/log/www/test.jikehaiwai.com
   			
   	* 创建nginx vhosts配置
   			
   			mkdir -p /Data/app/nginx/conf/vhosts
   			
   			## 增加nginx server配置 
   			vim test.jikehaiwai.com.conf
   			vim test.jikehaiwai.com-ssl.conf
   			
   			## nginx.conf引入虚拟主机配置
   			include vhosts/test.jikehaiwai.com.conf
   			
   			
   	* 发布代码
   			
   			## 切换至项目发布目录
   			cd /root/webroot_svncode/
   			
   			## 备份代码
   			/bin/sh 1_publish_bak.sh hifang
   			
   			## 从SVN更新并push代码至remote server
   			/bin/sh 2_publish_code.sh hifang
   			
   			## 同步更新配置文件(需要开发确认文件内容)
   			/bin/sh 3_publish_conf.sh hifang
   			
   			
   			## 同步图片
   			
   			
   			
   			
   					 			
   	