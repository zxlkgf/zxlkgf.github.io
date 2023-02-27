# SpringBoot项目-收藏管理

# 8 收藏管理

## 8.1 创建实体类

```sql
CREATE TABLE t_favorites(
fid INT PRIMARY KEY AUTO_INCREMENT COMMENT '收藏商品在数据表的id',
uid INT COMMENT '归属的用户id',
pid INT COMMENT '归属的商品id',
image VARCHAR(255) COMMENT '商品图片保存地址',
price BIGINT COMMENT '商品的价格',
title VARCHAR(255) COMMENT '商品的标题',
sell_point VARCHAR(255) COMMENT '商品的卖点',
status INT COMMENT '商品的状态'
); 
```

## 8.2 创建实体类

```java
/**
 * @author zxl
 * @description 收藏实体类
 * @date 2022/11/6
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Favorite extends BaseEntity {
    private Integer fid;
    private Integer uid;
    private Integer pid;
    private String image;
    private Long price;
    private String title;
    private String sellPoint;
    private Integer status;
}
```

---

## 8.3 加入收藏

### 8.3.1 后端-持久层

---

#### 1.编写sql

```sql
//插入收藏
insert into t_favorite(除了fid)values(值)
//判断是否存在该收藏
SELECT * FROM t_favorite WHERE uid = ? AND pid = ?
```

---

#### 2.编写Mapper接口

```java
    /**
     * 根据pid和uid查询收藏商品是否在库
     * @param uid
     * @param pid
     * @return
     */
    Favorite findFavoriteByPidAndUid(Integer uid,Integer pid);

    /**
     * 插入收藏
     * @param favorite 收藏数据
     * @return
     */
    Integer addFavorite(Favorite favorite);
```

---

#### 3.编写映射文件

```xml
<mapper namespace="com.zxl.store.mappers.FavoriteMapper">
<!--    Integer addFavorite(Favorite favorite);-->
    <insert id="addFavorite" parameterType="com.zxl.store.entity.Favorite" useGeneratedKeys="true" keyProperty="fid">
        INSERT INTO t_favorite(
            uid,
            pid,
            image,
            price,
            title,
            sell_point,
            status
        )
        VALUES (
            #{uid},
            #{pid},
            #{image},
            #{price},
            #{title},
            #{sellPoint},
            #{status}
        )
    </insert>
<!--    Favorite findFavoriteByPidAndUid(Integer uid,Integer pid);-->
    <select id="findFavoriteByPidAndUid" resultType="com.zxl.store.entity.Favorite">
        SELECT * FROM t_favorite WHERE uid = #{uid} AND pid = #{pid}
    </select>
</mapper>
```

---

#### 4.单元测试

---

### 8.3.2 后端-业务层

#### 1.编写异常

```java
//收藏已经存在异常
public class FavoriteExistException extends ServiceException {}
```

----

#### 2.编写业务层抽象方法

```java
/**
 * @author zxl
 * @description 收藏业务层的接口
 * @date 2022/11/6
 */
public interface IFavoriteService  {

    /**
     * 插入收藏
     * @param uid
     * @param pid
     */
    Integer addFavorite(Integer uid,Integer pid);
}
```

#### 3.编写实现逻辑

```java
/**
 * @author zxl
 * @description 收藏接口的业务层实现类
 * @date 2022/11/6
 */
@Service
public class IFavoriteServiceImpl implements IFavoriteService {
    @Autowired(required = false)
    private FavoriteMapper favoriteMapper;
    @Autowired
    private IProductService productService;

    @Override
    public Integer addFavorite(Integer uid, Integer pid) {
        //先判断是否已经存在收藏
        Favorite res = favoriteMapper.findFavoriteByPidAndUid(uid, pid);
        //判断结果
        if(res!=null){
            throw new FavoriteExistException("该收藏品已经存在！");
        }
        //商品不存在的话，执行下一步
        Favorite favorite = new Favorite();
        //根据pid查询信息
        Product p = productService.findProductById(pid);
        //加入数据
        favorite.setUid(uid);
        favorite.setPid(pid);
        favorite.setImage(p.getImage());
        favorite.setPrice(p.getPrice());
        favorite.setTitle(p.getTitle());
        favorite.setSellPoint(p.getSellPoint());
        favorite.setStatus(p.getStatus());
        //执行插入
        Integer row = favoriteMapper.addFavorite(favorite);
        //判断插入结果
        if(row!=1)throw new InsertException("插入收藏时出现未知错误");
        System.out.println(favorite.getFid());
        return favorite.getFid();
    }
}
```

