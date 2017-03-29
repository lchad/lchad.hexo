---
title: Android一行代码创建Gif动图
date: 2017-03-26 20:05
categories: Android
toc: true
---

开篇之前先看一下效果.以下示例程序Apk可以在本文底部找到下载地址.

## 一脸懵逼.gif 是如何产生的?
![](https://github.com/lchad/Gifflen-Android/raw/master/img/GIF.gif)
---


话说,那还是 2016 年的时候,由于自己比较喜欢用表情包斗图,所以为了方便,就开发了一个创建表情图片的 APP ,功能简单但是实用,我的很多朋友也都下载使用了,很多人跟我反馈说,如果可以做成动态图就好了.之后我 Google 了一番,发现并没有什么好的方案,于是这个功能就太监了.想说的就是拖了这么久,这个周末花了两天的时间把这个功能实现了.

由于 Android 平台对Gif的支持很很好,没有现成的Java Api可以用,所以我借助NDK作为桥梁,通过C++语言实现对32-bit ARGB图片进行 Color Quantization,转化成Gif动态图(256色域).这个项目的色彩转换算法是基于 [Gifflen @Bitmap color reduction and GIF encoding](http://jiggawatt.org/badc0de/android/index.html#gifflen) 的.

通过对 Gifflen 的C++源码进行一些定制和修改,然后编译出native library（即 .so 文件）,然后打包到 APK 中.我们可以很容易的在Android系统上创建一个Gif动图. 本项目的NDK部分代码是采用CMake的方式进行构建的.Android Studio2.2版本之后已经对NDK编程有了很好的支持.体验下来感觉很棒.

---

## 使用方法

**1.添加动态链接库**

使用我编译好的[动态链接库文件](https://github.com/lchad/Gifflen-Android/tree/master/so), 支持 `arm-v8a`, `armaib`, `armabi-v7a`, `mips`, `mips64`, `x86`, `x86-64` 等7个平台,你可以根据自己的需要添加对应的native library到项目中.

**2.添加Gifflen工具类**

应用层的所有接口都在 [Gifflen.java](https://github.com/lchad/Gifflen-Android/blob/master/app/src/main/java/com/lchad/gifflen/Gifflen.java) 这个类中,你要做的就是在自己项目的`app/main/java`路径下创建`com/lchad/gifflen`文件夹(这个路径很重要一个字母都不能错),然后把Gifflen.java复制到这个特殊的路径下,注意,请一定要把Gifflen.java放到以上指定位置(即`your-project-name/app/main/java/com/lchad/gifflen`)下,否则会在运行时JNI会报错,找不到对应的jni方法.这一点需格外注意.

**3.配置读写存储的权限**
由于涉及到文件操作,需要事先获取Android的读写外部存储权限,在
 AndroidManifest.xml 中添加一下两行:
```
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```
如果是Android6.0及以上需注意动态申请权限.

```
 ActivityCompat.requestPermissions(MainActivity.this, new String[]{
                            Manifest.permission.WRITE_EXTERNAL_STORAGE,
                            Manifest.permission.READ_EXTERNAL_STORAGE},
                    REQUEST_PERMISSIONS);
```

**4.初始化工具类 Gifflen **
初始化采用Builder模式,使用起来十分清爽.
```
Gifflen mGiffle = new Gifflen.Builder()
                        .color(mColor)	 //色域范围是2~256,且必须是2的整数次幂.
                        .delay(mDelayTime) //每相邻两帧之间播放的时间间隔.
                        .quality(mQuality) //色彩量化时的quality值.
                        .width(500)	    //生成Gif文件的宽度(像素).
                        .height(500)	   //生成Gif文件的高度(像素).
                        .build();
```

**5.开始创建 Gif 图片**

- 从 File 列表创建
```
        List<File> files = getFileList();
        mGiffle.encode(320, 320, "target path", files);
        mGiffle.encode("target path", files); 
```

- 从 Uri 列表创建
```
        List<Uri> uris = getUriList();
        mGiffle.encode(context, 500, 500, uris);
        mGiffle.encode(context, "target path", uris);
```

- 从 TypeArray 创建
```
	 TypeArra mDrawableList = getResources().obtainTypedArray(R.array.source);
	 mGiffle.encode(MainActivity.this, "target path", 500, 500, mDrawableList);
	 mGiffle.encode(MainActivity.this, "target path", mDrawableList);
```

- 从 Bitmap 数组创建
```
        Bitmap[] bitmaps = getBitmaps();
        mGiffle.encode("target path", 500, 500, bitmaps);
        mGiffle.encode("target path", bitmaps);
	//注意:此时要考虑Bitmap的大小以及个数,否则可能会造成OOM.
```



- 从 drawable id 数组创建
```
	int[] drawableIds = new int[]{
                R.drawable.mengbi1,
                R.drawable.mengbi2,
                R.drawable.mengbi3};
        mGiffle.encode(context, "target path", 500, 500, drawableIds);
        mGiffle.encode(context, "target path", drawableIds);
```

以上五种创建方式都支持重载(本质上都是对Bitmap进行操作),宽度和高度在encode()的时候都可以缺省,这时会使用创建Gifflen时传入的值,如果创建Gifflen时仍然没有传值,则会使用一个默认的值 320.

## 示例Apk程序
[点我下载](https://fir.im/18z5)

## 源码(GitHub) 欢迎Star,提Issue, PR
[Gifflen-Android](https://github.com/lchad/Gifflen-Android)