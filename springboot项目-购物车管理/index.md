# SpringBoot项目-购物车管理


## 6 购物车

## 6.1 创建数据表

```sql
CREATE TABLE t_cart (
    cid INT AUTO_INCREMENT COMMENT '购物车数据id',
    uid INT NOT NULL COMMENT '用户id',
    pid INT NOT NULL COMMENT '商品id',
    price BIGINT COMMENT '加入时商品单价',
    num INT COMMENT '商品数量',
    created_user VARCHAR(20) COMMENT '创建人',
    created_time DATETIME COMMENT '创建时间',
    modified_user VARCHAR(20) COMMENT '修改人',
    modified_time DATETIME COMMENT '修改时间',
    PRIMARY KEY (cid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

---

## 6.2 创建实体类

```java
/**
 * @author zxl
 * @description   购物车商品信息实体类
 * @date 2022/11/4
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Cart extends BaseEntity{
//    cid INT AUTO_INCREMENT COMMENT '购物车数据id',
//    uid INT NOT NULL COMMENT '用户id',
//    pid INT NOT NULL COMMENT '商品id',
//    price BIGINT COMMENT '加入时商品单价',
//    num INT COMMENT '商品数量',
//    created_user VARCHAR(20) COMMENT '创建人',
//    created_time DATETIME COMMENT '创建时间',
//    modified_user VARCHAR(20) COMMENT '修改人',
//    modified_time DATETIME COMMENT '修改时间',
    private Integer cid;
    private Integer uid;
    private Integer pid;
    private Long price;
    private Integer num;
}
```

---

## 6.3  加入购物车

---

### 6.3.1 后端-持久层

---

#### 1.编写sql语句

```sql
//1.向购物车插入数据
INSERT INTO t_cart (aid除外) values （值列表）
//2.当当前购物车内已经存在当前商品，则直接更换当前商品数量
UPDATE t_cart SET num = ? WHERE cid = ?
//3.插入和更新的操作是取决于购物车是否有该商品
//所有需要查询商品是否存在
SELECT * FROM t_cart WHERE uid = ? AND cid = ?
```

---

#### 2.编写Mapper接口和抽象方法

```java
/**
 * @author zxl
 * @description 购物车持久层的Mapper接口
 * @date 2022/11/4
 */
public interface CartMapper {

    /**
     * 插入Cart数据
     * @param cart 购物车数据
     * @return
     */
    Integer addCart(Cart cart);

    /**
     * 更新Cart内容
     * @param cid 购物车id
     * @param num 更新物品数量
     * @param modifiedUser 更新人
     * @param modifiedTime 更新时间
     * @return
     */
    Integer updateCartInfo(Integer cid, Integer num, String modifiedUser, Date modifiedTime);

