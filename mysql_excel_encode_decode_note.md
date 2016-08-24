# mysql导出excel文件的几种方法 

## 方法一
php教程用mysql的命令和shell

	select * into outfile './bestlovesky.xls' from bestlovesky where 1 order by id desc  limit 0, 50;

## 方法二 
把bestlovesky.xls以文本方式打开，然后另存为，在编码选择ansi编码，保存

	echo "select id,name from bestlovesky where 1 order by id desc limit 0, 50;"| /usr/local/mysql/bin/mysql -h127.0.0.1-uroot -p123456 > /data/bestlovesky.xls

## 方法三

	mysql your_database  -uroot  -p  -e  "select   *   from   test.table2 "   >   /home/test.xls

用sz命令将文件下载到本地，打开如果中文乱码，
因为office默认的是gb2312编码，服务器端生成的很有可能是utf-8编码，这个时候你有两种选择，

1.在服务器端使用iconv来进行编码转换
	
	iconv -futf8 -tgb2312 -otest2.xls test.xls

如果转换顺利，那么从server上下载下来就可以使用了。
转换如果不顺利，则会提示：
	
	iconv: illegal input sequence at position 1841 类似于这样的错误，


2.先把test.xls下载下来，这个时候文件是utf-8编码的，用excel打开，乱码。
把test.xls以文本方式打开，然后另存为，在编码选择ANSI编码，保存