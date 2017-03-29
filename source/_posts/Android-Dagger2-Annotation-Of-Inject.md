---
title: 聊聊Dagger2的@Inject注解
date: 2017-03-18 23:48
categories: Android
toc: true
cover: /img/dagger.png
---
### @Inject
Dagger2中最重要的注解,也是大多数人最先接触到的一个.在学习Dagger2的过程中,也看过不少相关文章,却几乎没看到能对`@Inject`进行全面详细的介绍,大多数都是一笔带过(可能是因为它确实很"简单"),转而开始剖析 `@Component`  `@Module`  `@Scope` 等深入概念,所以今天这篇文章就说说这个Dagger2中最简单的注解.
<!--more-->
`Inject` 这个词的字面意思是 `注射,注入` ,动词属性,但是仔细想一下`@Inject`在整个Dagger2框架中做的事并不是进行注入操作,因为这是`Component`的职责,如果需要一个词来提炼`@Inject`的功能的话,我觉得 `标记`这个词是我能想到的最好的;关于`@Inject`有一点我需要强调一下,那就是`@Inject`不是`单一职责`的,它可以修饰`变量` `构造方法` `普通的方法`,而且在修饰以上三者的时候,所发挥的作用也是不同的,所以这对初学者很容易造成混乱,所以接下来我会对着三种情况进行详细的分析.

### 1.修饰变量
这也是最初级用到做多的场景,也很容易理解,代码如下:
```
public class DaggerActivity extends AppCompatActivity {
    
    @Inject DaggerPresenter mDaggerPresenter;
    
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dagger);
        //传统方式初始化DaggerPresenter
        //mDaggerPresenter = new DaggerPresenter();
        geDaggerComponent().inject(this);    //依赖就是这个时刻被注入的
        mDaggerPresenter.start();
        //......other code
    }
}

```

当`@Inject`修饰一个变量时,表示这个变量(`mDaggerPresenter`)的初始化需要Dagger2来帮我们自动完成.前提就是这个类(`DaggerPresenter`)的构造方法也要被`@Inject`来标记.

在我们的程序执行第9行之前,我们的依赖都是null,依赖注入的所有操作都是在这一行来进行的,而且还有一个限制,那就是这个被`@Inject`修饰的field不能是private的,这就要涉及到Dagger2的原理了.我们来看一下Dagger2生成的注入代码:

```

//This class is generated automatically by Dagger 2
public final class DaggerActivity_MembersInjector implements MembersInjector<DaggerActivity> {

    //...

    @Override
    public void injectMembers(DaggerActivity daggerActivity) {
        if (daggerActivity == null) {
            throw new NullPointerException("Cannot inject members into a null reference");
        }
        supertypeInjector.injectMembers(daggerActivity);
        daggerActivity.mDaggerPresenter = presenterProvider.get();
        daggerActivity.analyticsManager = analyticsManagerProvider.get();
    }
}
```

在编译的时候Dagger2会通过APT(Android Annotation Processor)扫描所有被注解的变量,生成的类 `DaggerActivity_MembersInjector` 会通过new的方式来产生对应的实例,这也就是注入的过程.



### 2.修饰构造方法
还是先上代码:
```
public class DaggerPresenter implements IDaggerPresenter {
    
    private DaggerDataSource mDaggerDataSource;

    //Inject标记构造方法
    @Inject
    public DaggerPresenter(DaggerDataSource daggerDataSource) {
        this.mDaggerDataSource = daggerDataSource;
    }

    @Inject
    public void setPresenter() {

    }

    public void start() {

    }
}

```
在DaggerPresenter中,我们可以看到 构造方法被`@Inject`标记了,在这里,`@Inject`还是作为一个标记的,但它的作用却是 提供实例,而且这个构造方法还可以是有参数的, 这个参数也是通过Dagger来提供的(也必须由Dagger提供),如果Dagger在编译期间没有扫描到 DaggerDataSource 被注解的构造方法,那么就会默认传递一个默认值null给DaggerPresenter的构造方法.


还有一种特殊的情况就是一些第三方的类,我们没有办法对它的源码进行修改,用`@Inject`来标记的方式行不通了,所以Dagger2给我们提供了另外一种方式提供类的实例,那就是`@provides`注解

代码如下

