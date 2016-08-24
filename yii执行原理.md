# Yii执行原理

## 图解

![image](http://www.gaojie360.cn/yii/img/yii.jpg)

##### ---------------------- 以下为详细流程 -----------------------

## 应用执行流程：

```
浏览器向服务器发送 Http Request
    |
  控制器(protected/controllers)
    |
    |---> Action
            |
         创建模型 (Model)
            |
         检查$_POST输入
            |
         渲染视图
            |
         render()第二个参数作为控制器与视图接口参数
            |
            |----> View (protected/views)
                     |
                   使用$this访问控制器的变量（包括layout, widget)
                   
               
-----------------------------------------------------------------

```

## 视图渲染流程：

```
render($view, $data, $return)
    |
beforeRender()
    |
渲染View文件，调用renderPartial()，要求处理输出结果
    |
    |----> 根据$view得到viewFile文件名
                    |
           renderFile()，要求返回渲染结果，做下一步处理
                    |
                    |-----------> 获取widget的数目
                                         |
                                  从Yii::app()获得render
                                  CWebApplication::getViewRenderer
                                  查询component['viewRenderer']，默认没有配置
                                         |
                                  Then, 调用renderInternal()
                                         |
                                         |---------> require View文件，渲染，根据需要返回渲染结果
                                                          |
                                         |<---------------|
                                         |
                    |<-------------------|
                    |
               处理输出结果processOutput()
                    |
               按照caller参数，返回输出，而不是echo输出
    |<--------------|
    |
渲染layout文件
    |

----------------------------------------------------------------------
```

## 加载控制器及其方法：

```
根据route信息，获得当前控制器
      |
初始化当前控制器，CController::init()，默认为空
      |
执行当前控制器，CController::run()
      |
      |----> 创建action，为空则默认为index
                     |
             得到CInlineAction的实例
                     |
             用父对象执行beforeControllerAction：默认是CWebApplication，直接返回TRUE
                     |
                执行action
                     |----> 备份原来的action
                                 |
                            执行beforeAction()
                                 |
                            runWithParams()----> 实际上是执行CInlineAction->runWithParams()
                                                             |
                                                 在实例中，执行SiteController->actionIndex()
                                                             |
                                                 渲染页面：render('index')
                                                             |
                                 |<--------------------------|
                                 |
                            执行afterAction()
                                 |
                            恢复原来action
                                 |
                     |<----------|
                     |
             用父对象执行afterControllerAction：默认是CWebApplication，为空
       |<------------|
     完成


----------------------------------------------------------------
```

## 应用执行流程：
```
index.php
    |
require_once($yii)
    |
    |------------->yii.php
                    |
                  require(YiiBase.php)
                    |
                    |---------------->YiiBase.php
                                        |
                                      Define YII_XXX global variable
                                        |
                                      Define Class YiiBase
                                        |
                                      Autload Class YiiBase （自动加载类机制）
                                        |
                                      require interface.php
                                        |
                    |<------------------|
                    |
                   define null Class Yii
                    |
    |<--------------|
    |
createWebApplication($config)---------->|
                                        |
                                      YiiBase::createApplication('CWebApplication',$config)
                                        |
                                      Create Instance of CWebApplication
                                        |
                                        |--------->CWebApplication
                                                      |
                                                   CApplication($config)构造函数
                                                      |
                                                      |------>|
                                                           setBasePath
                                                              |
                                                           set path alias
                                                              | 
                                                           preinit() 空方法
                                                              | 
                                                           initSystemHandlers()
                                                              | 
                                                           configure($config) 将配置文件信息保存到Application
                                                              |
                                                           attachBehaviors()
                                                              | 
                                                           preloadComponents() --> 装载在configure($config)中配置需要preload的components
                                                              | 
                                                           init()                                                              | 
                                                      |<------|
                                                      |
                                        |<------------|
                                        |
    |<----------------------------------|
    |
app->run()
    |
    |---->CWebApplication::processRequest()
                      |
                      |----> CWebApplication::runController($route)
                                       |
                                       |---->createController($route)
                                                    |
                                                 如果$route是空，添加默认controller，对于CWebApplication的controller是"site"
                                                    |
                                                 Controller类是SiteController，require该类文件
                                                    |
                                                 如果该类是CController的子类，修改id[0]为大写，创建该类的实例
                                                    |
                                                    |---->CSiteController
                                                              |
                                                          extends from Controller 这是客户化控制器的基本类，存在于components下
                                                          定义了页面的通用布局
                                                              |
                                                          使用CController构造函数创建对象CSiteController，具体初始化数据见yii-1.png
                                                              |
                                                    |<--------|
                                                备份$this->_controller
                                                $this->_controller = $controller
                                                    |
                                                 调用控制器类的init()方法，默认为空
                                                    |
                                                 调用控制器类的run()方法，默认为CController的run()
                                                    |
                                                    |---->createAction()
                                                              |
                                                           if($actionID==='') $actionID设置为$this->default ("index")
                                                              |
                                                              |Yes
                                                              |----> return CInlineAction($this, $actionID)
                                                              |No              |
                                                           从Map创建           |
                                                              |      执行当前类CInlineAction的父类CAction的构造函数
                                                              |      设置_controller和$id
                                                              |                |
                                                              |<---------------|
                                                              |
                                                              |
                                                           这里得到一个CAction的实例
                                                              |
                                                           $this->getModule()作为parent，为空则使用Yii::ap
                                                              |
                                                           $parent->beforeControllerAction() ??
                                                              | 
                                                            $this->runActionWithFilters($action,$this->filters());
                                                              | 
                                                            $parent->afterControllerAction($this,$action);
                                                    |<--------|
                                                    |
                                                  恢复原来的$oldController
                                       |<-----------| 
                                       |
                       |<--------------|
                       |
                  End of processRequest()
                       |
    |<-----------------|
    |
End of app->run()

------------------------------------------------------------
```