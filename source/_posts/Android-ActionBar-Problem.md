
---
title: 关于ActionBar无法显示logo的问题
date: 2015-03-23 19:24
categories: Android
toc: true
---
不知道大家有没有看过Google官方给出的培训教程,昨天我在看ActionBar这一节的时候,有一个问题一直困扰着我.这篇guide的链接我放在下面先.大家可以去看看,多看文档绝对收获多多.
[http://developer.android.com/guide/topics/ui/actionbar.html](http://developer.android.com/guide/topics/ui/actionbar.html)

其中有一段话,我截取下来跟大家一起看一下
![](http://upload-images.jianshu.io/upload_images/174711-e32113be4dcf9ef7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我这个小菜鸟就献个丑,翻译一下,翻译的不好的话,可不要打我啊.
<!--more-->
**使用logo来替换icon**
**默认情况下,系统会在<application>或者<activity>标签中通过[Android](http://lib.csdn.net/base/android):icon属性把你应用的图标显示在action bar上,可是,我们也可以通过android:logo属性来指定其他的图标进行显示.**
******
**一个logo通常应该比icon要宽一点,但不应该包含不必要的文字.合理的使用情景是,这个logo应该被你的用户所熟知,并且它就代表着你这个品牌的标志.YouTube的手机客户端就是一个好的例子--logo代表着用户预期的标志,而应用的icon是一个为了迎合应用启动图标方形的形状要求的修改版本.**
****
所以好奇的我就马上在Android Studio上新建了一个项目,试一试这个特性,然后悲伤地事情就来了.这么改竟然没用,这个logo死活就是不出来.于是我Google了一下,发现了一个让它出现的办法,不过稍微麻烦一点.需要在OnCreate()方法里加上几句话.(强调一下项目里面的Activity父类是ActionBarActivity,如果父类是Activity的话就没有这种问题).


```
ActionBar actionBar = getSupportActionBar();  
actionbar.setDisplayShowHomeEnabled(true);  
actionBar.setLogo(R.drawable.ic_action_refresh);  
actionBar.setDisplayUseLogoEnabled(true);  
```

这样就会出现logo了,像下面这样.![](http://upload-images.jianshu.io/upload_images/174711-4c38e95f32ddaa42?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是目前我知道的能解决问题的唯一办法.我还看见一种说法就是,在最新的更新中Google说过,已经推荐用兼容包里的ToolBar替换掉原来的ActionBar,母亲啊我还是不太懂,等我看明白了最新的API再回来更新.