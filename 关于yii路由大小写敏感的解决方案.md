## 关于Yii路由大小写敏感的解决方案

示例：默认情况下，以下路由

URI-1: [http://www.w3c.com/StoreHouse/show](http://www.w3c.com/StoreHouse/show)

URI-2: [http://www.w3c.com/storehouse/show](http://www.w3c.com/storehouse/show)

在windows开发环境下没有区别，而源码部署到linux/unix服务器环境下之时，倘若该控制器的路径是root/project/StoreHouseController.php中的actionShow()，则URI-1可正常访问，URI-2访问出现http-404异常。


##### 究其原因：


在yii框架的framework/web/CUrlManager.php中定义了开放属性


	/**
	* @var boolean whether routes are case-sensitive. Defaults to true. By setting this to false,
	* the route in the incoming request will be turned to lower case first before further processing.
	* As a result, you should follow the convention that you use lower case when specifying
	* controller mapping ({@link CWebApplication::controllerMap}) and action mapping
	* ({@link CController::actions}). Also, the directory names for organizing controllers should
	* be in lower case
	*/
	public $caseSensitive=true;


默认true值表示路由地址大小写敏感，false值表示路由不区分大小写。


##### 据Yii官方参考描述：


Note: By default, routes are case-sensitive. It is possible to make routes case-insensitive by setting [CUrlManager::caseSensitive](http://www.yiiframework.com/doc/api/1.1/CUrlManager#caseSensitive) to false in the application configuration. When in case-insensitive mode, make sure you follow the convention that directories containing controller class files are in lowercase, and both [controller map](http://www.yiiframework.com/doc/api/1.1/CWebApplication#controllerMap) and [action map](http://www.yiiframework.com/doc/api/1.1/CController#actions) have lowercase keys.


当在大小写不敏感模式中时，要确保你遵循了相应的规则约定，即：包含控制器类文件的目录名小写，且控制器映射和动作映射 中使用的键为小写。

###### 解决方案

所以，除了配置main.php中的组件配置项urlManage属性caseSensitive=true以外，还应该确保以下几点：

1. 控制器所在目录名称为小写；

2. 控制器文件名和类名仅限于形如“StorehouseController.php”，杜绝“StoreHouseController.php” ；

3. 如若控制器文件上层目录结构中有modules，则所属的模块目录名称也需要为小写，模块入口文件XyzModule.php遵循第2条约定。


*出现以上的约束情况的原因如下*：

在yii框架中framework/web/CWebApplication.php之方法createController()创建控制器方法中，


	if(!$caseSensitive)
		$id=strtolower($id); //若然等于false，则控制器ID转换成小写形式

首先会读取配置信息，其中包含检查了caseSensitive属性，若然等于false，则控制器ID即转换成全小写形式。

继而，每当创建控制器方法被调用的同时，控制器的首字母皆转换成大写形式。

	//每当创建控制器实例，即时将控制器ID首字母转为大写形式
	$className=ucfirst($id).'Controller';


如此，便造成了以上异常现象。