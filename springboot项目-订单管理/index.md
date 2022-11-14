# SpringBoot项目-订单管理


# 订单管理

## 7.1 创建数据表

```sql
CREATE TABLE t_order (
    oid INT AUTO_INCREMENT COMMENT '订单id',
    uid INT NOT NULL COMMENT '用户id',
    aid INT NOT NULL COMMENT '收货地址id',
    recv_name VARCHAR(20) NOT NULL COMMENT '收货人姓名',
    recv_phone VARCHAR(20) COMMENT '收货人电话',
    recv_province VARCHAR(15) COMMENT '收货人所在省',
    recv_city VARCHAR(15) COMMENT '收货人所在市',
    recv_area VARCHAR(15) COMMENT '收货人所在区',
    recv_address VARCHAR(50) COMMENT '收货详细地址',
    total_price BIGINT COMMENT '总价',
    status INT COMMENT '状态：0-未支付，1-已支付，2-已取消，3-已关闭，4-已完成',
    order_time DATETIME COMMENT '下单时间',
    pay_time DATETIME COMMENT '支付时间',
    created_user VARCHAR(20) COMMENT '创建人',
    created_time DATETIME COMMENT '创建时间',
    modified_user VARCHAR(20) COMMENT '修改人',
    modified_time DATETIME COMMENT '修改时间',
    PRIMARY KEY (oid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE t_order_item (
    id INT AUTO_INCREMENT COMMENT '订单中的商品记录的id',
    oid INT NOT NULL COMMENT '所归属的订单的id',
    pid INT NOT NULL COMMENT '商品的id',
    title VARCHAR(100) NOT NULL COMMENT '商品标题',
    image VARCHAR(500) COMMENT '商品图片',
    price BIGINT COMMENT '商品价格',
    num INT COMMENT '购买数量',
    created_user VARCHAR(20) COMMENT '创建人',
    created_time DATETIME COMMENT '创建时间',
    modified_user VARCHAR(20) COMMENT '修改人',
    modified_time DATETIME COMMENT '修改时间',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

---

## 7.2 创建实体类

```java
/**
 * @author zxl
 * @description t_order表对应的实体类
 * @date 2022/11/6
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order extends BaseEntity{
    private Integer oid;
    private Integer uid;
    private Integer aid;
    private String recvName;
    private String recvPhone;
    private String recvProvince;
    private String recvCity;
    private String recvArea;
    private String recvAddress;
    private Long totalPrice;
    private Integer status;
    private Date orderTime;
    private Date payTime;
}
/**
 * @author zxl
 * @description t_order_item表对应的实体类
 * @date 2022/11/6
 */
public class OrderItem extends BaseEntity {
    private Integer id;
    private Integer oid;
    private Integer pid;
    private String title;
    private String image;
    private Long price;
    private Integer num;
}
```

---

## 7.3 创建订单

---

### 7.3.1 后端-持久层

#### 1.编写sql

```sql
//在t_order表中插入数据
INSERT INTO t_order(oid除外)VALUES(字段的值)

//在t_order_item中插入数据
INSERT INTO t_order_item (id除外)VALUES(字段的值)
```

---

#### 2.编写Mapper接口的抽象方法

```java
/**
 * @author zxl
 * @description 订单类的Mapper接口
 * @date 2022/11/6
 */
public interface OrderMapper {
    /**
     * 插入order
     * @param order 订单数据
     * @return
     */
    Integer insertOrder(Order order);

    /**
     * 插入order_item数据
     * @param orderItem
     * @return
     */
    Integer insertOrderItem(OrderItem orderItem);
}
```

---

#### 3.编写Mapper接口的映射文件

```xml
<mapper namespace="com.zxl.store.mappers.OrderMapper">
    <!--    Integer insertOrder(Order order);-->
    <insert id="insertOrder" useGeneratedKeys="true" keyProperty="oid">
        insert into t_order(uid,aid,recv_name,recv_phone,recv_province,
                            recv_city,recv_area,recv_address,
                            total_price,status,order_time,pay_time,
                            created_user,created_time,modified_user,modified_time)
                        values(
                            #{uid},#{aid},#{recvName},#{recvPhone},#{recvProvince},#{recvCity},
                            #{recvArea},#{recvAddress},#{totalPrice},#{status},#{orderTime},
                            #{payTime},#{createdUser},#{createdTime},#{modifiedUser},#{modifiedTime}
                               )
    </insert>

    <!--    Integer insertOrderItem(OrderItem orderItem);-->
    <insert id="insertOrderItem">
        insert into t_order_item(oid,pid,title,image,price,num,
                        created_user,created_time,modified_user,modified_time)
                        values(#{oid},#{pid},#{title},#{image},#{price},
                               #{num},#{createdUser},#{createdTime},#{modifiedUser},#{modifiedTime}
                          )
    </insert>
