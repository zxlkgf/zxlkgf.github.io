# SpringBoot项目-商品管理

# 5 商品管理

---

## 5.1 创建数据表

```sql
DROP TABLE IF EXISTS t_product;

CREATE TABLE t_product (
  id int(20) NOT NULL COMMENT '商品id',
  category_id int(20) DEFAULT NULL COMMENT '分类id',
  item_type varchar(100) DEFAULT NULL COMMENT '商品系列',
  title varchar(100) DEFAULT NULL COMMENT '商品标题',
  sell_point varchar(150) DEFAULT NULL COMMENT '商品卖点',
  price bigint(20) DEFAULT NULL COMMENT '商品单价',
  num int(10) DEFAULT NULL COMMENT '库存数量',
  image varchar(500) DEFAULT NULL COMMENT '图片路径',
  status int(1) DEFAULT '1' COMMENT '商品状态  1：上架   2：下架   3：删除',
  priority int(10) DEFAULT NULL COMMENT '显示优先级',
  created_time datetime DEFAULT NULL COMMENT '创建时间',
  modified_time datetime DEFAULT NULL COMMENT '最后修改时间',
  created_user varchar(50) DEFAULT NULL COMMENT '创建人',
  modified_user varchar(50) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


//具体录入信息参照githu
```

---

## 5.2 定义实体类

```java
/**
 * @author zxl
 * @description 商品的实体类
 * @date 2022/11/4
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product extends BaseEntity{
    private Integer id;
    private Integer categoryId ;
    private String itemType;
    private String title;
    private String sellPoint;
    private Long price;
    private Integer num;
    private String image;
    private Integer status;
    private String priority;
}
```

---

## 5.3 热销商品

### 5.3.1 商品排行-持久层

---

#### 1.编写sql语句

```sql
//查询热销商品的sql
SELECT * FROM t_product WHERE status = 1 ORDER BY priorty DESC LIMIT 0,4
```

---

#### 2.编写Mapper接口和抽象方法

```java
/**
 * @author zxl
 * @description 处理商品数据的Mapper接口
 * @date 2022/11/4
 */
public interface ProductMapper {
    /**
     * 按照priority查找热销前五的商品数据
     * @return 返回商品数据集合
     */
    List<Product> findHotList();
}
```

---

#### 3.编写Mapper接口的映射

```xml
<mapper namespace="com.zxl.store.mappers.ProductMapper">
<!--    List<Product> findHotList();-->
<select id="findHotList" resultType="com.zxl.store.entity.Product">
    SELECT * FROM t_product WHERE status = 1 ORDER BY priority DESC LIMIT 0,4
</select>
</mapper>
```

---

#### 4.单元测试

---

### 5.3.2 商品排行-业务层

---

#### 1.异常规划

没有异常    

---

#### 2.定义业务层接口和抽象方法

```java
/**
 * @author zxl
 * @description 处理商品的业务层接口
 * @date 2022/11/4
 */
public interface IProductService {
    /**
     * 按照priority查找热销前五的商品数据
     * @return 返回商品数据集合
     */
    List<Product> findHotList();
}
```

---

#### 3.编写业务层方法逻辑

```java
/**
 * @author zxl
 * @description 处理商品的业务层实现类
 * @date 2022/11/4
 */
public class IProductServiceImpl implements IProductService {
    @Autowired(required = false)
    private ProductMapper productMapper;

    @Override
    public List<Product> findHotList() {
        //查找数据
        List<Product> res = productMapper.findHotList();
        return res;
    }
}
```

---

#### 4.单元测试

---

### 5.3.3 商品排行-控制层

#### 1.异常处理

没有异常

---

#### 2.设计请求

请求路径：/product/hot_list

请求参数：无

请求类型：get

响应类型：JsonResult<List< Product>>

---

#### 3.处理请求，编写控制层方法

```java
/**
 * @author zxl
 * @description  处理商品相关请求的控制类
 * @date 2022/11/4
 */
@RestController
@RequestMapping("/product")
public class ProductController extends BaseController {
    @Autowired(required = false)
    private IProductService productService;

    /**
     *  返回热销商品
     * @return
     */
    @RequestMapping(value = "/host_list",method = RequestMethod.GET)
    public JsonResult<List<Product>> getHotList(){
        List<Product> data = productService.findHotList();
        return new JsonResult<>(OK,data);
    }
}
```

