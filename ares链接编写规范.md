## Ares URL编写规范

##### 背景: 
因为link会涉及SEO优化相关逻辑，页面中显示的link必须为 **绝对化** 和 **伪静态化**, 因此就需要我们在页面生成link时特别注意

* 切记:__不可因偷懒将link写死__。

##### 解决方案:
生成和解析url都必须使用统一的处理函数，在函数里对url做更进一步的处理。

##### 试用范围:
* 本规范目前仅适用于网站展示前台(包含pc端和mobile端)，
* 管理后台最好也使用此规范，但不强制要求；
* ~~(待定)不需要伪静态化的链接~~
	* form表单提交action，需要注意url必须相对于根路径
	( /index.php?k1=v1&k2=v2 )	 **注意index.php前的slash**
	* 搜索页面
	* Ajax异步请求的url

##### 如何规范:

根据使用场景不同，分为三种情况使用

1. 业务Controller中
	
	使用Yii框架定义好的方法**createAbsoluteUrl**:
	
		$this->createAbsoluteUrl($route, $params = array(), $schema = '');
	
	参数说明:
	* 参数 $route 为路由信息，Required，格式为ControllerId/ActionId
	* 参数 $params 为请求参数，Optional，默认为空数组
	* 参数 $schema 为协议类型，Optional，，默认为http请求方式，安全模式可设置为https
	

2. Smarty模板中
	
	使用Smarty函数插件(已封装好，直接使用即可)，**yii_createurl**
	
		{yii_createurl c=ControllerId}
		{yii_createurl c=ControllerId a=ActionId}
		{yii_createurl c=ControllerId a=ActionId var1=value1 ...}
		{yii_createurl c=ControllerId a=ActionId ssl=true}

	参数说明:	
	* 参数 c   为控制器ID，Required
	* 参数 a   为动作ID，Required
	* 参数 ssl 为协议类型，Optional，设置为true则表示为https
	* 参数 var1 为请求参数1，Optional
	* 参数 var2 为请求参数2，Optional
	* 。。。
3. JS代码中

	使用js全局函数(已封装好，直接使用即可)，**gYiiCreateUrl**
	
	
		function gYiiCreateUrl(controllerId, actionId, paramsStr)
	
	
	参数说明:
		1. 参数controllerId为 控制器ID
		2. 参数actionId为 动作ID
		3. 参数paramsStr为 参数的字符串组合，形如:k1=v1&k2=v2&k3=v3
	
	Example：
	
		gYiiCreateUrl('ajax', 'addFavorite', 'k1=v1&k2=v2&k3=v3....');

4. 数据服务类中

	使用封装好的Ares工具函数**generateAbsoluteUrl**
	
		AresUtil::generateAbsoluteUrl($route, $params = array(), $schema = 'http')
	
	参数说明:	
	* 参数 $route 为路由信息，Required，格式为ControllerId/ActionId
	* 参数 $params 为请求参数，Optional，默认为空数组
	* 参数 $schema 为协议类型，Optional，，默认为http请求方式，安全模式可设置为https

##### 特别强调
0. 一般业务需要只需要前三个函数就足够了，即：
   * PHP函数**$this->createAbsoluteUrl**
   * smarty函数**yii_createurl**
   * JS函数**gYiiCreateUrl**)
1. 页面出现的所有链接必须使用以上三个函数处理，如果不能满足需求请及时提出
2. 前台页面中不允许使用相对地址 (保证可以通过urlManager完成伪静态化)
3. 前台静态资源(图片/css/js)必须使用绝对地址(原因在于相对路径在静态化后导致路径错误)，已在components/Controller.php中统一处理
4. ajax请求中地址使用绝对地址
