## 浅谈 iOS 版本号

FIR.im 一直在尽量兼容不同使用习惯的版本号形式, 但是在使用中我们发现好多开发者对怎么更好的用版本号来标示应用很陌生. 这是篇基础文章, 简单介绍 iOS 的版本号.

![image](http://firclub.qiniudn.com/FoSuESkBT9ubPjLRJm6c57uFe0m4)

##### 名词解释

* Version, 通常说的版本号, 是应用向用户宣传说明时候用到的标识. 一般有2段或者3段式, 如:2.1,8.1.2

	Version 一般由产品部门确定, 完全迥异的更新需要改变主版本号, 比如 QQ 4.0 的变化非常大, 主版本的变化会更加吸引用户的眼球,所以有的应用会频繁的更新主版本号, 比如 FireFox 20.0 . 两段式的副版本号既包含小功能更新也会包含 bug 修复等,三段式副版本基本都是新功能添加和大问题修复,第三段则表示稳定版本基本都是修复 bug

* Build , 编译号指一次唯一编译标识, 通常是一个递增整数(安卓强制为数字, iOS 可以是字符串)

	Build 都是给内部使用, 用来确定一个唯一版本. 与前面提到的 Version 不会有太大联系.

##### iOS 开发中,这个2个号码都可以任意字符串或数字.

我们目前遇到的情况有:

* 忽略了 Version 或 Build. 这两个号中的一个常年的不会发生变化.
* 颠倒了 Version 和 Build.

![image](http://firclub.qiniudn.com/FvIegW3X0g5_RWpXduMzb3kRyIJF)

获取方法也很简单:

```
NSDictionary *info= [[NSBundle mainBundle] infoDictionary];

info[@"CFBundleShortVersionString"]; //Version
info[@"CFBundleVersion"]; // Build
```

##### 为什么使用版本号

* 方便标示和沟通

	前面提到 版本号更新会给推广产生一定的积极作用. 所以版本号不要太长, 如果像这样 "我们隆重推出了 某某某 1.7.14.19257 !", 这个会让用户感觉很乏味很像电视购物,而且也不利于传播. 如果是 "某某 3.0, 大有不同 !"可能就会产生更好的沟通效果.

* 方便追踪 Bug

	一个应用有 Bug 是肯定的, 但是很快的定位解决问题却体现出团队和程序员的能力. 我们经常遇到有开发者说我提交一个版本, 但是下载下来有还是旧的. 我们帮他解决问题的时候,他自己都搞不清哪个是哪个了, 如果能在"关于"之类的地方显示当前的版本, 就会容易找到问题.

或者是测试团队的同事, 可能手里同时有几个不同分支的版本在测试, 他们需要精确的描述一个测试版本.

##### 自动改变 Build 号

前面提到, Version 是不需要自动变化的, 根据产品或者市场部门的需求,适时的手动改一下就好.


###### 1. agvtool (Apple-generic versioning tool)

agvtool, 是苹果的命令行工具, 也是集成在 Xcode 中最方便的工具. 我们在自动编译 SDK 的脚本中用的就是这个方法. 其实就用了一行(其他的高级用法可以参考前面的链接):

	agvtool next-version

使用前需要在 Xcode 里简单配置一下, 如图:

![image](http://firclub.qiniudn.com/FitXOnXNdQbUpoRRWuiCa9VgtfmF)

2. 基于SCM的版本控制号

SCM 现在常用的有 Git 和 SVN, 还有一些相对小众的比如 hg 这里就不多做介绍了.

如果用 Git/SVN 来管理代码(相信已经没有人不用了) 我们可以用代码的提交次数来代替Build号.

* Git

		REV=`git rev-list HEAD | wc -l | awk '{print $1}'`

其中 HEAD是分支名, 代表当前分支, 可以直接替换成其他分支名, 比如master,dev.

这个脚本放到

* SVN

		REV=`svnversion -nc | sed -e 's/^[^:]*://;s/[A-Za-z]//'`

后面都是一样的:

	/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${REV}" "${TARGET_BUILD_DIR}"/${INFOPLIST_PATH}

这样每次编译app的时候就自动把版本号加到Info.plist的CFBundleVersion键值下

把上面2行代码 加到 "Build Phase > Run Script"就可以了:

![image](http://firclub.qiniudn.com/FsHdTgeR_CAVjwTQTz0N9j-R1ZIV)

3. 基于日期时间

用发布日期作为版本好也是许多应用常用的方式, 因为好记好理解. 这里直接附上代码:

	REV=`date +%y%m%d`  #输出格式141120的六位日期格式,可以根据自己喜欢改变格式

后面都是一样的:

	/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${REV}" "${TARGET_BUILD_DIR}"/${INFOPLIST_PATH}

使用方法同上.

#### 怎么使用

只要配置好了版本号, 其他的事情就不需要人工干预了, 这里介绍2种使用场景.

1. Crash 收集

收集 Crash 是应用开发必要的环节, 通过分析和修复 Crash 信息可以大大提高应用的稳定性而不会让更多的用户失望甚至删除应用.
目前有很多收集工具, 比如 FIR.im 旗下的BugHD, Crashlytics等.

![image](http://firclub.qiniudn.com/Fk8-VkIqnU7M1a9XQdTPrHOP-_Ns)

2. 用户反馈

能主动反馈问题的用户都是极品用户, 不管要求是不是合理我们都要认真对待.
不管是用各种 SDK 还是用 Email 都要尽量的带上版本号, 系统信息, 方便确认用户需求.最不济也要在"关于"里面能让用户找到相关的版本信息以便描述问题.