---

#### 4.单元测试

---

### 8.3.3 后端-控制层

#### 1.处理异常

将异常交给全局处理

----

#### 2.设计请求

请求路径：/favorite/addFavorite

请求类型：HttpSession session,Integer pid

请求方式：post

响应类型：JsonResult< Integer>

---

#### 3.处理请求编写控制类

```java
/**
 * @author zxl
 * @description 添加收藏的控制类
 * @date 2022/11/6
 */
@RestController
@RequestMapping("/favorite")
public class FavoriteController extends BaseController {
    @Autowired
    private IFavoriteService favoriteService;

    @RequestMapping(value = "/addFavorite",method = RequestMethod.POST)
    public JsonResult<Integer> addFavorite(Integer pid, HttpSession session){
        //uid
        Integer uid = getUserIdFromSession(session);
        //执行插入
        Integer fid = favoriteService.addFavorite(uid, pid);
        return new JsonResult<>(OK,fid);
    }
}
```

---

#### 4.前端页面

```javascript
                //绑定收藏事件
                $("#btn-add-to-collect").click(function () {
                    if (confirm("确定要将此商品加入收藏吗？")) {
                        $.ajax({
                            url: "/favorite/addFavorite",
                            type: "post",
                            data: {"pid": pid},
                            dataType: "json",
                            success: function (res) {
                                if (res.state == 200) {
                                    alert("收藏成功！");
                                } else {
                                    alert(res.message);
                                }
                            },
                            error: function (err) {
                                alert("服务器出现错误，加入购物车失败！")
                            }
                        })
                    }
                });
```

---

## 8.4 显示收藏

### 8.4.1 后端-持久层

---

#### 1.编写sql

```sql
//根据用户uid和收藏商品的状态去查找
SELECT * FROM t_favorite WHERE uid = #{uid} AND status=#{status}
```

---

#### 2.编写Mapper接口抽象方法

```java
    /**
     * 按照uid和status查找用户的收藏列表
     * @param uid 用户的id
     * @param status 商品状态
     * @return 返回结果集合
     */
    List<Favorite> findFavoritesByUidAndStatus(Integer uid,Integer status);
```

---

#### 3.编写映射文件

```xml
<!--    List<Favorite> findFavoritesByUidAndStatus(Integer uid,Integer status);-->
    <select id="findFavoritesByUidAndStatus">
        SELECT * FROM t_favortie WHERE uid = #{uid} AND status = 1;
    </select>
```

---

#### 4.单元测试

---

### 8.4.2 后端-业务层

---

#### 1.编写异常

---

#### 2.编写业务层抽象方法

```java
    /**
     * 查找指定条件的收藏
     * @param uid 用户uid
     * @param pageNum 当前页码
     * @param pageSize 每页数据多少
     * @return
     */
    PageInfo<Favorite> findFavorite(Integer uid, Integer pageNum, Integer pageSize);
```

---

#### 3.编写实现逻辑

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

---

#### 4.单元测试

---

### 8.4.3 后端-控制层

----

#### 1.处理异常

---

#### 2.设计请求

请求路径：/favorite/findFavorites

请求类型：HttpSession session,Integer pageNum,Integer pageSize

请求方式：get

响应类型：JsonResult<Favorite>

---

#### 3.处理请求 编写控制方法

```java
    //查询
    @RequestMapping(value = "/findFavorites",method = RequestMethod.GET)
    public JsonResult<PageInfo<Favorite>> findFavorites(HttpSession session, Integer pageNum, Integer pageSize){
        Integer uid = getUserIdFromSession(session);
        PageInfo<Favorite> favorites = favoriteService.findFavorite(uid, pageNum, pageSize);
        return new JsonResult<>(OK,favorites);
    }
```

