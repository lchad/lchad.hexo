---
title: 闭包中一个不易察觉的陷阱
date: 2015-08-24 15:52
categories: Python
toc: true
---
`Python`语言是支持函数式编程的,我们可以在一个函数的函数体中定义另一个完整的函数,甚至返回这个函数.在函数内部定义的函数和外部定义的函数是相同的,唯一的区别就是在函数内部定义的函数是不能被外部访问的.
<!--more-->
下面的一段代码定义了两个函数`f()` 和 `g()`,在函数`f()`中把`g()`作为返回值返回,这里的`g()`你可以参考`C语言`中函数的指针来进行理解,它们两个几乎在语义和用法上几乎完全相同(C语言中函数名即为`指向函数的指针`).

```
def g():  
    print 'g()...'  
  
def f():  
    print 'f()...'  
    return g  
```

我们把函数`g()`的定义移到`f()`内部,这样除了`f()`之外的区域就无法调用`g()`,代码如下:


```

def f():  
    print 'f()...'  
    def g():  
        print 'g()...'  
    return g  
```

但是有一种情况我们是无法把函数`g()`移动到函数`f()`外面的,请观察一下如下代码:


```
list = [1, 2, 3, 4, 5]  
  
def f(list):  
    def g():  
        return sum(list)  
    return g  
```

list是一个列表,作为参数传进函数`f()`中,此时我们发现无法把函数`g()`移到函数`f()`的外面,因为函数`g()`的功能是计算list中所有元素值的和,必须依赖于函数`f()`的参数;如果函数`g()`不在函数`f()`里面,则无法取得数据进行计算,这样我们就引出了闭包的概念.


**闭包(Closure)**:

```
内层函数引用了外层函数的变量(包括它的参数),然后返回内层函数的情况,这就是闭包.
```

接下来就要重点介绍本文标题中提到的那个不易察觉的陷阱了.

观察如下代码:

```

# 希望一次返回3个函数，分别计算1x1,2x2,3x3:  
def count():  
    fs = []  
    for i in range(1, 4):  
        def f():  
             return i*i  
        fs.append(f)  
    return fs  
  
f1, f2, f3 = count()  
  
print f1(), f2(), f3()  
```

这里的`count()`函数是通过函数的闭包来返回一个包含三个函数的列表,分别是计算1x1的函数, 计算2x2的函数, 计算3x3的函数,那么我们分别调用执行`f1()`函数, `f2()`函数, `f3()`函数的执行结果是多少呢?会不会是我们期望的1 4 9 呢?您不妨自己在机器上运行一下这段代码,如果嫌麻烦的话可以看一下我的运行结果截图:
![](http://upload-images.jianshu.io/upload_images/174711-a7a2d8c05e254c15?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

竟然是三个9!!! 第一次遇到这个问题其实我是拒绝滴,难道是加了特效么,我**duang**的一下就迷惑了

翻了翻书才知道这是Python语言的特性所致.当`count()`函数返回三个函数时,这三个变量所引用的变量i的值已经变成了3,因为在返回的时候三个函数并没有被调用,所以此时它们并没有及时计算它们对应的i乘以i的值,等到三个函数都返回时,然后调用三个函数,此时i的值已经为3,计算i乘以i的值自然就都是9了.

所以在返回闭包的情况下,我们一定要注意的一点就是:
**返回函数千万不要引用任何一个循环变量,或者在之后会发生改变的变量.**

当然对于这种情况我们还是有解决方法的.我们在内层函数`f()`内再定义一个内层函数`g()`,用这个函数的参数绑定循环变量的当前值,这样的话,无论循环变量之后如何改变,每一次循环中的循环变量i的值就都保存在了第三层函数`g()`中,如此得到了我们期望的输出结果.

```
#coding:utf-8  
__author__ = 'chad'  
# 希望一次返回3个函数，分别计算1x1,2x2,3x3:  
def count():  
    fs = []  
    for i in range(1, 4):  
        def f(j):  
            def g():  
                return j*j  
            return g  
        fs.append(f(i))  
    return fs  
  
f1, f2, f3 = count()  
  
print f1(), f2(), f3()  
```

程序的执行结果如下:
![](http://upload-images.jianshu.io/upload_images/174711-2d303179bed99f01?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果嫌这段代码过于臃肿,可以考虑使用lambda表达式进行缩减.