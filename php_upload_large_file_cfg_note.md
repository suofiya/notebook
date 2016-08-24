## 修改 PHP上传文件大小
#### nginx: 413 Request Entity Too Large的处理办法

开发环境：CentOS + Nginx + PHP + MySql + phpMyAdmin
在用 phpMyAdmin 进行 sql 数据库导入的时候，经常需要上传比较大的 sql 数据文件，而这时会常碰见 nginx报错：413 Request Entity Too Large。
解决此问题，根据上传数据文件的大小，你需要调节两个地方的参数配置：

1、php 默认上传文件大小限制为 2M，如果超出 2M 你需要修改 php 配置文件 php.ini 里面的参数
	
	post_max_size = 8M （表单提交的最大限制，此项不是限制上传单个文件的大小，而是针对整个表单提交的数据进行限制。）
	upload_max_filesize = 2M （上传的单个文件的最大限制）

需要保证 **post_max_size >= upload_max_filesize** ，也就是前者不小于后者。

修改之后一定要重启 php-fpm 。

2、除了修改 php 配置，你也需要修改nginx配置文件 nginx.conf 
打开 nginx 配置文件 nginx.conf，找到 http{} 段，在其中添加一行配置：

	client_max_body_size 8m;
	client_body_buffer_size 128k;

其中 8m 可以根据需要上传文件大小自行设定。

修改之后一定要重新载入 nginx （service nginx reload）。