---

#### 4.前端页面

```javascript
        <script type="text/javascript">
            //为了查询设置的全局参数
            var pageNum = 0;
            var pageSize = 0;
            var prePage = 0;
            var nextPage = 0;
            var navigatepageNums = [];

            //根据分页条的选择的页数进行查询
            function PaginationListSelect(num,size) {
                showCollectProduct(num,size)
            }
            function addDataToList(res) {
                //在填充数据之前必须先将这两个div标签内的所有元素情况，不然会出现叠加情况
                $("#collectList").empty();
                $("#PaginationList").empty();
                let collectListStr = "";
                //获取分页数据中的数据数量
                let dataLength = res.data.list.length;
                //填充商品到页面
                for(let i = 0;i<dataLength;i++){
                    let favorite = res.data.list[i];
                    collectListStr = "<div class=\"col-md-3\">"
                            + "<div class=\"goods-panel\">"
                            + "<img src=.." + favorite.image + "collect.png" + "  class=\"img-responsive\" />"
                            + "<p>￥" + favorite.price  + ".00" + "</p>"
                            + "<div class=\"text-row-3\">"
                            + "<a href=product.html?pid=" + favorite.pid
                            + "><small>"
                            + favorite.title
                            + "</small></a></div>"
                            + "<span style='padding-right: 10px'>"
                            + "<a href='javascript:void(0)' onclick='CancelCollect(#{fid})'  class='btn btn-default btn-xs add-fav'><span class='fa fa-heart'></span>取消收藏</a>"
                            + "</span>"
                            + "<span style='padding-right: 10px'>"
                            + "<a href='javascript:void(0)' onclick='addCollectToCart(#{pid},#{price})' class=\"btn btn-default btn-xs add-cart\"><span class=\"fa fa-cart-arrow-down\"></span>加入购物车</a>"
                            + "</span></div></div>";

                    collectListStr = collectListStr.replaceAll("#{fid}",favorite.fid);
                    collectListStr = collectListStr.replaceAll("#{pid}",favorite.pid);
                    collectListStr = collectListStr.replaceAll("#{price}",favorite.price);
                    $("#collectList").append(collectListStr);
                }
                //重新填充分页条
                // 将数据返回的部分需要数据进行填充至全局参数
                pageNum = res.data.pageNum; //当前页
                pageSize = res.data.pageSize;  //每页显示数
                prePage =  res.data.prePage; //上一页
                nextPage = res.data.nextPage; //下一页
                navigatepageNums = res.data.navigatepageNums;//分页栏的数字

                let firstPage = "<a id='first' href='#' onclick='PaginationListSelect(prePage,pageSize)' style='padding-right: 8px'>上一页</a>";
                let lastPage = "<a id='end' href='javascript:void(0)' onclick='PaginationListSelect(nextPage,pageSize)' style='padding-right: 8px'>下一页</a>";
                let PaginationListStr = "";
                //判断是否是第一页
                if (res.data.isFirstPage){ //为true表示当前是第一页
                    firstPage = "<a id='first' href='javascript:void(0)' "
                            + "style='opacity: 0.2;padding-right: 8px;color: black'>上一页</a>";
                    PaginationListStr += firstPage;
                }else { //为false表示当前不是第一页
                    PaginationListStr += firstPage;
                }
                //填充分页的页码数
                for (let i = 0; i < navigatepageNums.length; i++) {
                    //当前页的页码
                    let nowNum = navigatepageNums[i];
                    if (nowNum === pageNum){ //相等表示i的次数和当前也相同，对页数显示做变化
                        PaginationListStr += "<a href='javascript:void(0)' " +
                                "style='padding-right: 8px;color: black' disabled='disabled'>" + "【"  + nowNum + "】" +"</a>"
                    }else {
                        PaginationListStr += "<a href='javascript:void(0)' onclick='PaginationListSelect(#{nowNum},pageSize)' " +
                                "style='padding-right: 8px'>" + nowNum +"</a>"
                    }
                    PaginationListStr = PaginationListStr.replaceAll("#{nowNum}",nowNum)

                }
                //判断是否是末页
                if (!res.data.isLastPage){ //取反为false表示当前是末页
                    PaginationListStr += lastPage;
                }else { //为true表示当前是末页
                    lastPage =  "<a id='end' href='javascript:void(0)' style='opacity: 0.2;padding-right: 8px;color: black'>下一页</a>"
                    PaginationListStr += lastPage;
                }
                //将拼接的str串插入指定id处
                $("#PaginationList").append(PaginationListStr);
            }

            function showCollectProduct(num,size){
                $.ajax({
                    url: "/favorite/findFavorites",
                    type: "get",
                    data: "pageNum=" + num + "&pageSize=" + size,
                    dataType: "json",
                    success: function (res) {
                        if(res.data.list.length !== 0){ //代表有数据
                            //从showPageDataIntoHtml.js中导入的方法
                            addDataToList(res)
                        }else { //代表没数据
                            alert("暂无收藏商品，先去逛逛吧！");
                        }

                    },
                    error : function (err) {
                        alert("服务器出现错误，查询收藏商品失败！")
                    }
                })
            }

            $(function () {
                //网页加载完成自动发送ajax请求查询第一页的收藏商品列表
                showCollectProduct(1,4);
            })
        </script>
```