---

#### 4.前端页面

```javascript
            function showHotList() {
                $("#hot-list").empty();
                $.ajax({
                    url:"/product/hot_list",
                    type:"GET",
                    dataType: "json",
                    success: function (res) {
                        for (let i = 0; i < res.data.length; i++) {
                            let str = "";
                            let product = res.data[i]
                            let image = ".." + product.image + "collect.png"
                            str = "<div class='col-md-12'>"
                                    + "<div class=\"col-md-7 text-row-2\">"
                                    + "<a href='javascript:void(0);'onclick='jumpWithId(#{id})'>" + product.title + "</a>"
                                    + "</div>"
                                    + "<div class=\"col-md-2\">￥" + product.price + "</div>"
                                    + "<div class=\"col-md-3\">"
                                    + "<img src=" + image  + " class='img-responsive' />"
                                    + "</div>"
                                    + "</div>"
                            str = str.replaceAll("#{id}",product.id)
                            $("#hot-list").append(str)
                        }
                    },
                    error: function () {
                        alert("查询错误，请等待攻城狮修复！！")
                    }
                });
            }
```

---

## 5.4 新到商品

### 5.4.1 新到商品-持久层

---

#### 1.编写sql语句

```sql
//按照上架物品的修改时间查询新商品
SELET * FROM t_product WHERE status = 1 
ORDER BY modified_time DESC LIMIT 0,4
```

---

#### 2.定义抽象方法

```java
    /**
     * 按照创建时间查找新到的商品
     * @return 返回新商品列表
     */
    List<Product> findNewProductList();
```

---

#### 3.定义mapper接口映射文件

```xml
<!--    List<Product> findNewProductList();-->
    <select id="findNewProductList" resultType="com.zxl.store.entity.Product">
        SELECT * FROM t_product WHERE status = 1 ORDER BY modified_time DESC LIMIT 0,4
    </select>
```

---

#### 4.单元测试

----

### 5.4.2 新到商品-业务层

---

#### 1.定义异常

无

---

#### 2.定义业务层抽象方法

```java
    /**
     * 按照商品状态和创建时间选取新商品集合
     * @return
     */
    List<Product> findNewProductList();
```

---

#### 3. 编写业务层逻辑

```java
    @Override
    public List<Product> findNewProductList() {
        //查找数据
        List<Product> res = productMapper.findNewProductList();
        return res;
    }
```

---

#### 4. 单元测试

---

### 5.4.3 新到商品-控制层

#### 1.处理异常

无

---

#### 2.定义请求

请求路径：/product/new_list

请求参数：无

请求类型：get

响应类型：JsonResult<List< Product>>

---

#### 3.处理请求，编写控制方法

```java
    /**
     *  返回新商品
     * @return
     */
    @RequestMapping(value = "/new_list",method = RequestMethod.GET)
    public JsonResult<List<Product>> getNewList(){
        List<Product> data = productService.findNewProductList();
        return new JsonResult<>(OK,data);
    }
```

---

#### 4.前端页面

```javascript
            function showNewList() {
                $("#new-list").empty();
                $.ajax({
                    url:"/product/new_list",
                    type:"GET",
                    dataType: "json",
                    success: function (res) {
                        for (let i = 0; i < res.data.length; i++) {
                            let str = "";
                            let product = res.data[i]
                            let image = ".." + product.image + "collect.png"
                            str = "<div class='col-md-12'>"
                                    + "<div class=\"col-md-7 text-row-2\">"
                                    + "<a href='javascript:void(0);'onclick='jumpWithId(#{id})'>" + product.title + "</a>"
                                    + "</div>"
                                    + "<div class=\"col-md-2\">￥" + product.price + "</div>"
                                    + "<div class=\"col-md-3\">"
                                    + "<img src=" + image  + " class='img-responsive' />"
                                    + "</div>"
                                    + "</div>"
                            str = str.replaceAll("#{id}",product.id)
                            $("#new-list").append(str)
                        }
                    },
                    error: function () {
                        alert("查询错误，请等待攻城狮修复！！")
                    }
                });
            }
```

