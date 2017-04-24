---
title: 给Android Studio设置代理
date: 2015-02-06 18:56
categories: Android
toc: true
---
Android Studio是基于JetBrains公司的IDEA开发的,Android Studio里的项目都是由Gradle构建的,Gradle集合了Ant和Maven的优点,又解决了他们的缺点,但是它有一个特点还是值得我们注意的.我们每一次点击这个
<!--more-->
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-c79328206ec65ebf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按钮来build我们的程序时,都会联网检查更新gradle信息,这个和Eclipse上还是不一样的,因为Gradle构建的时候要联网,但是联网就联网呗,偏偏还要连接到墙外面的网络,我当初刚开始折腾Android开发的时候可被它给坑苦了,现在想一想真是蛋疼啊,信心满满的装好了高大上的Android Studio,但是由于对它不够了解,且没有深刻理解付费科学上网的重要性,时常出现Gradle Sync Failed的错误,弄得我在Ubuntu和Windows上来回折腾了好几回,一直以为是自己，的系统有问题,直到后来随着学习的深入才搞明白.

这真的是一个大坑,设想一下,如果我当初稍微不坚定一点,那么可能就跟Android开发Say GoodBye了.所以才有了写下这篇博客的念头,希望这之后的新人们能少踩一些坑.

作为一个天朝的Android程序员,如果要是不会科学上网的话,那是不是有点太逊了.先说说我吧.第一次跳出墙是刚上大二的时候,偶然间听说了一个叫Go Agent的开源软件,能免费FQ,所以我在折腾了两个多小时之后终于体会了一把什么叫外面的世界,当时还截了几张YouTube和FaceBook的图发在QQ空间上(后来觉得比较二逼就删掉了),当时还真的是挺兴奋的.后来知道了Go Agent是由Python语言编写的,并且很多人似乎对这门语言很推崇,我就特意去图书馆借了本Python核心编程翻了翻.算是对Go Agent的致敬吧.至于如何使用Go Agent我在这里就不介绍了,大家有兴趣的话可以找一篇帖子试一下.

现在已经不用GoAgent,因为最近GFW屏蔽的比较严重了,Go Agent十分的不稳定,总是需要更换IPList,所以转战到ShadowSocks了.中文名叫做影梭![](http://upload-images.jianshu.io/upload_images/174711-ce3ea1f008eb5f7e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240),就是它.是付费的，但是价格真的比较良心。

好了,扯得好像有点远了,快点进入正题吧.

先打开我们的Android Studio,点击工具栏的file下的settings,如下图

![](http://upload-images.jianshu.io/upload_images/174711-6498783233d9f502?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之后再搜索框上面输入Proxy,然后按第四步提示点击,如下图

![](http://upload-images.jianshu.io/upload_images/174711-2583e9cbab166d14?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之后就进入了设置代理的界面了,如下图

![](http://upload-images.jianshu.io/upload_images/174711-ed14ec0583e20854?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认情况下,No Proxy是被选中的,意思是不需要设置代理.如果你用的是ShadowSocks代理的话则可以按照下面的5 6 7 8四步来做,如下图:

![](http://upload-images.jianshu.io/upload_images/174711-4fb19ea6239addb7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里稍微解释一下,

Manual proxy configuration翻译过来是人工设置代理的意思.
ShadowSocks是SOCKS代理方式
127.0.0.1的意思是用你本机做代理
1080是ShadowSocks默认的端口号

这时候如果你的ShadowSocks是能正常工作的话,那么就可以实现Android Studio上网了.测试一下,点击工具栏的Help下的Check for Update选项,如果没有提示不能联网或者提示你更新Studio的话,就说明你成功了少年.

最后,如果你用的是GoAgent的话,只需要把端口号修改为8087就可以了,其他任何一步都不需要改变,至于其他的VPN的话,请参考自己的软件进行设置吧.