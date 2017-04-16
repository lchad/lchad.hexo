---
title: Android Studio创建Activity时关闭ConstrantLayout默认选项
date: 2017-04-16 19:38
categories: Android Studio
toc: true
---
Android Studio在三月份的时候发布了最新的``2.3.0``版本,其中一个点值得我们注意,那就是通过模板新建一个Empty Activity的时候,默认布局变成了``ConstrantLayout``,还会在module的build.gradle内添加它的依赖,这对于不习惯使用``ConstrantLayout``的我来说,实在是太痛苦了.

解决这个问题比较垃圾的方法是不再使用模板来新建Activity,而是新建一个Class,一个布局文件,在AndroidManifest.xml里添加对应的Activity的标签.

但是这种方式一点都不酷.于是就花了点时间研究下Android Studio的模板机制,顺利的把这个问题解决了.解决方法如下:
## Mac系统
xml布局模板文件的路径:
```
/Applications/Android Studio.app/Contents/plugins/android/lib/templates/activities/common/root/res/layout/simple.xml.ftl
```

布局模板文件的内容:
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
<#if hasAppBar && appBarLayoutName??>
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:showIn="@layout/${appBarLayoutName}"
</#if>
    tools:context="${relativePackage}.${activityClass}">

<#if isNewProject!false>
    <TextView
<#if includeCppSupport!false>
        android:id="@+idmple_text"
</#if>
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</#if>
</android.support.constraint.ConstraintLayout>
```
我们需要把它修改成如下:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="${relativePackage}.${activityClass}">

    <TextView
        android:id="@+id/sample_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

</RelativeLayout>
```

module下build.gradle文件的路径:

```
Applications/Android Studio.app/Contents/plugins/android/lib/templates/activities/common/recipe_simple.ftl
```
它的内容:
```
<recipe folder="root://activities/common">

<#if appCompat && !(hasDependency('com.android.support:appcompat-v7'))>
    <dependency mavenUrl="com.android.support:appcompat-v7:${buildApi}.+"/>
</#if>
    <dependency mavenUrl="com.android.support.constraint:constraint-layout:+" />

    <instantiate from="root/res/layout/simple.xml.ftl"
                 to="${escapeXmlAttribute(resOut)}/layout/${simpleLayoutName}.xml" />

<#if (isNewProject!false) && !(excludeMenu!false)>
    <#include "recipe_simple_menu.xml.ftl" />
</#if>
</recipe>
```
我们需要把他修改成如下:

```
<recipe folder="root://activities/common">

<#if appCompat && !(hasDependency('com.android.support:appcompat-v7'))>
    <dependency mavenUrl="com.android.support:appcompat-v7:${buildApi}.+"/>
</#if>

    <instantiate from="root/res/layout/simple.xml.ftl"
                 to="${escapeXmlAttribute(resOut)}/layout/${simpleLayoutName}.xml" />

<#if (isNewProject!false) && !(excludeMenu!false)>
    <#include "recipe_simple_menu.xml.ftl" />
</#if>
</recipe>

```

## Windows系统
对应的,在Windows系统上只是模板路径不一样而已,下面贴出Windows系统下两个模板文件的路径:

xml布局模板文件路径:
```
C:\Program Files\Android\Android Studio\plugins\android\lib\templates\activities\common\root\res\layout\simple.xml.ftl
```
module下的build.gradle模板文件路径:
```
C:\Program Files\Android\Android Studio\plugins\android\lib\templates\activities\common\recipe_simple.ftl
```

现成的模板文件可以在这个[GitHub项目](https://github.com/lchad/HookEmptyActivityTemplate)里下载.复制替换重启Android Studio应该就好了.


参考链接:
[月半](http://yueban.github.io/2016/08/29/Android%20Studio%20%E6%A8%A1%E6%9D%BF%E5%B0%8F%E7%BB%93/)
[鸿阳](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820397&idx=1&sn=dd26d1dc56a31bff805afbbf65e15d3d&scene=0#wechat_redirect)