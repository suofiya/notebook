# svn强制提交注释规则及触发邮件通知

详见文档 [svn强制提交注释规则及触发邮件通知](http://www.2cto.com/os/201401/271911.html)

Linux上面的强制提交代码

	# cd /svnroot/test/hooks
	# cp pre-commit.tmpl pre-commit
	# vi pre-commit
	在末尾删除原来的，添加上以下参数
	REPOS="$1"
	TXN="$2"
	SVNLOOK=/usr/bin/svnlook
	# check that logmessage contains at least 5 alphanumeric characters
	LOGMSG=`$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c`
	if [ "$LOGMSG" -lt 5 ];
	then
		echo -e "/nEmpty log message not allowed. Commit aborted!" 1>&2
		exit 1
	fi
	//[ "$LOGMSG" -lt 5 ] -lt 5这个5是至少为5个字符，请注意。
	# chmod a+x pre-commit   //添加可执行权限给pre-commit


# （总结）CentOS Linux搭建SVN Server配置详解

[搭建SVN Server配置详解](http://my.oschina.net/danyang/blog/132840)