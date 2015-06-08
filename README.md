# iOS Best Practices
这里总结一下iOS最佳实践，主要参考Futurice developers的文章[iOS Good Practices](https://github.com/futurice/ios-good-practices)，同时加上了一些自己的理解

##目录
* [初始化项目](#setup_project)
* [iOS架构](#iOS_architecture)
* [API设计](#api_design)
* [编码规范](#code_style)
* [iOS开源库](#iOS_open_source_lib)
* [自动化构建](#build_automation)
* [可执行文件瘦身](#skim_executable_files)

<b id="setup_project"></b>
##初始化项目
###UI采用Interface Builder还是手写代码
当开始一个iOS项目时，一个常见的问题就是编写所有的views是采用手写代码的方式还是使用Interface Builder的Storyboard或XIB文件。两种方式都可以实现相同UI效果，但几点需要考虑：

####为什么使用代码？
* 当多人协作开发时，由于Storyboard复杂的XML结构，它更容易发生版本冲突，这比手写代码的方式更难合并代码。
* 手写代码更容易地结构化和重用views，因此使你的代码库保持[DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。
* 所有的信息都在同一个地方。在Interface Builder中，你必须通过点击所有inspectors才能找到你想要的东西。

####为什么使用Storyboard？
* 为了更少的技术要求，Storyboard通过微调颜色或布局约束等方式来直接对项目做出贡献是一种很好的方式。但它需要设置工程并且需要时间来学习基础。
* 由于你无需构建项目就可以预览一些变化，迭代速度就变得更快了
* 在Xcode 6，自定义字体和UI元素最后都以可视化的方式在Storyboard上显示，当你设计最终外观时，给你更加好的灵感。
* 从iOS 8开始，[Size Class](http://blog.futurice.com/adaptive-view-ios8)允许你为不同的类型的设备和屏幕来设计并且没有重复


###Git版本控制与.gitignore
创建工程之后，一个好的步骤就是使用[git](https://git-scm.com)对工程进行版本控制并且有个合适的`.gitignore`文件。这个文件的作用主要是忽略不想要的文件(用户设置、临时文件等)，不将它们存放在仓库。幸运的是，Github都提供[Objective-C](https://github.com/github/gitignore/blob/master/Objective-C.gitignore) 和 [Swift](https://github.com/github/gitignore/blob/master/Swift.gitignore)两种模板。

###Build设置与Faux Pas
为了提高**代码质量**，你可以配置一些Build Settings：

* 启用**Analyze During 'Build'**，在源文件运行Clang静态分析工具
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Enable-Build-Static-Analysis.png)

* 启用**Treat Warnings as Errors**，它会导致所有的警告都当做错误来处理
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Treat-Warnings-As-Errors.png)

除了配置build settings之外，你还可以通过工具[Faux Pas](http://fauxpasapp.com/)来检测静态错误。它通过分析你代码库，然后查找出你意想不到的问题。在上架任何iOS或Mac的app之前使用它吧。

###使用CocoaPods管理库
如果你打算引入额外的依赖(例如，第三方库)到你工程，[CocoaPods](http://www.cocoapods.org/)提供一种容易和快速的集成方式。下面简单介绍它的安装和使用

####安装CocoaPods
使用以下命令就能安装**CocoaPods**：

```
sudo gem install cocoapods
```
输入上述命令可能很久都没响应，因为CocoaPods也被墙了。所以使用淘宝的镜像会快不少，

```
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
```

####使用CocoaPods
* 使用`pod search`来搜索你需要安装的库。例如，你想安装`mantle`这个iOS库，使用这个命令来搜索：`pod search mantle`，显示以下结果：
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/pod-search.png)
红色圈住的内容就是安装库时`Podfile`文件需要的配置信息

* 在工程目录下创建`Podfile`文件。这个文件主要保存依赖库信息，告诉CocoaPods去下载和管理哪些依赖库。创建之后，将`pod 'Mantle', '~> 1.5.4'`追加到Podfile文件
![](https://raw.githubusercontent.com/samlaudev/iOS-Best-Practices/master/ScreenShots/Podfile-Demo.png)

* 使用`pod install --verbose --no-repo-update`安装第三方库。

* 如果在后来为了添加或删除第三库而修改`Podfile`文件，使用`pod update --verbose --no-repo-update`更新第三方库。

* 由于安装和更新库的命令太长，使用shell的命名别名来简化，以后只需输入`pod_install`和`pod_update`即可安装和更新第三方库了

 ```
 #setup pod update alias name
 alias pod_update='pod update --verbose --no-repo-update'
 alias pod_install='pod install --verbose --no-repo-update'
 ```
* 安装好第三方库之后，关闭Xcode，然后打开`.xcworkspace`而不是`.xcproject`文件，然后引入对应第三方库的头文件，你就可以使用第三方库了。

###项目目录结构
为了组织目录里面的上百个源代码文件，最好的方式就是根据你的架构来设置目录结构。例如，你可以采用以下目录结构：
	
    ├─ AppDelegate
    ├─ General
    ├─ Macro			
    ├─ Sections
    ├─ Helpers
    ├─ Vendor
    ├─ Resources
   
* ####AppDelegate
这个目录主要存放AppDelegate.h(.m)文件，是整个app的入口文件，所以单独拿出来。

* ####General
这个目录放会被重用的Views、Controllers、Categories和DataSources等。里面大概是这样：

 ```
 General
     |- Cell
        |- ELMBaseCell
        ...
     |- Views
        |- ELMScollView
        |- ELMPullToRefresh
        ...
     |- Controllers
        |- ELMBaseViewController
        ...
     |- Categories
        |- UIViewController+Sizzle
        |- UIImageView+Downloader
        ...
     |- DataSources
        |- ArrayDataSource
        ...
 ```
这里的`ELM`是项目的首字母缩写

* ####Macro
这个目录下放了整个app会用到的宏定义，里面大概是这样：

 ```
 Macro
     |- AppMacro.h
     |- NotificationMacro.h
     |- VendorMacro.h
     |- UtilsMacro.h
     ...
 ```
如果有新的类型的宏定义，可以再新建一个相关的Macro.h。

* ####Sections
这个目录下面的文件对应的是app的具体单元，如导航、瀑布流等等。里面大概是这样：

 ```
 Sections
     |- Menu
     |- Setting
     |- Collection
     ...
 ```
 而每个具体单元对应存放几个部分`View`, `Controller`, `ViewModel`, `Model`, `Network`的类。
 
* ####Helpers
这个目录放一些帮助类，文件名与功能挂钩。里面大概是这样：

 ```
 Helpers
     |- ELMShareHelper
     |- ELMHelper
     |- ELMEmotionHelper
     ...
 ```
 帮助类的主要作用是帮助Controller瘦身，也可以提供一定程度的复用。

* ####Vendors
这个目录放第三方的类库/SDK，如UMeng、WeiboSDK、WeixinSDK等等。

* ####Resources
这个目录下放的是app会用到的一些资源，例如声音、数据，但**图片**存放在`Images.xcassets`目录。

项目结构的参考链接：[iOS项目的目录结构和开发流程](http://limboy.me/ios/2013/09/23/build-ios-application.html)

###Assets资源

<b id="iOS_architecture"></b>
##iOS架构

<b id="api_design"></b>
##API设计
###面向对象设计原则
###设计模式

<b id="code_style"></b>
##编码规范
关于Objective-C的编码规范，请参考我翻译的[raywenderlich.com Objective-C编码规范](https://github.com/samlaudev/Objective-C-Coding-Style)

<b id="iOS_open_source_lib"></b>
##iOS开源库
* [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
* [AFNetworking](https://github.com/AFNetworking/AFNetworking)
* [Mantle](https://github.com/Mantle/Mantle)
* [Masonry](https://github.com/SnapKit/Masonry)
* [Classy](https://github.com/cloudkite/Classy)
* [ClassyLiveLayout](https://github.com/olegam/ClassyLiveLayout)
* [Kiwi](https://github.com/kiwi-bdd/Kiwi)

<b id="build_automation"></b>
##自动化构建
###构建配置

###分析

###部署配置

<b id="skim_executable_files"></b>
##可执行文件瘦身
