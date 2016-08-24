## 如何写好表单类需求(PHP版)

表单操作是每个Phper都必须要掌握的基本开发技能，也是经常bug频出的地方。

完成一个完美(相当于优秀)表单类操作，考验的是每一个Phper思路是否清晰、考虑是否全面、是否细致耐心、思维是否广阔，也是真正考验开发者功力的地方。

但是在实际项目开发过程中，我们会发现表单类需求是bug出现频率最多的地方，即便是经验丰富的老手也会在此掉“坑”里。

本文通过作者自身开发经验和教训总结，罗列出以下checklist，供Phper参考。

##### 表单基本介绍

`
<form action="xxx" method="POST" >
<p>邮箱: <input type="text" name="email" style="width: 300px" /></p>
<p>密码: <input type="text" name="password" style="width: 300px" /></p>
<p><input type="submit" value="提交" /></p>
</form>
`

通常from表单提交是，需要设置提交方式，即指定method=“GET/POST”，通过这个提交方法就决定了浏览器在提交数据时，通过什么方式来传递它们。

* 如果是 POST，那么表单数据将放在请求体中被发送出去。
* 如果是 GET，那么表单数据将会追加到查询字符串(QueryString)中，以查询字符串的形式提交到服务端。
* 建议: 表单通常还是以post方式提交比较好，这样可以不破坏URL，况且URL还有长度限制。

**注意**: 如果采用GET方式提交表单，并且action中含有queryString时，会导致参数丢失（HTML的规定，通过GET方法提交表单时，action地址里的query string会被丢弃）。
附: [HTML 4.0.1](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.3.4) , [HTML 5](http://www.w3.org/TR/2011/WD-html5-20110525/association-of-controls-and-forms.html#form-submission-algorithm)

	<form id="form1" action="/test.php?param1=aaa">
    	<input type="text" name="param2" value="bbb"/>
	</form>

上述例子中，

form1提交的时候我期望跳转的网址是:`/test.php?param1=aaa&param2=bbb`

可是实际是:`/test.php?param2=bbb`

如何解决这个问题呢？？

so easy，搞个hidden即可！

	<form id="form1" action="/index.php?param1=aaa">
		<input type="hidden" name="param1" value="aaa" />
    	<input type="text" name="param2" value="bbb"/>
	</form>

当然，通过JS方式也可以保留，但是需要变成，比hidden方式麻烦，不推荐使用。

具体做法: 监听form submit事件，onSubmit()时，依次做：

1. 取出action的值，把query string（问号后面那一串）解析出来
2. 往form里apeend两个hidden input元素，name和value使用上面第一步解析出来的结果
3. 用form.submit()提交表单

##### 表单展示布局

* 后台消息展示区(after Submit)
	* success message
	* error message
	* caution message
	* warning message
	* ... 

* 表单操作区
	* input
	* select
	* radio
	* checkbox

* 表单错误显示(before Submit)
	* error tip
	
##### 表单数据校验