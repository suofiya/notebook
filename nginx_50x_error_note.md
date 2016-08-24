# Nginx出现502和504错误提示的解决方案

## Note

一般优先排除**sql慢查询**导致cpu被耗尽等原因

## 正常处理方式

今天在测试机器上搞了个web论坛，discuz的，还算轻车熟路，直接用nginx+fast-cgi，搞了几下就运行一起来了，可是没多久就出现问题了，访问首页的时候，总是提示502或者504的，很是郁闷，潜意识里觉得是fast-cgi没搞正确，在度娘上搜了下果然如此，简单记录下吧。

Nginx 502 Bad Gateway的含义是请求的PHP-CGI已经执行，但是由于某种原因（一般是读取资源的问题）没有执行完毕而导致PHP-CGI进程终止。
Nginx 504 Gateway Time-out的含义是所请求的网关没有请求到，简单来说就是没有请求到可以执行的PHP-CGI。

一般来说Nginx 502 Bad Gateway和php-fpm.conf的设置有关，而Nginx 504 Gateway Time-out则是与nginx.conf的设置有关。
php-fpm.conf有两个至关重要的参数，一个是“max_children”,另一个是”request_terminate_timeout” ，但是这个值不是通用的，而是需要自己计算的。

解决办法如下：

1. 调整php-fpm.conf的相关设置：

		<value name="max_children">32</value>
		<value name="request_terminate_timeout">60s</value>

2. 调整nginx.conf的相关设置：

		fastcgi_connect_timeout 600; #set timeout
		fastcgi_send_timeout 600;
		fastcgi_read_timeout 600;
		fastcgi_buffer_size 256k;
		fastcgi_buffers 16 256k;   # 16*256,可以继续调大
		fastcgi_busy_buffers_size 512k;
		fastcgi_temp_file_write_size 512k;

    关于fastcgi buffer相关的日志，可以参考[这篇](http://www.phpvim.net/os/ubuntu/fastcgi_temp_error_and_nginx_buffer.html)。

3. 终级解决方案：
上面所示的解决方案只能临时解决问题，而如果网站的访问量确实非常非常大，而Nginx+FastCGI只能对处理瞬间或短时间内的高并发有很好的效果，所以目前唯一的终极解决方案是：定时平滑重启php-cgi。shell:


		#vi /home/www/scripts/php-fpm.sh
		#!/bin/bash
		# This script run at */1
		/usr/local/php/sbin/php-fpm reload