# mysql数据库连接慢的原因

### 问题

有两个myslq数据库，分别装在了两个服务器上，即210&249；
其他服务器上连接数据库，发现249的数据库连接很慢，而210正常；
结果是：249数据库出了问题。

### 尝试的解决办法：

1.重启apache （在/usr/local/apache/bin 下 apachectl -k restart) 不管用；

2.重启数据库所在服务器（在Linux下输入reboot）不管用；

3.在网上搜帖子“连接mysql数据库速度很慢的原因，发现mysql就会试图去解析来访问的机器的domain name,在经历一段时间后才取出数据．

在网上找了很久才发现，一个参数：`skip-name-resolve`, 在mysql的配置文件`my.cnf`中,在`[mysqld]`下面加上这个配置就可以了．前不久断网时登录内类系统后台奇慢的问题，也是由这个原因引起的。

	首先找到mysql的配置文件my.cnf，在/etc/下，按照帖子的方法，修改【mysqld】，加上了skip-name-resolve；
	然后重启MySQL，先关闭：
	在/bin/下 mysqladmin -uroot -p密码 shutdown， 
	ps aux|grep mysql 观察mysql是否被关闭，
	启动：mysqld_safe &；
	重启过后，管用，访问速度很快~~

kill -9 `ps -ef|grep mysql|grep -v grep||awk '{print $2}'`
ps aux|grep mysql
/Data/app/mysql5.1.57/bin/mysqladmin -uroot -pMhxzKhl shutdown;
/Data/app/mysql5.1.57/bin/mysqld_safe --defaults-file=/Data/app/mysql5.1.57/my.cnf &

#### 这里推荐安全的重启方法

	$mysql_dir/bin/mysqladmin -u root -p shutdown
	$mysql_dir/bin/mysqld_safeß &

mysqladmin和mysqld_safe位于Mysql安装目录的bin目录下，很容易找到的。