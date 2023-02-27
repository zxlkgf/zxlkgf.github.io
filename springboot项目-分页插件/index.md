# SpringBoot项目-分页插件

# 分页插件学习
```
1.github:https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md

2.CSDN:https://blog.csdn.net/m0_48736673/article/details/124805124
```
---

## 1.使用步骤

### 1.在pom.xml添加如下依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>最新版本</version>
    <!--<version>1.2.3<version>-->
</dependency>
```

---

### 2.在springboot中添加部分配置

在application.properties中添加配置

```properties
#pagehelper配置
pagehelper.helper-dialect=mysql
pagehelper.reasonable=true
pagehelper.support-methods-arguments=true
pagehelper.params=count=countSql
```

### 3.注册一个PageHelper类

```java
    @Bean
    public PageHelper pageHelper() {
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();
        properties.setProperty("offsetAsPageNum", "true");
        properties.setProperty("rowBoundsWithCount", "true");
        properties.setProperty("reasonable", "true");
        properties.setProperty("dialect", "mysql");
        pageHelper.setProperties(properties);
        return pageHelper;
    }
```

---

### 4.业务层或者需要使用的层设置开启

```java
    @Override
    public PageInfo<Favorite> findFavorite(Integer uid, Integer pageNum, Integer pageSize) {
        //开启分页功能
        //pageNum是当前页,pageSize是每页显示的数据量
        PageHelper.startPage(pageNum,pageSize);
        //
        List<Favorite> favorites = favoriteMapper.findFavoritesByUidAndStatus(uid, 1);
        //
        PageInfo<Favorite> pageInfo = new PageInfo<>(favorites);
        return pageInfo;
    }
```

### 5.控制层返回查询结果即可

```java
    @RequestMapping(value = "/findFavorites",method = RequestMethod.GET)
    public JsonResult<PageInfo<Favorite>> findFavorites(HttpSession session, Integer pageNum, Integer pageSize){
        Integer uid = getUserIdFromSession(session);
        PageInfo<Favorite> favorites = favoriteService.findFavorite(uid, pageNum, pageSize);
        return new JsonResult<>(OK,favorites);
    }
```

---

## 部分参数的解释

```
1.dialect：默认情况下会使用 PageHelper 方式进行分页，
   如果想要实现自己的分页逻辑,可以实现
   Dialect(com.github.pagehelper.Dialect)接口
   然后配置该属性为实现类的全限定名称。

2.reasonable:分页合理化参数，默认值为false。当该参数设置为true时，
    pageNum<=0 时会查询第一页,pageNum>pages（超过总数时),会查询最后一页。
    默认false时，直接根据参数进行查询。
```

---

### pageInfo参数大全

```java
//当前页
    private int pageNum;

    //每页的数量
    private int pageSize;

    //当前页的数量
    private int size;

    //由于startRow和endRow不常用，这里说个具体的用法
    //可以在页面中"显示startRow到endRow 共size条数据"
    //当前页面第一个元素在数据库中的行号
    private int startRow;

    //当前页面最后一个元素在数据库中的行号
    private int endRow;
    //总记录数
    private long total;

    //总页数
    private int pages;

    //结果集(每页显示的数据)
    private List<T> list;

    //第一页
    private int firstPage;

    //前一页
    private int prePage;

    //是否为第一页
    private boolean isFirstPage = false;

    //是否为最后一页
    private boolean isLastPage = false;

    //是否有前一页
    private boolean hasPreviousPage = false;

    //是否有下一页
    private boolean hasNextPage = false;

    //导航页码数
    private int navigatePages;
    //导航页第一页
    private int navigateFirstPage;
    //导航页第二页
    private int navigateLastPage;

    //所有导航页号
    private int[] navigatepageNums;
```