```
@Module
public class DaggerModule {
    private final ThirdPartyParam mThirdPartyParam;

    public DaggerModule(ThirdPartyParam thirdPartyParam) {
        this.mThirdPartyParam = thirdPartyParam;
    }    

    @Provides
    @PerActivity
    IMainPresenter provideThirdParty() {
        return new ThirdParty (mThirdPartyParam);
    }
}

```
在Module里,我们通过一个被`@Provides`注解修饰的方法,使用new的方式来产生ThirdParty的实例.但是ThirdParty构造方法需要的参数实例ThirdPartyParam,不能再由Dagger提供,需要我们自己提供.

### 3.修饰普通方法
其实`@Inject`还可以修饰普通方法的,这在我用了Dagger2很长一段时间之内都是不知道的.这个用法除了在官方文档,我还没有见过有什么文章介绍过,虽说这样的使用场景很少,但还是有必要介绍一下的.

还是先看代码:

```
public class DaggerPresenter {
    private final DaggerView mDaggerView;
    @Inject
    public DaggerPresenter(DaggerView daggerView) {
        mDaggerView = daggerView;
        Log.d("DaggerTAG", "construction method was called in " + System.currentTimeMillis());
    }

   //Inject标记普通方法
    @Inject
    public void methodOne() {
        Log.d("DaggerTAG", "methodOne was called in "+ System.currentTimeMillis());
    }

    //Inject标记普通方法
    @Inject
    public void methodTwo() {
        Log.d("DaggerTAG", "methodTwo was called in "+ System.currentTimeMillis());
    }
    public void start() {

    }
}
```

在DaggerPresenter中,有两个方法 `methodOne()` `methodTwo()`,他们都被`@Inject`所修饰,我们分别在构造方法和这两个方法的方法体里面打印出方法的名字和调用的时间,验证一下我们的观点.
当`@Inject`修饰一个普通方法的时候,它的作用是:当构造方法执行完毕之后,会立即执行这个方法,这个我通过log的方式做了一下验证,结果如下

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-8af9f820cee2988f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/174711-abb1d864f50e68a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们看到`methodOne()` 和 `methodTwo()` 都是在构造方法执行之后立刻被执行到.


既然有了这种用法,接下来的问题就是为什么需要这种用法呢?它的使用场景在哪里呢?

接下来我结合Google官方推出的 [Android Architecture](https://github.com/googlesamples/android-architecture) 开源项目来窥探一下它的使用场景.之前一直在看这个项目的源码,我觉得有一个处能够非常完美的解释`@Inject`的第三种职责,即`方法注入`,在`todo-mvp-dagger`这个分支上的`TaskPresenter`中,有一段代码我们可以看一下:

```

final class TasksPresenter implements TasksContract.Presenter {

    private final TasksRepository mTasksRepository;

    private final TasksContract.View mTasksView;

    private TasksFilterType mCurrentFiltering = TasksFilterType.ALL_TASKS;

    private boolean mFirstLoad = true;

    /**
     * Dagger strictly enforces that arguments not marked with {@code @Nullable} are not injected
     * with {@code @Nullable} values.
     */
    @Inject
    TasksPresenter(TasksRepository tasksRepository, TasksContract.View tasksView) {
        mTasksRepository = tasksRepository;
        mTasksView = tasksView;
    }

    /**
     * Method injection is used here to safely reference {@code this} after the object is created.
     * For more information, see Java Concurrency in Practice.
     */
    @Inject
    void setupListeners() {
        mTasksView.setPresenter(this);
    }
	
	//other code

```

在TaskPresenter中,我们可以看到`setupListeners()`方法是被`@Inject`修饰的,而且它只是一个普通的public方法,它的注释的第一句话非常的有价值:

```
Method injection is used here to safely reference {@code this} after the object is created.	
//翻译:在对象被创建之后,这里的方法注入是为了安全的引用`this`
```

具体的做法也就是下面这一行

```
    mTasksView.setPresenter(this);
```

在MVP架构中,我们需要让View层持有Presenter的引用,这个`setupListeners()`方法做的就是这件事:在Presenter被完全构建完之后,让mTasksView来持有 **被完全构建的this**.而且这是一个自动化的过程,并不需要我们来手动调用,所以如果还有其他类似的需求,我们都可以这样使用`@Inject`.

需要注意的是这种被`@Inject`修饰的普通方法是不应该有返回值,它的参数也是由Dagger2来提供.

以上就是我对`@Inject`浅薄的认识.