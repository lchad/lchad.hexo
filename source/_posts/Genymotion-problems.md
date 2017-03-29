---
title: Genymotion那点事儿
date: 2016-05-14 01:04
categories: Android
toc: true
cover: /img/genymotion.png
---
`Genymotion`是一款非常好用的Android模拟器.它在Android开发者中使用率估计也是最高的.在我使用Genymotion的一年多时间内,碰到过几个令人蛋疼甚至蛋碎的问题,折腾它也浪费了我很多的精力和时间,所以今天就把使用Genymotion中遇到的问题做一个总结,希望大家尤其是像我一样的新手避免重复踩坑,把尽可能多的精力都花在代码上.

关于如何安装Genymotion的问题我不会多说,也没这个必要,因为无论我怎么说肯定没有官方网站上`文档`讲的清楚.比较重要的点稍微提一下:
**1.**要去[Genymotion官网](https://www.genymotion.com/)注册一个账号.
**2.**安装[Virtual Box](https://www.virtualbox.org/wiki/Downloads)虚拟机。
<!--more-->
现在我假设你已经成功安装了Genymotion.接下来介绍一下你`可能`遇到的各种奇葩的问题.

## No.1 : 无法下载Genymotion内的镜像
大概在2015年下半年的时候,我突然遇到了这样令人`懵逼`的情况:不翻墙的时候无法安装虚拟机镜像，挂上了ShadowSocks之类的代理之后还是没办法安装，在Genymotion里登录账号的时候它可能会这样:

    genymotion invalid reply from server this may occur if you are using a proxy
即使登陆成功但是选择一个镜像点击下载的时候会还`可能`会这样：

    Unable to create virtual device. server returned HTTP status code 0.

这特么就尴尬了.Google了好久也试了很多方法，大致找到了几种解决办法.
**1.**这是我在`StackOverFlow`上找到的方法，点击`Genymotion`->`Setting`->`Network`,勾选上use HTTP proxy选项,`HTTP Proxy`这里输入 1.234.45.50 ,`Port`输入 3128 .如下图:

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-db7d0804a2c3134d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过这个方法我成功下载了一个镜像,但是之后再次尝试也遇到失效的情况.所以这个方法不是很可靠.

**2.**还有一种方法是用[Proxifier](https://www.proxifier.com/)这个软件给Genymotion再加一层网络代理,代理加上之后就能下载了,亲测可行.至于`Proxifier`这个软件的使用我就不介绍了,大家随便找一篇帖子看看就会用了.(Proxifier支持`Mac`和`Windows`平台,有一个月的免费体验资格,作为程序员还是应该多多支持正版,你懂得.)

**3.**接下来这个方法就比较巧妙,下载速度也是最快的,可以达到满速,但缺点就是稍微麻烦了一点.Genymotion会维持一个日志文件,文件名叫做`Genymotion.log`具体位置是在你的User目录下的.Genymotion目录下,如果你是Linux系统的话可以参考下图:

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-b0e8763238fe4d2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Windows系统是在`C:\Users\liuchad\AppData\Local\Genymobile`这个目录下.

现在我在`Terminal`下用`VIM`打开这个日志文件,如下图:
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-5854a8790257b4db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大致浏览一下这个文件会发现一些类似[Genymotion] [Debug][downloadFile]....格式的行,例如:

    3月 14 13:48:07 [Genymotion] [Debug] Launching download
    3月 14 13:48:07 [Genymotion] [Debug] [downloadFile] 
    "https://cloud.genymotion.com/vmtemplate/a9af19b8-8df9-41a8-81ac-a6294648ecc1/getova" 
    -> "/Users/liuchad/.Genymobile/Genymotion/ova/genymotion_vbox86p_5.1_151117_200001.ova"

如果你翻到文件的最底部的那一条,其实它就是你刚刚想要下载但是没有成功的那个镜像的`请求地址`.所以我们可以尝试着把这个链接粘贴到浏览器里,神奇的事情发生了,我们得到了一段`json`串:

    [
    {
    platform: "tp",
    ova_name: "genymotion_vbox86tp_5.1_151117_203708.ova",
    ova_hash: "d57e3f5425ead4dc40b91cadbc179868af47fcfe",
    ova_size: 259917312,

    ova_url: "http://files2.genymotion.com/dists/5.1.0/ova/genymotion_vbox86tp_5.1_151117_203708.ova"
    }
    ]

点击ova_url后面的链接就自动在浏览器里下载对应的ova文件了.

**需要注意的一点:**要事先在浏览器里登录Genymotion的账号,如果处于未登录状态的话,会被`重定向`到Genymotion的官网,也就无法获取这段json串了.

下载完毕之后把准备好的`ova文件`拷贝到这个路径下(**Linux系统**):

    /Users/liuchad/.Genymobile/Genymotion/ova

Windows系统下对应的路径是:

    C:\Users\liuchad\AppData\Local\Genymobile\Genymotion\ova
最后重新进入到Genymotion的下载界面,点击你这个ova文件对应的那个镜像下载,瞬间就下载好了.

**强调一下:**以上请把`liuchad`替换成你自己的系统用户名.

## No.2 : 无法安装应用
如此费尽周折之后,你高兴的成功安装好了Genymotion,也下载好了各种机型的虚拟机,准备开心的写代码了,但这世界往往不会如想象的那么美好.

因为你可能会这样:


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-dea5a4e41f2a2a7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还可能会这样:

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-82a4d13f92394099.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其实Genymotion上出现无法安装应用的原因在官网的FAQ上就可以找到线索,请看下面这一段:

 `Q : Why does Genymotion return an error message when I try to install an ARM application?
 A : When you try to install an application compiled for the ARM architecture in Genymotion, you get the following error message:INSTALL_FAILED_CPU_ABI_INCOMPATIBLE.
 Genymotion uses the x86 architecture, therefore your application is not directly compatible. Depending on your situation, follow either of these methods:
 -You are the developer of the application:
 You only need to add the x86 build target to your current targets. For more information, please click here.
 -You are not the developer of the application:
 You can either install ARM to x86 translation libraries, which you can find on the Internet or you can contact the developer of the application to ask for x86 architecture support.
An ARM application running on Genymotion is less stable and efficient than an x86 application. Therefore, we strongly recommend that you only use x86 applications with Genymotion.`

Android完全是被ARM所统治的,而Genymotion为了提高运行速度,使用了x86的架构,这也是为什么Genymotion能够比其他模拟器快的根本原因.所以在x86架构上安装ARM的应用肯定会出现冲突不兼容的情况,解决的办法就是安装`Genymotion-ARM-Translation`补丁,补丁文件可以到这里下载:

链接: http://pan.baidu.com/s/1nv1r3EL 密码: pfjj

下一步就是直接把这个下载好的补丁文件(**不用解压!不用解压!不用解压!**)拖拽到Genymotion里,如果是早前版本的Genymotion的话,会直接提示安装成功,重启一下虚拟机就可以正常安装应用了,但是如果是在较高版本的Genymotion中拖拽成功之后会提示如下:

    Files successfully copied to: /sdcard/Download

竟然只是把文件复制到了虚拟机里,没办法,我们还需要手动的把补丁安装到虚拟机里.进入adb,依次输入如下三条命令：

    >  adb shell
    >  cd /sdcard/Download/
    >  sh /system/bin/flash-archive.sh /sdcard/Download/Genymotion-ARM-Translation_v1.1.zip
最后,重启模拟器即可.至于adb是什么,adb怎么用,由于篇幅的限制,这里不会介绍.

 
以下我使用Genymotion版本信息:
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-6f6b02115b4dd4c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 在Genymotion中输入中文
Genymotion默认是不支持输入中文的,有些情况下我必须要输入中文怎么办呢?这就有些蛋疼了.我尝试安装了很多款手机输入法,例如,搜狗输入法,百度输入法.触宝输入法,Google拼音输入法,QQ手机输入法,这些都是没什么卵用的,测试下来只有必应输入法是可以输入中文的.微软大法好!最后还要在设置里勾选上`硬件`->`显示输入法`,如下:


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-77fd08ffdabb6763.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后的效果就是这样滴:

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-75f6fc4882828960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)