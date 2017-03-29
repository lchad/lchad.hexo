---
title: 解决华为手机不输出log的问题
date: 2016-11-20 18:56
categories: Android
toc: true
---
最近换了一台华为P8的测试机，EMUI（4.0.1）真够丑，我的圆形Launcher图标也会被处理成圆角矩形。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-10f909f95aa2ee18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

还有一个更严重的问题，我在代码里的`Log.d`和`Log.w`日志永远打不出来，程序崩溃之后的话，也看不到报错信息，只能靠打断点和瞎猜，目测浪费了我至少好几个小时的宝贵时间，NND。
<!--more-->
搜了一下，华为手机好像普遍都有这个问题，来，接锅吧，华为。华为的系统默认会把日志打印系统关闭掉， 打开的方式是在拨号应用里输入`*#*#2846579#*#*`，进入隐藏的设置界面：

![4075B264C613DB2822C44C69F83D692B.png](http://upload-images.jianshu.io/upload_images/174711-2e762fed48fccbc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)
点击`1.后台设置`
![CA802F8041A9DF3AC636E99A8C3E0F66.png](http://upload-images.jianshu.io/upload_images/174711-afff4a89501584ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)
点击`3.LOG设置`

![E2EEF5A89ADF5F5531D51E415AD85382.png](http://upload-images.jianshu.io/upload_images/174711-f682b7b91dc3db13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

把这些都勾选上。

接下来，做一个测试，我在Android Studio工程的onCreate()里插入一行代码：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-962d6081c368b3b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行之后，程序崩溃，在logcat下面可以看到如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-d23fec9a646e096a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
log系统终于恢复正常了。

另外之前还遇到过，华为手机打开开发开发者Android Studio不显示的问题，原因是Android驱动在Windows上没有安装成功，我之前的解决方法是安装豌豆荚，之后豌豆荚会自动下载对应的驱动。其实这个问题还有更简单的解决方法。答案还是在这个隐藏界面里。把下面的USB端口设置切换成Google模式的话，问题就迎刃而解了。
![1CE3D6BB4840541EF4E3B4E8342B6C2D.png](http://upload-images.jianshu.io/upload_images/174711-be273e37e52b487e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)