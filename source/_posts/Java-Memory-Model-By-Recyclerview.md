---
title: 从RecyclerView的一个bug看Java内存模型
date: 2016-03-15 23:49
categories: Java
toc: true
cover: /img/android.jpg
---

今天在写一个RecyclerView的Demo,大致的状况就是请求网络分页加载数据,解析成bean然后填到列表里,展示瀑布流出来.但是写完之后列表却一直都是空的,但是断点里也能清楚地看到数据解析成功,被设置到了adapter中,反复看了好久,最后还是在同事的指点下,跳出了坑,其实问题是Java引用的问题,我先把错误的代码贴出来.大家观摩一下.
<!--more-->
Adapter:


public class RecyclerViewAdapter extends RecyclerView.Adapter<PostViewHolder> {  
  
    private Context context;  
    public List<PostsEntity> list = new ArrayList<>();  
  
    public RecyclerViewAdapter(Context context, List<PostsEntity> list) {  
        this.context = context;  
        this.list = list;  
    }  
  
    @Override  
    public PostViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {  
        PostViewHolder holder = new PostViewHolder(LayoutInflater.from(  
                context).inflate(R.layout.post_detail_item, viewGroup, false));  
        return holder;  
    }  
  
    @Override  
    public void onBindViewHolder(PostViewHolder viewHolder, int position) {  
        //填充界面等操作...  
    }  
  
    @Override  
    public int getItemCount() {  
        return list == null ? 0 : list.size();  
    }  
  
} 
 
```

Activity:

```
public class HomeActivity extends AppCompatActivity {  
  
    @Bind(R.id.recyclerview)  
    RecyclerView recyclerview;  
  
    private List<PostsEntity> originalList;  
    private RecyclerViewAdapter adapter;  
  
    @Override  
    protected void onCreate(@Nullable Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_home);  
        ButterKnife.bind(this);  
        init();  
        loadData();  
    }  
  
    private void init() {  
        list = new ArrayList<>();  
        adapter = new RecyclerViewAdapter(HomeActivity.this, originalList);  
        LinearLayoutManager mManager = new LinearLayoutManager(HomeActivity.this, LinearLayoutManager.VERTICAL, false);  
        recyclerview.setAdapter(adapter);  
        recyclerview.setLayoutManager(mManager);  
    }  
  
    private void loadData() {  
        OkHttpUtils.get().url(URL).build().execute(new StringCallback() {  
            @Override  
            public void onError(Call call, Exception e) {  
            }  
  
            @Override  
            public void onResponse(final String response) {  
                Gson gson = new Gson();  
                final PostBean bean = gson.fromJson(response, PostBean.class);  
                assembleData(bean);  
            }  
        });  
    }  
  
    private void assembleData(PostBean bean) {  
        originalList = bean.getPosts();<span style="white-space:pre"> </span>//问题就出现在了这里!!!  
        adapter.notifyDataSetChanged();  
    }  
}  
```

好了直接看出问题的可以打我了哭哭,没看出来的往下看,听我缩.

讲解原因之前,先介绍一下Java的内存模型,其实在自己刚开始学习Java的时候(一年多之前)记过这样一条笔记:


但是,如果当初能够真正的理解这些概念,也不至于有今天的事情,还是那句话:纸上得来终觉浅,绝知此事要躬行.
上面的话大致可以这样理解,Book book = new Book();这句话其实创建了两个变量:book为引用变量,存储在堆栈中;new出来的Book()为值变量,存在堆中.book的值就是Book()的地址.这是分析上面问题的先决条件.

下面再上一张我自己画的关于这段出错程序的图,



上图一共分为6部,三个区域(从左至右为Activity的栈空间, Java的堆空间,Adapter的栈空间),我们一步一步看.

1.在Activity的onCreate()方法中我们初始化一个空的originalList,用来存放接下来从服务端获取的bean.

2.我们在new RecyclerViewAdapter的时候,把originalList以构造方法参数的形式传进去.

3.在Adapter的构造方法内部,this.list = list;这一行就是第三步.

4.在loadData()方法中我们请求到了真正的数据bean.getPosts(),这是一个以PostEntity为元素的列表.

5和6.在我们出错的那一行,originalList = bean.getPosts();此时original作为一个引用变量,它的值变了,不再指向上面的那块内存区域,从而指向下面的那块内存区域.

可是这时候我们的adapter内的list还是指向原来的内存区域,那块区域里还是空的,自然RecyclerView就是空的了,不展示任何东西.所以问题症结所在也就一目了然了.

下面贴一下正确的做法:

private void assembleData(PostBean bean) {  
       list.clear();  
       list.addAll(bean.getPosts())  
       adapter.notifyDataSetChanged();  
   }  


最后要做一下总结,出现这个问题还是因为自己的基础不过关,当初是记过笔记,但大多还是流于表面形式,没有在编码实践中有所体会.正所谓"基础不牢,地动山摇",现在踩得每一个坑都是自己以前埋下的.