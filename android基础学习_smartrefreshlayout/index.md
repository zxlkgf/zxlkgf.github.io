# Android基础学习_SmartRefreshLayout


---

# 1.SmartRefreshLayout

简介:下拉刷新、上拉加载、二级刷新、淘宝二楼、RefreshLayout、OverScroll，Android智能下拉刷新框架，支持越界回弹、越界拖动，具有极强的扩展性，集成了几十种炫酷的Header和 Footer。  

[项目源地址](https://github.com/scwang90/SmartRefreshLayout)

--- 

# 2.使用

### 2.1 在build.gradle中添加如下

```xml
compile 'com.android.support:appcompat-v7:25.3.1'                   //必须 25.3.1 以上

// 注意：分包之后不会有默认的Header和Footer需要手动添加！还是原来的三种方法！
implementation  'io.github.scwang90:refresh-layout-kernel:2.0.6'      //核心必须依赖
implementation  'io.github.scwang90:refresh-header-classics:2.0.6'    //经典刷新头
implementation  'io.github.scwang90:refresh-header-radar:2.0.6'       //雷达刷新头
implementation  'io.github.scwang90:refresh-header-falsify:2.0.6'     //虚拟刷新头
implementation  'io.github.scwang90:refresh-header-material:2.0.6'    //谷歌刷新头
implementation  'io.github.scwang90:refresh-header-two-level:2.0.6'   //二级刷新头
implementation  'io.github.scwang90:refresh-footer-ball:2.0.6'        //球脉冲加载
implementation  'io.github.scwang90:refresh-footer-classics:2.0.6'    //经典加载
```

```xml
android.useAndroidX=true
android.enableJetifier=true
```

需要依赖 androidx.appcompat

```xml
implementation 'androidx.appcompat:appcompat:1.0.0'                 //必须 1.0.0 以上
```

---

### 2.2 在XML布局文件中添加 SmartRefreshLayout

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!--在Xml布局中引入SmartRefreshLayout-->
    <com.scwang.smart.refresh.layout.SmartRefreshLayout
        android:id="@+id/refreshLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <com.scwang.smart.refresh.header.ClassicsHeader
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
        <ListView
            android:id="@+id/List_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:overScrollMode="never"
            android:background="#fff" />
        <com.scwang.smart.refresh.footer.ClassicsFooter
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
    </com.scwang.smart.refresh.layout.SmartRefreshLayout>
</LinearLayout>
```

---

### 2.3 在 Activity 或者 Fragment 中添加代码

```java
RefreshLayout refreshLayout = (RefreshLayout)findViewById(R.id.refreshLayout);
refreshLayout.setRefreshHeader(new ClassicsHeader(this));
refreshLayout.setRefreshFooter(new ClassicsFooter(this));
refreshLayout.setOnRefreshListener(new OnRefreshListener() {
    @Override
    public void onRefresh(RefreshLayout refreshlayout) {
        refreshlayout.finishRefresh(2000/*,false*/);//传入false表示刷新失败
    }
});
refreshLayout.setOnLoadMoreListener(new OnLoadMoreListener() {
    @Override
    public void onLoadMore(RefreshLayout refreshlayout) {
        refreshlayout.finishLoadMore(2000/*,false*/);//传入false表示加载失败
    }
});
```

---

### 2.4 使用指定的 Header 和 Footer

#### 1.全局设置

```java
public class App extends Application {
    //static 代码段可以防止内存泄露
    static {
        //设置全局的Header构建器
        SmartRefreshLayout.setDefaultRefreshHeaderCreator(new DefaultRefreshHeaderCreator() {
                @Override
                public RefreshHeader createRefreshHeader(Context context, RefreshLayout layout) {
                    layout.setPrimaryColorsId(R.color.colorPrimary, android.R.color.white);//全局设置主题颜色
                    return new ClassicsHeader(context);//.setTimeFormat(new DynamicTimeFormat("更新于 %s"));//指定为经典Header，默认是 贝塞尔雷达Header
                }
            });
        //设置全局的Footer构建器
        SmartRefreshLayout.setDefaultRefreshFooterCreator(new DefaultRefreshFooterCreator() {
                @Override
                public RefreshFooter createRefreshFooter(Context context, RefreshLayout layout) {
                    //指定为经典Footer，默认是 BallPulseFooter
                    return new ClassicsFooter(context).setDrawableSize(20);
                }
            });
    }
}
```

注意：方法一 设置的Header和Footer的优先级是最低的，如果同时还使用了方法二、三，将会被其它方法取代

---

#### 2.XML布局文件指定

```xml
<com.scwang.smart.refresh.layout.SmartRefreshLayout
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/refreshLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#444444"
    app:srlPrimaryColor="#444444"
    app:srlAccentColor="@android:color/white"
    app:srlEnablePreviewInEditMode="true">
    <!--srlAccentColor srlPrimaryColor 将会改变 Header 和 Footer 的主题颜色-->
    <!--srlEnablePreviewInEditMode 可以开启和关闭预览功能-->
    <com.scwang.smart.refresh.header.ClassicsHeader
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="@dimen/dimenPaddingCommon"
        android:background="@android:color/white"
        android:text="@string/description_define_in_xml"/>
    <com.scwang.smart.refresh.footer.ClassicsFooter
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</com.scwang.smart.refresh.layout.SmartRefreshLayout>
```

注意：方法二 XML设置的Header和Footer的优先级是中等的，会被方法三覆盖。而且使用本方法的时候，Android Studio 会有预览效果

---

#### 3.Java代码设置

```java
final RefreshLayout refreshLayout = (RefreshLayout) findViewById(R.id.refreshLayout);
//设置 Header 为 贝塞尔雷达 样式
refreshLayout.setRefreshHeader(new BezierRadarHeader(this).setEnableHorizontalDrag(true));
//设置 Footer 为 球脉冲 样式
refreshLayout.setRefreshFooter(new BallPulseFooter(this).setSpinnerStyle(SpinnerStyle.Scale));
```

----

# 3.具体配置参数

参考一下地址,官方页面还有许多有趣的UI加载页面，可以尝试一下。

[github样例地址]([SmartRefreshLayout/art/md_property.md at master · scwang90/SmartRefreshLayout · GitHub](https://github.com/scwang90/SmartRefreshLayout/blob/master/art/md_property.md))

---

# 4.上拉刷新下拉加载

具体操作可以通过重写SmartRefreshLayout的onrefresh()或者onLoadMore()

```java
refreshLayout.setOnRefreshListener(new OnRefreshListener() {
        @Override
        public void onRefresh(@NonNull RefreshLayout refreshLayout) {
            Log.d(TAG,"onRefresh");
            //ListView或者RecycleView内元素加载或者刷新操作
            refreshData();
            //传入值为UI加载画面持续时间
            refreshLayout.finishRefresh(1000);//传入false表示刷新失败
        }
    });

refreshLayout.setOnLoadMoreListener(new OnLoadMoreListener(){
        @Override
        public void onLoadMore(@NonNull RefreshLayout refreshLayout) {
            Log.d(TAG,"onLoadMore");
            //ListView或者RecycleView内元素加载或者刷新操作
            addData();
            //传入值为UI加载画面持续时间
            refreshLayout.finishLoadMore(1000);//传入false表示刷新失败
        }
   });
```