</mapper>
```

---

#### 单元测试

---

### 7.3.2 后端业务层

---

#### 1.编写异常

插入失败，已经被定义

---

#### 2.编写业务层抽象方法

```java
    /**
     * 插入订单数据
     * @param aid 收货地址id
     * @param uid 用户id
     * @param totalPrice 商品总价
     * @param username 操作人
     */
    Order insertOrder(Integer aid,Integer uid,Long totalPrice,String username);


    /**
     * 添加orderItem
     * @param oid
     * @param cid
     * @param num
     * @param username
     */
    void insertOrderItem(Integer oid,Integer cid,Integer num,String username);
```

---

#### 3.实现逻辑

```java
/**
 * @author zxl
 * @description  订单的业务层接口实现类
 * @date 2022/11/6
 */
@Service
public class IOrderServiceImpl implements IOrderService {
    @Autowired(required = false)
    private OrderMapper orderMapper;
    @Autowired
    private IAddressService addressService;
    @Autowired
    private ICartService cartService;
    @Autowired
    private IProductService productService


    @Override
    public Order insertOrder(Integer aid, Integer uid, Long totalPrice, String username) {
        //根据控制层传入的aid进行查询
        Address address = addressService.findAddressByAid(aid);

        //创建一个用于向持久层传输的Order实体类对象
        Order order = new Order();

        //补全order对象的空白字段
        order.setUid(uid);
        order.setAid(aid);
        order.setRecvName(address.getName());
        order.setRecvPhone(address.getPhone());
        order.setRecvProvince(address.getProvinceName());
        order.setRecvCity(address.getCityName());
        order.setRecvArea(address.getAreaName());
        order.setRecvAddress(address.getAddress());
        order.setTotalPrice(totalPrice);
        order.setStatus(0); //表示未支付
        Date createdTime = new Date();
        order.setOrderTime(createdTime);
        order.setPayTime(null);
        order.setCreatedUser(username);
        order.setModifiedUser(username);
        order.setCreatedTime(createdTime);
        order.setModifiedTime(createdTime);

        //调用持久层进行插入
        int result = orderMapper.insertOrder(order);

        if (result == 0){
            throw new InsertException("服务器出现错误，创建订单失败");
        }
        //根据oid查询指定的订单，并返回给控制层
        return  orderMapper.findOrderByOid(order.getOid());
    }

    @Override
    public void insertOrderItem(Integer oid, Integer cid, Integer num, String username) {
        //根据cid查询订单获取pid
        Cart cart = cartService.findCartByCid(cid);

        //取出pid的值
        Integer pid = cart.getPid();

        //根据pid查询商品信息
        Product product = productService.findProductById(pid);

        //创建一个用于向持久层传输的OrderItem实体类对象
        OrderItem orderItem = new OrderItem();

        //补全orderItem对象的空白字段
        orderItem.setOid(oid);
        orderItem.setPid(pid);
        orderItem.setTitle(product.getTitle());
        orderItem.setImage(product.getImage());
        orderItem.setPrice(product.getPrice());
        orderItem.setNum(num);
        Date createdTime = new Date();
        orderItem.setCreatedUser(username);
        orderItem.setCreatedTime(createdTime);
        orderItem.setModifiedUser(username);
        orderItem.setModifiedTime(createdTime);

        //调用持久层进行插入
        int result = orderMapper.insertOrderItem(orderItem);
        if (result == 0){
            throw new InsertException("服务器出现错误，创建订单失败");
        }
    }
}
```

----

#### 4.单元测试

---

### 7.3.3 后端控制层

#### 1.处理异常

---

#### 2.设计请求

请求路径： /order/createOrder 

请求参数：Integer aid,Long totalPrice,HttpSession session

​ ②Integer oid,Integer cid,Integer pid,Integer num,HttpSession session

请求类型：post

响应类型：JsonResult< Order> 

---

#### 3.设计控制类

```java
/**
 * @author zxl
 * @description
 * @date 2022/11/6
 */
@RestController
@RequestMapping("/Order")
public class OrderController extends BaseController {
    @Autowired
    private IOrderService orderService;

    /**
     * Description : 处理用户创建order订单的请求
     * @date 2022/7/18
     * @param aid 用户选中的地址aid
     * @param totalPrice 商品的总金额
     * @param session 项目启动自动生成的session对象
     * @return top.year21.computerstore.utils.JsonResult<java.lang.Void>
     **/
    @PostMapping("/createOrder")
    public JsonResult<Order> createOrder(Integer aid, Long totalPrice, HttpSession session){
        //从session中取出用户名和uid
        Integer uid = getUserIdFromSession(session);
        String username = getUsernameFromSession(session);

        //调用业务层方法执行插入操作
        Order order = orderService.insertOrder(aid,uid, totalPrice,username);

        return new JsonResult<>(OK,order);
    }

}
```

---

#### 4.前端页面

```javascript

```

---