---

## 5.5 显示商品

### 5.5.1 显示商品-持久层

---

#### 1.编写sql

```sql
//查找按照商品id查找商品
 SELECT id,title,price,image FROM t_product
 WHERE id = #{id}
```

---

#### 2.编写Mapper接口的抽象方法

```java
    /**
     * 按照商品的id查找商品
     * @param pid
     * @return
     */
    Product findProductById(Integer id);
```

---

#### 3.编写Mapper接口的映射文件

```xml
<!--    Product findProductById(Integer id);-->
    <select id="findProductById" resultType="com.zxl.store.entity.Product">
        SELECT * FROM t_product WHERE status = 1 AND id = #{id}
    </select>
```

---

#### 4.单元测试

---

### 5.5.2 显示商品-业务层

#### 1.异常处理

商品为找到异常

商品状态异常(为未上架或者已经删除的商品)

```java
/**
 * @author zxl
 * @description 商品未找到异常
 * @date 2022/11/4
 */
public class ProductNotFoundException extends ServiceException {}

/**
 * @author zxl
 * @description 商品状态异常
 * @date 2022/11/4
 */
public class ProductStatusException extends ServiceException {}
```

---

#### 2.编写业务层接口的抽象方法

```java
    /**
     * 根据商品id查找商品
     * @param id 商品id
     * @return 返回商品信息
     */
    Product findProductById(Integer id);
```

---

#### 3.编写业务层具体逻辑

```java
    @Override
    public Product findProductById(Integer id) {
        //根据id查询商品信息
        Product res = productMapper.findProductById(id);
        //判断商品存在或者商品状态是否为上架
        if(res==null){
            throw new ProductNotFoundException("查询商品不存在");
        }
        if(res.getStatus()!=1){
            throw new ProductStatusException("查询商品尚未上架");
        }
        //传输商品信息
        return res;
    }
```

---

#### 4.单元测试

---

### 5.5.3 显示商品-控制层

#### 1.异常处理

将上述两个异常加入BaseConrtroller

---

#### 2.设计请求

请求路径：/product/{id}

请求参数：Integet id

请求类型：get

响应类型：JsonResult< Product>

---

#### 3.处理请求，设计控制层方法

```java
    @RequestMapping(value = "/{id}")
    public  JsonResult<Product> findProductById(@PathVariable("id")Integer id){
        //按照获得的id查询商品
        Product data = productService.findProductById(id);
        return new JsonResult<>(OK,data);
    }
```

#### 4.前端页面

```javascript
//index.html
            function jumpWithId(id) {
                let jumpUrl = "product.html?id="+id;
                location.href=jumpUrl;
            }
//function 
//返回一个参数
function getOne(){
    var result;
    //返回字符串从url的?处开始
    var url = decodeURI(window.location.search);
    //如果等于-1，代表没有找到，即网页连接没有携带任何参数
    if (url.indexOf("?") != -1){
        //返回一个新的字符串，从url连接=符号处索引+1的位置开始返回
        result = url.substr(url.indexOf("=")+1);
    }
    return result;
}
//product.html
            //接收上一个页面传来的连接
            var pid = getOne();

            function showInThisProductHtml(){
                //判断是否携带参数
                if (location.search.substring(1).indexOf("id") === -1){
                    location.href = "500.html"
                    return false;
                }

                //在页面加载完成时自动发送此ajax请求并填充表单
                $.ajax({
                    url: "/product/" + pid,
                    type: "get",
                    dataType: "json",
                    success:function (res) {
                        let product = res.data;
                        //将普通的数据填充至页面
                        $("#product-title").text(product.title)
                        $("#product-sell-point").append(product.sellPoint)
                        $("#product-price").text(product.price)
                        $("#stock").text(product.num)
                        //将数据库查询的图片进行替换
                        let image = ".." +  product.image
                        //遍历填充图片数据
                        for (let i = 1; i <= 5; i++) {
                            $("#product-image-" + i + "-big").attr("src",image + i + "_big.png")
                            $("#product-image-" + i).attr("src",image + i + ".jpg")
                        }
                    }
                })
            }
```

---

