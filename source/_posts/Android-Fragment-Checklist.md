
---
title: Android中关于Fragment使用的CheckList
date: 2016-08-09 00:07
categories: Android
toc: true
---

`Fragment`自Android3.0时代被推出,它 的出现一方面是为了缓解 `Activity` 任务过重的问题(大型项目中`Activity`会变得十分臃肿)，另一方面是为了处理在不同屏幕上 UI 组件的布局问题(适配平板)，而且它还提供了一些新的特性（例如 `Retainable`）来处理一些在 `Activity` 中比较棘手的问题,`Fragment`相对`Activity`来讲比较省资源,启动速度也相对较快(例如新版知乎就采用了一个`Activity`配合多个`Fragment`的方式,在使用的过程中能明显感觉到打开页面比之前快很多).但是`Fragment`也存在一个很严重的问题,那就是生命周期过于复杂,会出现一些莫名其妙的bug,像`FaceBook`,`Square`这些公司就对`Fragment`敬而远之.
<!--more-->
究竟`Fragment`到底是好是坏,大家也总是吵个不休,没有一个结论,但是我个人还是比较喜欢使用`Fragemnt`的,可能是受到了我的启蒙书籍`<<Android编程权威指南>>`(这本书是一本基础书,里面对`Fragment`极其的推崇,奉行的原则就是`AUF`（ Always Use Fragments）,所以这这本书对`Fragment`的讲解还是很不错的)的影响吧.所以在这里对`Fragment`应该注意的地方做一个总结.

1.在`Fragment`的`onCreateView()`方法中,我们要通过`LayoutInflater`来`inflate()`出一个布局(即`rootView`),但是在某些情况下(`ViewPager`随着页面滑动),这个`onCreateView()`方法会被调用很多次,这时我们最好对`rootView`做一下非空判断,否则多次重复执行`inflate()`操作,并没有任何意义,

```

public class TestFragment extends Fragment {  
    private View mRootView;  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {  
        if (null == mRootView) {  
            mRootView = inflater.inflate(layout_id, container, false);  
        }  
          
        return mRootView;  
    }  
}  
```

2.`Fragment`中可以通过`saveInstanceState()`的方式来保存`Fragment`的状态

```
     /** 
     * 从服务端拉取下来的数据 
     */  
    private String serverData;  
  
    @Override  
    public void onSaveInstanceState(Bundle outState) {  
        super.onSaveInstanceState(outState);  
  
        outState.putString("data", serverData);  
    }  
  
    @Override  
    public void onActivityCreated(Bundle savedInstanceState) {  
        super.onActivityCreated(savedInstanceState);  
        serverData = savedInstanceState.getString("data");  
    }

``` 

3.Activities的生命周期都是由ActivityManager来管理的,Fragments的生命周期都是由托管Fragments的Activity来管理的.

4.向Fragment传递参数的最佳方式为使用setArguments()的方法，这样可以避免在横竖屏切换的时候Fragment自动调用自己的无参构造函数，导致数据丢失。


```

private static final String EXTRA_STRING = "extra_string";  
  
    public static Fragment newInstance(String data) {  
        Bundle args = new Bundle();  
        args.putString(EXTRA_STRING, data);  
        TestFragment fragment = new TestFragment();  
        fragment.setArguments(args);  
        return fragment;  
    }

```

5.不应该在Fragment内管理View的状态,View本身有和Activity类似的onSaveInstanceState()方法来保存View的状态,如果有自定义View存在的话,View状态的保存应该在自定义View的内部处理,而不应该使用Fragment来参与.

6.在一个Activity中通过FragmentManager来添加管理Fragment的最佳实践:


```

 public class MainActivity extends Activity {  
        private TestFragment mTestFragment;  
  
        @Override  
        protected void onCreate(Bundle savedInstanceState)  
        {  
            super.onCreate(savedInstanceState);  
            setContentView(R.layout.activity_main);  
  
            FragmentManager fm = getSupportFragmentManager();  
            mTestFragment = (TestFragment) fm.findFragmentById(R.id.id_fragment_container);  
  
            if(mTestFragment == null) {  
                mTestFragment = new TestFragment();  
                fm.beginTransaction().add(R.id.id_fragment_container,mTestFragment).commit();  
            }  
  
        }  
  
    }


```

第13行对`fragment`做了一下非空判断,原因:当`Activity`因为配置发生改变(旋转屏幕)或内存不足被系统杀死,造成重建时,我们的`Fragment`会被保存下来,但是会创建新的`FragmentManager`,新的`FragementManager`会首先去获取保存下来的`fragment`队列,重建`Fragment`队列.所以我们在这里做一下判断,从而恢复之前的状态.防止`Fragment`被重复创建.

`R.id.id_fragement_container`在这里有两个意义:①告知`fragmentManager`此`Fragment`的位置②此`fragment`的唯一标识.

7.`FragmentTransaction#commit`函数是异步执行的（其把本次transaction的所有操作添加到消息队列里）,所以某些情况下,会有一些奇怪的现象产生,这也是值得我们注意的地方.所幸Android也提供了`FragmentManager#executePendingTransactions()`方法，强制同步执行。

8.在使用`FragmentManager`的`add()`,`replace()`方法来管理`Framgment`的时候,可能会遇到`Fragment`重叠显示的情况,解决的方法是在调用`add()`,`remove()`方法的时候,一定要选择带`Tag`参数的那个方法.

9.Google官方推荐使用`DialogFragment`的方式来管理对话框,它的好处是在屏幕旋转的时候,对话框不会消失,而`AlertDialog`在这种情况下就会消失掉.

10.`Fragment`有一个`retainInstance`的属性,我们可以在`onCreate()`方法中,通过调用`setRetainInstance()`方法来设置它的值.`retainInstance`默认为`false`.此时,当屏幕旋转的时候,fragment会随着托管它的`Activity`一起销毁并重建,但是调用`setRetainInstance(true)`之后,可以保留`Fragment`,已保留的fragment不会随activity一起被销毁。相反，它会被一直保留并在需要时原封不动的传递给新的activity.对于已保留的fragment实例，其全部实例变量的值也将保持不变，因此可放心继续使用。还需要格外注意的一点是:只有当activity因设备配置发生改变被销毁时， fragment才会短时间处于被保留状态。如果activity是因操作系统需要回收内存而被销毁，则所有被保留的fragment也会被随之销毁。

11.我们可以方便的通过`ViewPager+Fragment+PagerAdapter`的方式实现左右滑动切换界面的效果,其中`PagerAdapter`有两个子类,`FragmentPagerAdapter`和`FragmentStatePagerAdapter`.它们之间有点细微的区别:
- `FragmentPagerAdapter`：对于不再需要的fragment，选择调用detach方法，仅销毁视图，并不会销毁fragment实例。

- `FragmentStatePagerAdapter`：会销毁不再需要的fragment，当当前事务提交以后，会彻底的将fragmeng从当前Activity的FragmentManager中移除，state标明，销毁时，会将其`onSaveInstanceState(Bundle outState)`中的bundle信息保存下来，当用户切换回来，可以通过该bundle恢复生成新的fragment，也就是说，你可以在`onSaveInstanceState(Bundle outState)`方法中保存一些数据，在onCreate中进行恢复创建。


所以,我们可以根据`ViewPager`的页数来灵活的选择自己应该使用Adapter,如果页数比较多的话,优先使用`FragmentStatePagerAdapter`,反之,则应该使用`FragmentPagerAdapter`.