---

## 8.5 删除收藏

---

### 8.5.1 后端-持久层

---

#### 1.编写sql

```sql
//取消收藏
DELETE FROM t_favorite WHERE fid = #{fid}
```

---

#### 2.编写Mapper抽象方法

```java
    /**
     * 删除收藏
     * @param fid 收藏id
     * @return
     */
    Integer deleteCollect(Integer fid);
```

---

#### 3.编写Mapper映射

```xml
<!--    Integer deleteCollect(Integer fid);-->
    <delete id="deleteCollect">
        DELETE FROM t_favorite WHERE fid = #{fid}
    </delete>
```

---

#### 4.单元测试

---

### 8.5.2 后端-业务层

---

#### 1.编写异常

删除异常已经定义

---

#### 2.编写业务层抽象方法

```java
    /**
     * 删除收藏
     * @param fid
     */
    void deleteCollect(Integer fid);
```

---

#### 3.编写实现逻辑

```java
    @Override
    public void deleteCollect(Integer fid) {
        Integer row = favoriteMapper.deleteCollect(fid);
        if(row!=1)throw new DeleteException("删除收藏是出现错误");
    }
```

---

#### 4.单元测试

---

### 8.5.3 后端-控制层

---

#### 1.处理异常

---

#### 2.设计请求

请求路径：/favorite/cancelFavorite

请求类型：Integer fid

请求方式：post

响应类型：JsonResult<Void>

---

#### 3.处理请求 编写控制方法

```java
    //删除
    @RequestMapping(value = "/cancelFavorite",method = RequestMethod.POST)
    public JsonResult<Void> cancelFavorite (Integer fid){
        favoriteService.deleteCollect(fid);
        return new JsonResult<>(OK);
    }
```

---

#### 4.前端页面

```javascript
            //取消收藏
            function CancelCollect(fid) {
                if(confirm("确定取消收藏吗？")){
                    $.ajax({
                        url:"/favorite/cancelFavorite",
                        type: "POST",
                        data:{"fid":fid},
                        dataType: "json",
                        success:function (res) {
                            if(res.state == 200){
                                alert("取消收藏成功");
                                location.reload();
                            }else{
                                alert("取消收藏失败");
                            }
                        }
                    });
                }
            }
```

____

## 8.6 加入购物车

已经被实现，只需要调用即可

---

### 8.6.1 前端页面

```javascript
			//加入购物车功能
			function addCollectToCart(pid,price){
				if (confirm("确定要将此商品加入购物车吗？")){
					$.ajax({
						url: "/cart/addCart",
						type: "post",
						data: {pid:pid,price:price,num:1},
						dataType: "json",
						success: function (res) {
							alert("已成功加入购物车，在购物车等您结算哟！")
						},
						error : function (err) {
							alert("服务器出现错误，加入购物车失败！")
						}
					})
				}
			}
```

---


