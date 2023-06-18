# Android基础学习_RecycleView简单使用


---

# 1.RecycleView
## 1.1 简介
RecyclerView 可以让您轻松高效地显示大量数据。您提供数据并定义每个列表项的外观，而 RecyclerView 库会根据需要动态创建元素。  
顾名思义，RecyclerView 会回收这些单个的元素。当列表项滚动出屏幕时，RecyclerView 不会销毁其视图。相反，RecyclerView 会对屏幕上滚动的新列表项重用该视图。这种重用可以显著提高性能，改善应用响应能力并降低功耗。  

---

## 1.2 实现方法

---

### 1.2.1 布局(省略)
使用LinearLayout包含一个RecyclerView即可，设置RecyclerView的id即可  

---

### 1.2.2 实现Adapter和ViewHolder
适配器Adapter，可以让开发者自定义recyclerview绑定的数据，自定义的Item的xml文件自定义每行的格式。  

---

#### 创建
1. 创建一个Adapter类(名字什么都可以，这里是RecyclerViewAdapter)  
2. Adapter类继承RecyclerView.Adapter<T>  
3. RecyclerView.Adapter内部的泛型定义为Adapter的内部类ViewHolder即可  
4. 内部添加私有成员变量Context和存储数据的List，并添加构造方法  

具体步骤如下:(下列例子所使用的的item格式,其内部仅包含一个TextView)  
```java
public class RecycleViewAdapter extends RecyclerView.Adapter<RecycleViewAdapter.ViewHolder> {
    private Context mContext;
    private ArrayList<String> fruitList;
    //构造方法
    public RecycleViewAdapter(Context context, ArrayList<String> list){
        this.mContext = context;
        this.fruitList = list;
    }

    @NonNull
    @Override
    public RecycleViewAdapter.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return null;
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        return null
    }

    @Override
    public int getItemCount() {
        return null
    }

    //自定义内部类ViewHolder
    public class ViewHolder extends RecyclerView.ViewHolder {
        public ViewHolder(@NonNull View itemView) {
            super(itemView);
        }
    }

}

```

---

##### onCreateViewHolder()
每当 RecyclerView 需要创建新的 ViewHolder 时，它都会调用此方法。此方法会创建并初始化 ViewHolder 及其关联的 View，但不会填充视图的内容，因为 ViewHolder 此时尚未绑定到具体数据。  
```java
    @NonNull
    @Override
    public RecycleViewAdapter.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View root = LayoutInflater.from(mContext).inflate(R.layout.recycle_list_item,parent,false);
        RecycleViewAdapter.ViewHolder mHolder = new ViewHolder(root);
        return mHolder;
    }

```

---

##### onBindViewHolder()
RecyclerView 调用此方法将 ViewHolder 与数据相关联。此方法会提取适当的数据，并使用该数据填充 ViewHolder 的布局。例如，如果 RecyclerView 显示的是一个名称列表，该方法可能会在列表中查找适当的名称，并填充 ViewHolder 的 TextView。  
```java
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        holder.tv_rv.setText(fruitList.get(position));
    }
```

---

##### getItemCount()
RecyclerView 调用此方法来获取数据集的大小。例如，在通讯簿应用中，这可能是地址总数。RecyclerView 使用此方法来确定什么时候没有更多的列表项可以显示。  

```java
    @Override
    public int getItemCount() {
        return fruitList.size();
    }
```

---

##### ViewHolder

```java
    public class ViewHolder extends RecyclerView.ViewHolder {
        TextView tv_rv;
        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            tv_rv = itemView.findViewById(R.id.tv_rv);
        }
    }
    
```

---

#### 使用
1. 在MainActivity内部创建一个存储数据的List，并通过RecyclerViewAdapter()的构造方法传入。  
2. 获取当前的布局管理器，并设置到RecyclerView中
3. 设置RecyclerView的Adapter
4. 如果需要进行线段分割可以使用RecyclerView.serItemDecoration进行分割。


```java
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_base);
fruitList = new ArrayList<>();
recyclerView  = findViewById(R.id.first_rv);
addFruit();
//
LinearLayoutManager layoutManager = new LinearLayoutManager(getApplicationContext());
//设置纵向或者横向
layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);
//添加切割线
recyclerView.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.HORIZONTAL));
//设置布局
recyclerView.setLayoutManager(layoutManager);
//适配器
adapter = new RecycleViewAdapter(getApplicationContext(),fruitList);
recyclerView.setAdapter(adapter);

```


### 补充

---

#### 1.关于LayoutManager  
RecyclerView 中的列表项由 LayoutManager 类负责排列。    
RecyclerView 库提供了三种布局管理器，用于处理最常见的布局情况：    
1. LinearLayoutManager 将各个项排列在一维列表中。  
2. GridLayoutManager 将所有项排列在二维网格中：  
   1. 如果网格垂直排列，GridLayoutManager 会尽量使每行中所有元素的宽度和高度相同，但不同的行可以有不同的高度。  
   2. 如果网格水平排列，GridLayoutManager 会尽量使每列中所有元素的宽度和高度相同，但不同的列可以有不同的宽度。  
3. StaggeredGridLayoutManager 与 GridLayoutManager 类似，但不要求同一行中的列表项具有相同的高度（垂直网格有此要求）或同一列中的列表项具有相同的宽度（水平网格有此要求）。其结果是，同一行或同一列中的列表项可能会错落不齐。  

---

#### 关于LayoutInflater
[参考地址](https://www.runoob.com/w3cnote/android-tutorial-layoutinflater.html)
---

##### Layout是什么鬼？
一个用于加载布局的系统服务，就是实例化与Layout XML文件对应的View对象，不能直接使用， 需要通过getLayoutInflater( )方法或getSystemService( )方法来获得与当前Context绑定的 LayoutInflater实例!  

---

#### LayoutInflater的用法

----

##### 获取LayoutInflater实例的三种方法：

```java
LayoutInflater inflater1 = LayoutInflater.from(this);  
LayoutInflater inflater2 = getLayoutInflater();  
LayoutInflater inflater3 = (LayoutInflater) getSystemService(LAYOUT_INFLATER_SERVICE); 
```
PS:后面两个其实底层走的都是第一种方法~

----

##### 加载布局的方法：
```text
public View inflate (int resource, ViewGroup root, boolean attachToRoot) 该方法的三个参数依次为:

①要加载的布局对应的资源id

②为该布局的外部再嵌套一层父布局，如果不需要的话，写null就可以了!

③是否为加载的布局文件的最外层套一层root布局，不设置该参数的话， 如果root不为null的话，则默认为true 如果root为null的话，attachToRoot就没有作用了! root不为null，attachToRoot为true的话，会在加载的布局文件最外层嵌套一层root布局; 为false的话，则root失去作用! 简单理解就是：是否为加载的布局添加一个root的外层容器~!
```

---

##### 通过LayoutInflater.LayoutParams来设置相关的属性:
比如RelativeLayout还可以通过addRule方法添加规则，就是设置位置：是参考父容器呢？ 还是参考子控件？又或者设置margin等等，这个由你决定~

---






```java

```