    /**
     * 按照用户的uid和商品的pid查找某条购物车数据
     * @param uid 用户的uid
     * @param pid 商品的id
     * @return 返回购物车数据
     */
    Cart findCartByUidAndPid(Integer uid,Integer pid);
}
```

---

#### 3.编写Mapper接口的映射文件

```xml
<mapper namespace="com.zxl.store.mappers.CartMapper">
    <!--    Integer addCart(Cart cart);-->
    <insert id="addCart" useGeneratedKeys="true" keyProperty="cid">
        INSERT INTO t_cart(uid,pid,price,num,created_user,created_time,modified_user,modified_time)
        VALUES (#{uid},#{pid},#{price},#{num},#{createdUser},#{createdTime},#{modifiedUser},#{modifiedTime})
    </insert>
<!--    Integer updateCartInfo(Integer cid, Integer num , String modifiedUser, Date modifiedTime);-->
    <update id="updateCartInfo">
        UPDATE t_cart SET num = #{num},modified_user = #{modifiedUser},modified_time = #{modifiedTime}
        WHERE cid = #{cid}
    </update>

<!--    Cart findCartByUidAndPid(Integer uid,Integer pid);-->
    <select id="findCartByUidAndPid" resultType="com.zxl.store.entity.Cart">
        SELECT * FROM t_cart WHERE uid = #{uid} AND pid = #{pid}
    </select>
</mapper>
```

---

#### 4.单元测试

----

### 6.3.2 后端-业务层

#### 1.规划异常

---

#### 2.编写业务层接口和抽象方法

```java
    /**
     * 将商品加入购物车
     * @param uid 用户id
     * @param pid 商品id
     * @param num 商品数量
     * @param username  用户名
     */
    void addCart(Integer uid,Integer pid,Integer num,Integer username);
```

---

#### 3.编写抽象方法的具体实现逻辑

```java
/**
 * @author zxl
 * @description 购物车业务层接口的实现类
 * @date 2022/11/4
 */
@Service
public class ICartServiceImpl implements ICartService {
    @Autowired(required = false)
    private CartMapper cartMapper;


    @Override
    public void addCart(Cart cart, String createdUser, Date createdTime, String modifiedUser, Date modifiedTime) {
        //查询当前商品是否在购物车存在
        Integer pid =  cart.getPid();
        Integer uid =  cart.getUid();
        Cart destCart = cartMapper.findCartByUidAndPid(uid, pid);
        //判断查询结果
        if(destCart==null){//如果不存在
            //补全四个字段
            cart.setCreatedUser(createdUser);
            cart.setModifiedUser(modifiedUser);
            cart.setCreatedTime(createdTime);
            cart.setModifiedTime(modifiedTime);

            //执行插入操作
            Integer integer = cartMapper.addCart(cart);
            //判断插入结果
            if(integer!=1){
                throw new InsertException("插入购物车数据时遭遇未知异常");
            }
        }else{//表示该商品存在数据
            //取出查询的数据数量
            Integer destNum = destCart.getNum();
            //取出新添加产品的数量
            Integer cartNum = cart.getNum();
            //计算结果
            Integer num = destNum + cartNum;

            //设置需要更新的字段
            cart.setNum(num);
            cart.setModifiedUser(modifiedUser);
            cart.setModifiedTime(modifiedTime);

            //执行更新操作
            Integer row = cartMapper.updateCartInfo(cart);

            if(row != 1 ){
                throw new UpdateException("更新购物车数据是遭遇未知异常");
            }
        }
    }
}
```

---

#### 4.单元测试

---

### 6.3.3 后端控制层

#### 1.处理异常

插入异常和更新异常均已被处理

---

#### 2.设计请求

请求地址:/cart/add_cart

请求参数:Integer pid,Integer price,Integer num, HttpSession session

请求类型:post

响应类型:JsonResult<Void>

---

#### 3.处理请求 编写控制方法

```java
/**
 * @author zxl
 * @description 处理购物车请求的控制类
 * @date 2022/11/5
 */
@RestController
@RequestMapping("/cart")
public class CartController extends BaseController {
    @Autowired
    private ICartService cartService;

    @RequestMapping(value = "/add_cart",method = RequestMethod.POST)
    public JsonResult<Void> addCart(Integer pid,Integer price,Integer num, HttpSession session){
        //从session获取uid和username
        String username = getUsernameFromSession(session);
        Integer uid = getUserIdFromSession(session);
        Date currentTime = new Date();
        //设置参数
        Cart cart = new Cart();
        cart.setUid(uid);
        cart.setPid(pid);
        cart.setPrice(Long.valueOf(price));
        cart.setNum(num);
        System.out.println(cart);
        //执行插入操作
        cartService.addCart(cart,username,currentTime,username,currentTime);
        return new JsonResult<>(OK);
    }
}
```

---

#### 4. 前端页面

```javascript
            //将物品加入购物车
            function addProductToCart(){
                $("#btn-add-cart").click(function () {
                    if(confirm("确定要将此商品加入购物车吗？")){
                        let price = $("#product-price").text();
                        let num = $("#num").val();
                        alert(price + ":"+num);
                        $.ajax({
                            url:"/cart/add_cart",
                            type:"post",
                            data:{
                                "pid":pid,
                                "price":price,
                                "num":num
                            },
                            dataType: "json",
                            success:function (json) {
                                if(json.state==200){
                                    alert("已经成功加入购物车");
                                }else{
                                    alert("加入购物车失败");
                                }
                            },
                        });
                    }
                });
            }
```

---

## 6.4 展示购物车

### 6.4.1 后端-持久层

---

##### 1.创建多表查询的实体类

```java
/**
 * @author zxl
 * @description Cart表和Product表联合查询的结果映射实体类
 * @date 2022/11/5
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CartVo {
    private Integer cid;
    private Integer pid;
    private Integer uid;
    private Long price;
    private Integer num;
    private String title;
    private String image;
    private String realPrice;
}
```

---

#### 2. 编写sql语句

由于购物车展示需要显示:

        商品名称(product)

        单价(product,cart)

        数量(cart)

        金额(product)

```sql
SELECT c.cid,
       c.uid,
       c.pid,
       c.price,
       c.num,
       p.title,
       p.image,
       p.price AS realPrice
FROM t_cart c 
LEFT JOIN  t_product p
ON    c.pid=p.id
WHERE c.uid = #{uid}
ORDER BY c.created_time DESC
```

---

#### 3.编写对应的Mapper接口的抽象方法

```java
    /**
     * 按照用户uid 查询所有的购物车记录
     * @param uid 用户uid
     * @return 返回购物车集合
     */
    List<CartVo> findAllCartByUid(Integer uid);
```

---

#### 4.编写Mapper接口对于的映射文件

```xml
<!--    List<CartVo> findAllCartByUid(Integer uid);-->
    <select id="findAllCartByUid" resultType="com.zxl.store.vo.CartVo">
        SELECT
        c.cid,
        c.uid,
        c.pid,
        c.price,
        c.num,
        p.title,
        p.image,
        p.price AS realPrice
        FROM t_cart c
        LEFT JOIN  t_product p
        ON    c.pid=p.id
        WHERE c.uid = #{uid}
        ORDER BY c.created_time DESC
    </select>
```

---

#### 5.单元测试

---

### 6.4.2 后端-业务层

---

#### 1.异常处理

----

#### 2.编写业务层抽象方法

```java
    /**
     * 根据用户uid查询购物车数据
     * @param uid 用户uid
     * @return
     */
    List<CartVo> findAllCartByUid(Integer uid);
```

---

#### 3.编写业务层方法的实现

```java
    @Override
    public List<CartVo> findAllCartByUid(Integer uid) {
        return cartMapper.findAllCartByUid(uid);
    }
```

---

#### 4.单元测试

---

### 6.4.3 后端-控制层

---

#### 1.异常处理

---

#### 2.设计请求

请求地址：/cart/show_carts

请求参数：HttpSession session

请求类型：get

响应类型：JsonResult< List< Cart>>

---

#### 3.处理请求，编写控制方法

```java
    @RequestMapping(value = "/show_carts",method = RequestMethod.GET)
    public JsonResult<List<CartVo>> showCarts(HttpSession session){
        //获取uid
        Integer uid = getUserIdFromSession(session);
        //获取数据
        List<CartVo> data = cartService.findAllCartByUid(uid);
        //返回数据
        return new JsonResult<>(OK,data);
    }
```

---

#### 4.前端页面

```javascript
        <script type="text/javascript">
            function showCarts(){
                $.ajax({
                    url: "/cart/show_carts",
                    type: "get",
                    dataType: "json",
                    success: function (res) {
                        if (res.data.length !== 0){
                            //先清空列表
                            $("#cart-list").empty();
                            for (let i = 0; i < res.data.length; i++) {
                                let cart = res.data[i];
                                let idNum = i;
                                let image = ".." + cart.image + "collect.png";
                                let totalPrice = cart.price * cart.num
                                let str =  "<tr>"
                                        +"<td><input onclick='checkOne()' id=cid" + idNum + " name='cids' value=" + cart.cid + " type='checkbox' class='ckitem' /></td>"
                                        + "<td><img  src=" + image  + " class='img-responsive' /></td>"
                                        + "<td>" + cart.title + "</td>"
                                        + "<td>¥<span id="+ "goodsPrice" + idNum +">"+ cart.price + "</span></td>"
                                        + "<td>"
                                        + "<input id=" + "countRec" + idNum + " type='button' value='-' class='num-btn' onclick='ajaxProductCountRec(#{idNum})' />"
                                        + "<input  id=" + "goodsCount"+ idNum +  " type='text' size='2' readonly='readonly' class='num-text' value=" + cart.num + ">"
                                        + "<input id=" + "countAdd" + idNum + " class='num-btn' type='button' value='+' onclick='ajaxProductCountAdd(#{idNum})' />"
                                        + "</td>"
                                        + "<td><span id=" + "goodsCast" + idNum + ">￥" + totalPrice + "</span></td>"
                                        + "<td>"
                                        + "<input type='button' onclick='delCartItem(#{deletedId})' class='cart-del btn btn-default btn-xs' value='删除' />"
                                        + "</td>"
                                        + "</tr>"
                                //替换数字
                                str = str.replaceAll("#{idNum}",idNum)
                                str = str.replaceAll("#{deletedId}",cart.cid)

                                //在表格中插入数据
                                $("#cart-list").append(str)
                                // 计算商品总数量和总价格
                                totalNum += 1;
                                countPrice  = countPrice  + totalPrice;
                            }
                        }else{
                            str = "<tr><td colspan='12' style='font-weight: bold;color: red;padding: 20px;font-size: medium'>" +
                                    "购物车暂无商品，请先去添加商品</td></tr>"
                            $("#cart-list").empty().append(str)
                        }
                    },
                    error : function (err) {
                        alert("服务器出现错误，查询失败！")
                    }
```

---

## 6.5 删除商品

### 6.5.1 后端-持久层

---

#### 1.编写sql

```sql
DELETE FROM t_cart WHERE cid = #{cid}
```

---

#### 2.编写Mapper接口的抽象方法

```java
    /**
     * 按照购物车id 删除购物车内容物
     * @param cid 购物车id
     * @return 返回影响行数
     */
    Integer deleteCartByCid(Integer cid);
```

---

#### 3.编写Mapper接口的映射文件

```xml
<!--    Integer deleteCartByCid(Integer cid);-->
    <delete id="deleteCartByCid">
        DELETE FROM t_cart WHERE cid = #{cid}
    </delete>
```

---

#### 4.单元测试

---

### 6.5.2 后端-业务层

#### 1.处理异常

删除异常已经被定义

----

#### 2.定义业务层抽象方法

```java
    /**
     * 按照cid删除购物车数据
     * @param cid cid
     */
    void deleteCartByCid(Integer cid);
```

---

#### 3.定义抽象方法实现逻辑

```java
    @Override
    public void deleteCartByCid(Integer cid) {
        //删除
        Integer row = cartMapper.deleteCartByCid(cid);
        //判断删除结果
        if(row!=1){
            throw new DeleteException("购物车数据删除异常!");
        }
    }
```

---

#### 4.单元测试

---

### 6.5.3 后端-控制层

#### 1.处理异常

---

#### 2.请求设计

请求地址：/cart/delete_cart

请求参数：Integer cid

请求类型：post

响应类型：JsonResult<Void>

---

#### 3,处理请求，编写控制层方法

```java
    @RequestMapping(value = "/delete_cart",method = RequestMethod.GET)
    public JsonResult<Void> showCarts(Integer cid){
        cartService.deleteCartByCid(cid);
        return new JsonResult<>(OK);
    }
```

---

#### 4.前端页面

```javascript
            //给每个删除按钮绑定点击事件
            function delCartItem(cid){
                if (confirm("确定要删除这条商品吗？")){
                    $.ajax({
                        url: "/cart/delete_cart",
                        type: "post",
                        data: {"cid":cid},
                        dataType: "json",
                        success:function (res) {
                            alert("删除成功")
                            location.reload();
                        },
                        error:function (error) {
                            alert("删除失败")
                        }
                    })
                }
            }
```

----

## 6.6 购物车数目增减

---

### 6.6.1 数目增减-持久层

#### 1.编写sql

```sql
//根据cid更新用户商品数量
UPDATE t_cart SET num = #{num},modified_user = #{modifiedUser},
                        #modified_time = #{modifiedTime}
                        WHERE cid = #{cid}
//根据cid查询用户cart信息
SELECT * FROM t_cart WHERE cid = #{cid};
```

---

#### 2.编写Mapper接口抽象方法

```java
    /**
     * 按照cid查询Cart
     * @param cid
     * @return
     */
    Cart findCartByCid(Integer cid);

    /**
     * 按照cid增减购物车商品的数量
     * @param num   数量
     * @param cid   购物车id
     * @param modifiedUser 操作人
     * @param modifiedTime 操作时间
     * @return
     */
    Integer updateCartNumByCid(Integer num,Integer cid,String modifiedUser,Date modifiedTime);
```

---

#### 3.编写Mapper接口映射文件

```xml
<!--    Cart findCartByCid(Integer cid);-->
    <select id="findCartByCid" resultType="com.zxl.store.entity.Cart">
        SELECT * FROM t_cart WHERE cid = #{cid}
    </select>

<!--    Integer updateCartNumByCid(Integer num,Integer cid,String username,Date time);-->
    <update id="updateCartNumByCid">
        UPDATE t_cart SET num = #{num},modified_user = #{modifiedUser},modified_time = #{modifiedTime} 
                      WHERE cid = #{cid}
    </update>
```

---

#### 4.单元测试

----

### 6.6.2 数目增减-业务层

#### 1.处理异常

---

#### 2.编写业务层抽象方法

```java
    /**
     * 更新购物车商品数量
     * @param cid
     * @param num
     * @param modifiedUser
     */
    void updateCartNumByCid(Integer cid,Integer num,String modifiedUser);
```

---

#### 3.编写实现逻辑

```java
    @Override
    public void updateCartNumByCid(Integer cid, Integer num, String modifiedUser) {
        //查询购物车数据
        Cart res = cartMapper.findCartByCid(cid);
        if(res==null){
            throw new CartInfoNotExistsException("购物车数据不存在");
        }
        //添加
        Integer row = cartMapper.updateCartNumByCid(num, cid, modifiedUser, new Date());
        //判断
        if(row!=1){
            throw new UpdateException("更新购物车商品数目出现未知异常");
        }
    }
```

---

#### 4.单元测试

---

### 6.6.3 数目增减-控制层

#### 1.异常处理

---

#### 2. 设计请求

请求地址：/cart/update_num

请求参数：Integer cid,Integer num,HttpSession session

请求类型：post

响应类型：JsonResult< void>

---

#### 3.处理请求，编写控制类

```java
    @RequestMapping(value = "/update_num",method = RequestMethod.POST)
    public JsonResult<Void> updateNum(Integer cid,Integer num,HttpSession session){
        String modifiedUser = getUsernameFromSession(session);
        cartService.updateCartNumByCid(cid,num,modifiedUser);
        return new JsonResult<>(OK);
    }
```

---

#### 4. 前端页面

```javascript
            /*按加号数量增*/
            function addNum(num) {
                var n = parseInt($("#goodsCount"+num).val());
                $("#goodsCount"+num).val(n + 1);
                calcRow(num);
            }

            /*按减号数量减少*/
            function reduceNum(num) {
                var n = parseInt($("#goodsCount"+num).val());
                if (n == 0)
                    return;
                $("#goodsCount"+num).val(n - 1);
                calcRow(num);
            }

            //计算单行小计价格的方法
            function calcRow(num) {
                //取单价 parseFloat() 函数可解析一个字符串，并返回一个浮点数。
                var vprice = parseFloat($("#goodsPrice"+num).html());
                //取数量
                var vnum = parseFloat($("#goodsCount"+num).val());
                //小计金额
                var vtotal = vprice * vnum;
                //赋值
                $("#goodsCast"+num).html("¥" + vtotal);
            }

            //向服务器发送ajax请求减少用户购物车的商品数量
            function ajaxProductCountRec(num){
                reduceNum(num);
                let cid = $("#cid"+num).val();
                let updateNum = $("#goodsCount"+num).val()
                $.ajax({
                    url : "/cart/update_num",
                    type: "post",
                    dataType: "json",
                    data:{cid:cid,num:updateNum},
                    error: function () {
                        alert("增加失败，请等待攻城狮修复！！")
                    }
                })
            }

            //向服务器发送ajax请求增加用户购物车的商品数量
            function ajaxProductCountAdd(num){
                addNum(num)
                let cid = $("#cid"+num).val();
                let updateNum = $("#goodsCount"+num).val()
                $.ajax({
                    url : "/cart/update_num",
                    type: "post",
                    dataType: "json",
                    data:{cid:cid,num:updateNum},
                    error: function () {
                        alert("增加失败，请等待攻城狮修复！！")
                    }
                })
            }
```

---

## 6.7 显示勾选的购物车数据

### 6.7.1 后端-持久层

---

#### 1.编写sql

用户在购物车列表随机勾选相关商品，在点击结算按钮之后，跳转到结算页面，在这个页面中需要展示上个页面所勾选的购物车对应数据

```sql
SELECT c.cid,
       c.uid,
       c.pid,
       c.price,
       c.num,
       p.title,
       p.image,
       p.price AS realPrice
FROM t_cart c 
LEFT JOIN  t_product p
ON    c.pid=p.id
WHERE c.cid IN (?,?,?)
ORDER BY c.created_time DESC
```

---

### 2.编写Mapper接口抽象方法

```java
    /**
     * 按照cids查询购物车数据
     * @param cids 
     * @return
     */
    List<CartVo> findVoByCid(Integer[] cids);
```

---

#### 3.编写映射文件

```xml
<!--    List<CartVo> findVoByCid(Integer[] cids);-->
    <select id="findVoByCid" resultType="com.zxl.store.vo.CartVo">
        SELECT
        c.cid,
        c.uid,
        c.pid,
        c.price,
        c.num,
        p.title,
        p.image,
        p.price AS realPrice
        FROM t_cart c
        LEFT JOIN  t_product p
        ON    c.pid=p.id
        WHERE c.cid IN (
            <foreach collection="array" item="cid" separator=",">
                #{cid}
            </foreach>
        )
        ORDER BY c.created_time DESC
    </select>
```

---

#### 4 单元测试

----

### 6.7.2 后端-业务层

---

#### 1.编写异常

---

#### 2.编写业务层抽象方法

```java
    /**
     * 按照cids查询购物车数据
     * @param cids
     * @return
     */
    List<CartVo> getVoByCid(Integer[] cids);
```

---

#### 3.编写实现逻辑

```java
    @Override
    public List<CartVo> getVoByCid(Integer[] cids) {
        //查询数据
        List<CartVo> data = cartMapper.findVoByCid(cids);
        //传输数据
        return data;
    }
```

---

#### 4.单元测试

---

### 6.7.3 后端-控制层

#### 1.处理异常

---

#### 2.设计请求

请求地址：/cart/list

请求参数：Integer[] cids

请求类型：get

响应类型：JsonResult<List<CartVo>>

---

#### 3.处理请求，编写处理方法

```java
    @RequestMapping(value = "/list",method = RequestMethod.GET)
    public JsonResult<List<CartVo>> getCartCids(Integer[] cids){
        List<CartVo> data = cartService.getVoByCid(cids);
        return new JsonResult<>(OK,data);
    }
```

---

#### 4.前端页面

1.将cart.html页面中结算按钮属性变更成type=“submit”

2.orderConfirm.html页面中添加自动加载从上个页面中传递过来的cids数据，再去请求ajax中进行填充当前页面区域中

```javascript
        <script type="text/javascript">
            //展示从购物车界面选中的商品信息
            function showOrderItem(){
                data = location.search.substr(1);//截取地址栏url?后的第二个元素，即购物车商品的cid
                //记录商品总数和总价格
                let totalNum = 0;
                let countPrice = 0;
                //自动发送ajax请求查询url地址上的cid信息
                $.ajax({
                    url : "/cart/list",
                    type: "get",
                    data: data,
                    dataType: "json",
                    success:function(res){
                        if (res.state === 200){
                            //填充信息
                            $("#cart-list").empty()
                            for (let i = 0; i < res.data.length; i++) {
                                let str = "";
                                let cartVo = res.data[i]
                                let totalPrice = cartVo.price * cartVo.num
                                str = "<tr>"
                                        + "<td id=cid" + i + " hidden='hidden'>" + cartVo.cid + "</td>"
                                        + "<td><img src=.." + cartVo.image + "collect.png" + " class='img-responsive' /></td>"
                                        + "<td>" + cartVo.title + "</td>"
                                        + "<td>¥<span>" + cartVo.price + "</span></td>"
                                        + "<td id=num" + i +  " >" + cartVo.num + "</td>"
                                        + "<td><span>" + totalPrice + "</span></td>"
                                        + "</tr>"
                                $("#cart-list").append(str)
                                //计算商品的数量和总金额
                                totalNum = totalNum + cartVo.num
                                countPrice = countPrice + totalPrice;
                            }
                            $("#all-count").empty().html(totalNum)
                            $("#all-price").empty().html(countPrice)
                        }else{
                            location.href = "500.html"
                        }
                    },
                    error: function () {
                        location.href = "500.html"
                    }
                });
            }
            $(function () {
                //页面加载完成执行查找
                showOrderItem();
            });
```

---


