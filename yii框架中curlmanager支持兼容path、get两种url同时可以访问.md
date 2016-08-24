## Yii框架中CUrlManager支持兼容path、get两种URL同时可以访问


#### 情况描述：

项目初期没有做URL相关的优化，URL格式是get类型的；

线上运营一段时间后有优化URL的需求，将URL改为path类型的；

#### 出现的问题：

URL改为path类型后，之前get类型的URL就不能访问到正确的地址了

#### 解决问题：

Yii框架是用CUrlManager来做路由的，查看parseUrl()方法可以看到解决方法

```
public function parseUrl($request)
{
    if($this->getUrlFormat()===self::PATH_FORMAT)
    {
        $rawPathInfo=$request->getPathInfo();
        $pathInfo=$this->removeUrlSuffix($rawPathInfo,$this->urlSuffix);
        foreach($this->_rules as $i=>$rule)
        {
            if(is_array($rule))
                $this->_rules[$i]=$rule=Yii::createComponent($rule);
            if(($r=$rule->parseUrl($this,$request,$pathInfo,$rawPathInfo))!==false)
                return isset($_GET[$this->routeVar]) ? $_GET[$this->routeVar] : $r;
        }
        if($this->useStrictParsing)
            throw new CHttpException(404,Yii::t('yii','Unable to resolve the request "{route}".',
                array('{route}'=>$pathInfo)));
        else
            return $pathInfo;
    }
    elseif(isset($_GET[$this->routeVar]))
        return $_GET[$this->routeVar];
    elseif(isset($_POST[$this->routeVar]))
        return $_POST[$this->routeVar];
    else
        return '';
}
```
将第一行的判断修改一下即可


`
if($this->getUrlFormat()===self::PATH_FORMAT && !isset($_GET[$this->routeVar]) && !isset($_POST[$this->routeVar]))
`

ok，现在两种类型的URL都能访问到正确的路径了。

#### NOTE
当然，框架里面的文件一般是不会修改的，此处只是提供一种思路。