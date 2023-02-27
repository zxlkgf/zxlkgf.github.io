# SpringBoot项目-地址管理

## 4.地址管理

收货地址的功能模块:列表展示，修改，删除，设置默认，新增收货地址

开发的顺序:新增收货地址->列表展示->设置默认收货地址->修改删除

### 4.1 创建数据表

创建数据表

```sql
CREATE TABLE t_address (
    aid INT AUTO_INCREMENT COMMENT '收货地址id',
    uid INT COMMENT '归属的用户id',
    NAME VARCHAR(20) COMMENT '收货人姓名',
    province_name VARCHAR(15) COMMENT '省-名称',
    province_code CHAR(6) COMMENT '省-行政代号',
    city_name VARCHAR(15) COMMENT '市-名称',
    city_code CHAR(6) COMMENT '市-行政代号',
    area_name VARCHAR(15) COMMENT '区-名称',
    area_code CHAR(6) COMMENT '区-行政代号',
    zip CHAR(6) COMMENT '邮政编码',
    address VARCHAR(50) COMMENT '详细地址',
    phone VARCHAR(20) COMMENT '手机',
    tel VARCHAR(20) COMMENT '固话',
    tag VARCHAR(6) COMMENT '标签',
    is_default INT COMMENT '是否默认：0-不默认，1-默认',
    created_user VARCHAR(20) COMMENT '创建人',
    created_time DATETIME COMMENT '创建时间',
    modified_user VARCHAR(20) COMMENT '修改人',
    modified_time DATETIME COMMENT '修改时间',
    PRIMARY KEY (aid)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

___

### 4.2 创建实体类

地址类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Address extends BaseEntity{
    /*
     * aid INT AUTO_INCREMENT COMMENT '收货地址id',
    uid INT COMMENT '归属的用户id',
    name VARCHAR(20) COMMENT '收货人姓名',
    province_name VARCHAR(15) COMMENT '省-名称',
    province_code CHAR(6) COMMENT '省-行政代号',
    city_name VARCHAR(15) COMMENT '市-名称',
    city_code CHAR(6) COMMENT '市-行政代号',
    area_name VARCHAR(15) COMMENT '区-名称',
    area_code CHAR(6) COMMENT '区-行政代号',
    zip CHAR(6) COMMENT '邮政编码',
    address VARCHAR(50) COMMENT '详细地址',
    phone VARCHAR(20) COMMENT '手机',
    tel VARCHAR(20) COMMENT '固话',
    tag VARCHAR(6) COMMENT '标签',
    is_default INT COMMENT '是否默认：0-不默认，1-默认',
    created_user VARCHAR(20) COMMENT '创建人',
    created_time DATETIME COMMENT '创建时间',
    modified_user VARCHAR(20) COMMENT '修改人',
    modified_time DATETIME COMMENT '修改时间',
     */
    private Integer aid;
    private Integer uid;
    private String name;
    private String provinceName;
    private String provinceCode;
    private String cityName;
    private String cityCode;
    private String areaName;
    private String areaCode;
    private String zip;
    private String address;
    private String phone;
    private String tel;
    private String tag;
    private Integer isDefault;
}
```

___

### 4.3 新增地址

#### 4.3.1 后端-持久层

##### 1.编写sql语句

```sql
//插入语句
insert into t_address(除了aid外字段)value(字段值)
//查看收货地址数量(一个用户最大只能20条)
select count(*) from t_address where uid = ?
```

____

##### 2.定义Mapper接口和抽象方法

```java
/**
 * @author zxl
 * @version 1.0.0
 * @date 2022/11/02
 * @desciption Address所对应的Mapper接口
 */
public interface AddressMapper {
    /**
     * 插入用户的收货地址数据
     * @param address 收货地址
     * @return {@link Integer} 受影响的行数
     */
    Integer addAddress(Address address);

    /**
     * 根据用户id统计用户收货地址数量
     * @param uid 用户uid
     * @return {@link Integer} 返回当前用户的收货地址总数
     */
    Integer userAddressCount(Integer uid);
}
```

___

##### 3.实现Mapper接口的映射

```xml
<mapper namespace="com.zxl.store.mappers.AddressMapper">
    <!--addAddress 插入用户的地址-->
    <insert id="addAddress" useGeneratedKeys="true" keyProperty="aid">
        INSERT INTO t_address (
            uid,name, province_name, province_code, city_name, city_code, area_name, area_code, zip,
            address, phone, tel, tag, is_default, created_user, created_time, modified_user, modified_time
        ) VALUES (
            #{uid}, #{name}, #{provinceName}, #{provinceCode}, #{cityName}, #{cityCode}, #{areaName},
            #{areaCode}, #{zip}, #{address}, #{phone}, #{tel}, #{tag}, #{isDefault}, #{createdUser},
            #{createdTime}, #{modifiedUser}, #{modifiedTime}
        )
    </insert>
    <!--userAddressCount 根据用户uid查询用户地址数量-->
    <select id="userAddressCount">
        SELECT COUNT(*) FROM t_address WHERE uid = #{uid}
    </select>
</mapper>
```

----

##### 4.单元测试Mapper接口

----

#### 4.3.2 后端-业务层

如果用户插入的地址是第一条时，将其设为默认的收货地址

##### 1.  规划异常

用户地址超出数量异常

```java
/**
 * @author zxl
 * @description 用户地址数目超出异常
 * @date 2022/11/2
 */
public class AddressCountLimitException extends ServiceException {}
```

____

##### 2.接口和抽象方法

```java
/**
 * @author zxl
 * @description 收货地址业务层接口
 * @date 2022/11/2
 */
public interface IAddressService {

    /**
     * 添加用户地址
     * @param uid      用户uid
     * @param username 用户名
     * @param address  地址
     */
    void addAddress(Integer uid, String username, Address address);

}
```

____

##### 3.实现抽象方法

```java
/**
 * @author zxl
 * @description 收货地址业务层的实现类
 * @date 2022/11/2
 */
@Service
public class IAddressServiceImpl implements IAddressService {
    @Autowired
    private AddressMapper addressMapper;

    @Value("${user.address.max-count}")
    private Integer maxCount;

    @Override
    public void addAddress(Integer uid, String username, Address address) {
        //先判断用户地址的条数
        Integer count = addressMapper.userAddressCount(uid);
        //判断是否超过20条记录
        if(count>=maxCount){
            throw new AddressCountLimitException("用户地址已经到达上限，请删除部分地址");
        }
        //判断当前地址是否为0
        if(count==0){
            address.setIsDefault(1);
        }
        //补全四项日志
        Date currentTime = new Date();
        address.setCreatedUser(username);
        address.setCreatedTime(currentTime);
        address.setModifiedUser(username);
        address.setModifiedTime(currentTime);
        //插入
        Integer row = addressMapper.addAddress(address);
        //判断插入结果
        if(row!=1){
            throw new InsertException("插入时产生未知异常");
        }
    }
}
```

____

##### 4.单元测试

----

#### 4.3.3 后端控制层

##### 1.处理异常

将异常添加到全局异常处理

____

##### 2.设计请求

请求路径：/address

请求参数：Address address,HttpSession session

请求类型：post

响应类型：JsonResult< Void>

____

##### 3.处理请求，创建相应controller

```java
/**
 * @author zxl
 * @description 地址操作的控制层
 * @date 2022/11/2
 */
@RestController
@RequestMapping("/address")
public class AddressController extends BaseController{
    @Autowired
    private IAddressService addressService;

    /**
     * 处理用户添加地址的操作
     * @param address 添加的地址
     * @param session 项目自动生成的Session
     * @return {@link JsonResult}<{@link Void}>
     */
    @PostMapping
    public JsonResult<Void> addAddress(Address address, HttpSession session){
        //从Session中获取Uid和Username
        Integer uid = getUserIdFromSession(session);
        String username = getUsernameFromSession(session);
        //添加
        addressService.addAddress(uid,username,address);
        return new JsonResult<>(OK);
    }
}
```

____

##### 4.前端页面

```javascript
        <script type="text/javascript">
            $(document).ready(function () {

                $("#btn-add-new-address").click(function() {
                    //判断手机号和收货人是否为空
                    let name = $("#name").val();
                    let phone = $("#phone").val();
                    let zip = $("#zip").val();
                    let tag = $("#tag").val();
                    if (phone == "" || name == ""){
                        $("#error-msg").html("请先填写需要添加的信息！");
                        return false;
                    }
                    //验证手机号是否符合要求
                    let checkPhone = /(^1\d{10}$)|(^[0-9]\d{7}$)/;
                    if (!checkPhone.test(phone)){
                        $("#error-msg").html("手机号不符合要求！");
                        return false;
                    }
                    //验证邮箱是否为空或者超出最大长度6
                    if(zip.length>=6){
                        $("#error-msg").html("邮箱的最大长度6");
                        return false;
                    }
                    //验证地址类型不可以超过6
                    if(tag.length>=6){
                        $("#error-msg").html("地址类型的最大长度6");
                        return false;
                    }
                    $.ajax({
                        url: "/address",
                        type: "POST",
                        data: $("#form-add-new-address").serialize(),
                        dataType: "JSON",
                        success: function(json) {
                            if (json.state == 200) {
                                alert("新增收货地址成功！");
                                location.href="address.html"
                            } else {
                                alert("新增收货地址失败！" + json.message);
                            }
                        },
                        error: function(xhr) {
                            alert("您的登录信息已经过期，请重新登录！HTTP响应码：" + xhr.status);
                            location.href = "login.html";
                        }
                    });
                });
            });
        </script>
```

____

## 4.4 获取省市区列表

### 4.4.1 创建数据表

```sql
//parent属性代表的是父区域的代码号，省的父代码号是+86
//code代表的是本身的代码号
//name就是code所代表的本身的名称
CREATE TABLE t_dict_district (
    id int(11) NOT NULL AUTO_INCREMENT,
    parent varchar(6) DEFAULT NULL,
    code varchar(6) DEFAULT NULL,
    name varchar(16) DEFAULT NULL,
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
//部分省市区数据太大
//请参考代码
```

___

### 4.4.2 创建实体类

创建District实体类

```java
**
 * @author zxl
 * @description 省市区的实体类
 * @date 2022/11/3
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class District extends BaseEntity {
    private Integer id;
    private String parent;
    private String code;
    private String name;
}
```

____

### 4.4.3 后端-持久层

#### 1.编写sql语句

```sql
//1.查询语句，根据父代号查询 (ASC 升序)
SELECT * FROM t_dict_district WHERE parent = ? ORDER BY code ASC
//2.根据code查询对应省市区名字
SELECT name FROM t_dict_district WHERE code = ?
```

____

#### 2.定义Mapper接口和抽象方法

```java
public interface DistrictMapper {
    /**
     * 根据用户的父代号查询区域信息
     * @param parent 父代号
     * @return 某个父区域下的所有区域列表
     */
    List<District> findDistrictByParent(String parent);

    /**
     * 根据code查询区域名称
     * @param code 区域代号
     * @return 返回区域名称
     */
    String findNameByCode(String code);
}
```

____

#### 3.编写映射文件

```xml
<mapper namespace="com.zxl.store.mappers.DistrictMapper">
<!--    List<District> findDistrictByParent(String parent);-->
    <select id="findDistrictByParent" resultType="com.zxl.store.entity.District">
        SELECT * FROM t_dict_district WHERE parent = #{parent} ORDER BY code ASC
    </select>
<!--    String findNameByCode(String code);-->
    <select id="findNameByParent" resultType="java.lang.String">
        SELECT name FROM t_dict_district WHERE code = #{code}
    </select>
</mapper>
```

---

#### 4.单元测试

---

### 4.4.4 后端-业务层

#### 1.规划异常

暂无异常可以规划

---

#### 2.定义Service接口和抽象方法

```java
/**
 * @author zxl
 * @description District业务层的接口类
 * @date 2022/11/3
 */
public interface IDistrictService {
    /**
     * 根据父代号查询区域信息(省市区)
     * @param parent 父代号
     * @return 返回多个查询结果
     */
    List<District> getDistrictByParent(String parent);

    /**
     * 按照code查询当前省市区名称
     * @param code code
     * @return 返回省市区名称
     */
    String getNameByCode(String code);
}
```

---

#### 3.编写具体的实现方法和处理逻辑

```java
/**
 * @author zxl
 * @description 处理省市区业务层接口的实现类
 * @date 2022/11/3
 */
@Service
public class IDistrictServiceImpl implements IDistrictService {
    @Autowired(required = false)
    private DistrictMapper districtMapper;

    //根据父代号查询省市区的信息
    @Override
    public List<District> getDistrictByParent(String parent) {
        List<District> districtByParent = districtMapper.findDistrictByParent(parent);
        //无效数据设为null
        for (District district : districtByParent) {
            district.setId(null);
            district.setParent(null);
        }
        return districtByParent;
    }

    //根据code查询省市区的名称
    @Override
    public String getNameByCode(String code) {
        return districtMapper.findNameByCode(code);
    }
}
```

---

#### 4.单元测试

---

### 4.4.5 后端-控制层

#### 1.异常处理

没有异常，不需要处理

---

#### 2.设计请求

请求路径：/district/parent

请求参数：String parent

请求类型：get

响应类型：JsonResult<List< District>>

---

#### 3.处理请求，创建一个控制类

```java
/**
 * @author zxl
 * @description 处理省市区相关业务的控制层
 * @date 2022/11/3
 */
@RestController
@RequestMapping("/district")
public class DistrictController extends BaseController{
    @Autowired(required = false)
    private IDistrictService districtService;

    @RequestMapping(value = "/parent",method = RequestMethod.GET)
    public JsonResult<List<District>> getDistrictByParent(String parent){
        //查询数据
        List<District> Data = districtService.getDistrictByParent(parent);
        //返回数据
        return new JsonResult<>(OK,Data);
    }
}
```

---

#### 4. 前端页面

```javascript
        <script type="text/javascript">
            $(document).ready(function () {
                //页面加载完成时，先把三个省市区的提示设置好
                let provinceFirst = '<option value="0">--- 请选择省 ---</option>';
                let cityFirst = '<option value="0">--- 请选择市 ---</option>';
                let areaFirst = '<option value="0">--- 请选择区 ---</option>';
                //插入提示框
                $("#province-list").empty();
                $("#city-list").empty();
                $("#area-list").empty();

                $("#province-list").append(provinceFirst);
                $("#city-list").append(cityFirst);
                $("#area-list").append(areaFirst);

                //记录省的点击查询次数
                let provinceClick = 0;
                $("#province-list").click(function () {
                    provinceClick++;
                    console.log(provinceClick);
                    //如果provinceClick为1代表首次点击
                    if(provinceClick==1){
                        let str = "";
                        $.ajax({
                            url:"/district/parent",
                            type:"get",
                            data:"parent=86",
                            dataType:"json",
                            success:function (json) {
                                //如果返回消息成功，将数据填充回列表
                                if(json.state==200){
                                    for(i = 0;i<json.data.length;i++){
                                        //每个district对象
                                        let district = json.data[i];
                                        str = '<option value="' + district.code + '">' + district.name + '</option>';
                                        $("#province-list").append(str);
                                    }
                                }
                            },
                            error:function () {
                                alert("查询省市区列表错误，请联系管理员修复！");
                            }
                        });
                    }
                });
                //监听省份的选择  为城市的选择做出变化
                $("#province-list").change(function () {
                    //清空select下的所有option元素
                    $("#city-list").empty();
                    $("#area-list").empty();
                    //追加默认值
                    $("#city-list").append(cityFirst);
                    $("#area-list").append(areaFirst);

                    //获取省份选择的是什么
                    let provinceChoice = $("#province-list").val();
                    //等于0 则不做请求
                    if(provinceChoice=="0")return false;

                    let str = "";
                    $.ajax({
                        url:"/district/parent",
                        type:"get",
                        data:"parent="+provinceChoice,
                        dataType:"json",
                        success:function (json) {
                            if(json.state==200){
                                for( i = 0;i<json.data.length;i++){
                                    //每个district对象
                                    let district = json.data[i];
                                    str = '<option value="' + district.code + '">' + district.name + '</option>';
                                    $("#city-list").append(str);
                                }
                            }
                        },
                        error:function () {
                            alert("查询省市区列表错误，请联系管理员修复！");
                        }
                    });
                });

                //监听城市选择 为区县的选择做出变化
                $("#city-list").change(function () {
                    //获取当前选择的城市是什么
                    let cityChoice = $("#city-list").val();
                    //清空select下的option元素
                    $("#area-list").empty();
                    //重新设置默认值
                    $("#area-list").append(areaFirst);

                    //判断默认值是什么
                    //如果是0则不发送ajax请求
                    if(cityChoice=="0")return false;

                    //发送请求
                    $.ajax({
                        url:"/district/parent",
                        type:"get",
                        data:"parent="+cityChoice,
                        dataType:"json",
                        success:function (json) {
                            if(json.state==200){
                                for( i = 0;i<json.data.length;i++){
                                    //每个district对象
                                    let district = json.data[i];
                                    str = '<option value="' + district.code + '">' + district.name + '</option>';
                                    $("#area-list").append(str);
                                }
                            }
                        },
                        error:function () {
                            alert("查询省市区列表错误，请联系管理员修复！");
                        }
                    });
                });

                //添加地址
                $("#btn-add-new-address").click(function() {
                    //判断手机号和收货人是否为空
                    let name = $("#name").val();
                    let phone = $("#phone").val();
                    let zip = $("#zip").val();
                    let tag = $("#tag").val();

                    //由于不知道的原因导致省市区的名称无法提交，所以进行这一步
                    let provinceName = $("#province-list").find("option:selected").text();
                    $("#provinceName").val(provinceName);
                    let cityName = $("#city-list").find("option:selected").text();
                    $("#cityName").val(cityName);
                    let areaName = $("#area-list").find("option:selected").text();
                    $("#areaName").val(areaName);

                    if (phone == "" || name == ""){
                        $("#error-msg").html("请先填写需要添加的信息！");
                        return false;
                    }
                    //验证手机号是否符合要求
                    let checkPhone = /(^1\d{10}$)|(^[0-9]\d{7}$)/;
                    if (!checkPhone.test(phone)){
                        $("#error-msg").html("手机号不符合要求！");
                        return false;
                    }
                    //验证邮箱是否为空或者超出最大长度6
                    if(zip.length>=6){
                        $("#error-msg").html("邮箱的最大长度6");
                        return false;
                    }
                    //验证地址类型不可以超过6
                    if(tag.length>=6){
                        $("#error-msg").html("地址类型的最大长度6");
                        return false;
                    }
                    $.ajax({
                        url: "/address",
                        type: "POST",
                        data: $("#form-add-new-address").serialize(),
                        dataType: "JSON",
                        success: function(json) {
                            if (json.state == 200) {
                                alert("新增收货地址成功！");
                                location.href="address.html"
                            } else {
                                alert("新增收货地址失败！" + json.message);
                            }
                        },
                        error: function(xhr) {
                            alert("您的登录信息已经过期，请重新登录！HTTP响应码：" + xhr.status);
                            location.href = "login.html";
                        }
                    });
                });
            });
        </script>
```

---

## 4.5 获取用户地址

### 4.5.1 后端-持久层

---

#### 1.编写sql语句

```sql
//获取用户收货地址
SELECT * FROM t_address WHERE uid = ? ORDER BY is_default DESC , 
created_time DESC
```

---

#### 2.定义抽象方法

只需要在AddressMapper接口中定义新的抽象方法即可

```java
    /**
     * 根据用户的uid查询用户的收货地址集合
     * @param uid 用户uid 
     * @return 返回收货地址数据
     */
    List<Address> findByUid(Integer uid);
```

---

#### 3.编写映射文件

```xml
<!--    List<Address> findByUid(Integer uid);-->
    <select id="findByUid" resultType="com.zxl.store.entity.Address">
        SELECT * FROM t_address WHERE uid = #{uid} ORDER BY is_default DESC ,created_time DESC
    </select>
```

---

#### 4.创建Mapper的单元测试

---

### 4.5.2 后端-业务层

#### 1.异常控制

---

#### 2.定义业务层抽象方法

```java
    /**
     * 根据用户的uid获取用户的收货地址信息集
     * @param uid   用户的uid
     * @return 返回用户的地址集合
     */
    List<Address> getAddressByUid(Integer uid);
```

---

#### 3.编写具体的业务处理逻辑

```java
   /**
     * 按照用户uid查询用户的收货地址集合
     * @param uid   用户的uid
     * @return 返回用户的收货地址的集合
     */
    @Override
    public List<Address> getAddressByUid(Integer uid) {
        //获取结果
        List<Address> res = addressMapper.findByUid(uid);
        //由于网页端只需要地址类型，收件人姓名，详细地址，联系电话，所以对信息进行部分过滤
        for (Address address : res) {
            address.setUid(null);
            address.setProvinceCode(null);
            address.setCityCode(null);
            address.setAreaCode(null);
            address.setTel(null);
            address.setCreatedTime(null);
            address.setCreatedUser(null);
            address.setModifiedTime(null);
            address.setModifiedUser(null);
            address.setIsDefault(null);
        }
        return res;
    }
```

---

#### 4.编写业务层单元测试

---

### 4.5.3 后端-控制层

#### 1.异常处理

---

#### 2.设计请求

请求地址:/address

请求参数:HttpSession session

请求类型:GET

相应类型:JsonResult<List<Address>>

---

#### 3.处理请求，编写控制层

```java
    /**
     * 处理网页端自动显示用户收货地址的请求
     * @param session 项目自动生成的session
     * @return 返回JsonResult<List<Address>>
     */
    @GetMapping
    public JsonResult<List<Address>> queryAllAddress(HttpSession session){
        List<Address> res = addressService.getAddressByUid(getUserIdFromSession(session));
        return new JsonResult<>(OK,res);
    }
```

---

#### 4.前端页面

```javascript
            //页面初始化加载用户收货地址信息
            $(document).ready(function () {
                $("#address-list").empty();
                $.ajax({
                    url:"/address",
                    type:"GET",
                    dataType:"json",
                    success:function (json) {
                        if(json.data.length!=0){
                            let list = json.data;
                            for (let i = 0; i < list.length; i++) {
                                let address = list[i];
                                let str = " ";
                                str = "<tr>"
                                        +"<td>"+address.tag+"</td>"
                                        +"<td>"+address.name+"</td>"
                                        +"<td>"
                                        +address.provinceName + address.cityName + address.areaName + address.address
                                        +"</td>"
                                        +"<td>"+address.phone+"</td>"
                                        + "<a href='javascript:void(0);' onclick='updateAddress(#{editAid})' class='btn btn-xs btn-info'>"
                                        + "<span class='fa fa-edit'></span>修改"
                                        + "</a>"
                                        + "</td>"
                                        + "<td>"
                                        + "<a href='javascript:void(0);' onclick='deleteAddress(#{deleteAid},#{isDefault})' class='btn btn-xs add-del btn-info'>"
                                        + "<span class='fa fa-trash-o'></span>删除"
                                        + "</a>"
                                        + "</td>"
                                        + "<td>"
                                        + "<a href='javascript:void(0);' onclick='setDefault(#{defaultAid})' class='btn btn-xs add-def btn-default'>设为默认</a>"
                                        + "</td>"
                                     + "</tr>"
                                //使用正则表达式替换获取该地址的aid值，#{aid}只是一个占位符的含义，没其他含义
                                str = str.replace("#{editAid}",address.aid)
                                str = str.replace("#{deleteAid}",address.aid)
                                str = str.replace("#{defaultAid}",address.aid)
                                str = str.replace("#{isDefault}",address.isDefault)
                                $("#address-list").append(str)
                            }
                            $(".add-def:eq(0)").hide();
                        }else{
                            let text = "<tr><td colspan='12' style='font-weight: bold;color: red;padding: 20px;font-size: medium'>"
                                    + "暂无收货地址，请先添加收货地址"
                                    + "</td></tr>"
                            $("#address-list").append(text)
                        }
                    },
                    error: function (error) {
                        alert(error.message)
                    }
                });
```

----

## 4.6 设置默认地址

### 4.6.1   后端-持久层

---

#### 1.编写sql语句

```sql
//检测当前用户想设置的地址是否存在
SELECT * FROM t_address WHERE aid = ?
//将所有的地址都设为非默认地址
UPDATE t_address SET is_default = 0 WHERE uid = ?
//更新选中地址的默认值
UPDATE t_address SET is_default = 1,modified_user = ?,modified_time=? 
WHERE aid = ?
```

---

#### 2.定义Mapper接口抽象方法

```java
    /**
     * 根据用户收货地址aid查询收货地址
     * @param aid 收货地址id
     * @return 返回地址
     */
    Address findByAid(Integer aid);

    /**
     * 根据用户uid将所有地址设为非默认地址
     * @param uid 用户uid
     * @return 返回影响行数
     */
    Integer updateNoneDefault(Integer uid);

    /**
     * 按照aid将该条收货地址设为默认地址
     * @param aid 用户收货地址aid
     * @param modifiedUser 执行操作的操作人
     * @param modifiedTime 执行操作的操作时间
     * @return 返回影响行数
     */
    Integer updateDefault(Integer aid, String modifiedUser, Date modifiedTime);
```

---

#### 3.编写Mapper接口的映射文件

```xml
<!--    Address findByAid(Integer aid);-->
    <select id="findByAid" resultType="com.zxl.store.entity.Address">
        SELECT * FROM t_user WHERE aid = #{aid}
    </select>
<!--    Integer updateNoneDefault(Integer uid);-->
    <update id="updateNoneDefault">
        UPDATE t_address SET is_default = 0 WHERE uid = #{uid}
    </update>
<!--    Integer updateDefault(Integer aid, String modifiedUser, Date modifiedTime);-->
    <update id="updateDefault">
        UPDATE t_address SET is_default = 1,modified_user = #{modifiedUser},modified_time=#{modifiedTime} WHERE aid = #{aid}
    </update>
```

---

#### 4.编写Mapper接口的单元测试

---

### 4.6.2 后端-业务层

#### 1.异常处理

更新时异常（已经存在，无需创建）

数据不存在异常

```java
/**
 * @author zxl
 * @description 用户地址未找到异常
 * @date 2022/11/3
 */
public class AddressNotFoundException extends ServiceException {}
```

---

#### 2.定义Service层抽象方法

```java
   /**
     *　修改用户选中地址为默认收货地址
     * @param uid 用户uid
     * @param aid 收货地址id
     * @param username 操作人
     * @return void
     */
    void setDefault(Integer uid,Integer aid,String username);
```

---

#### 3.编写方法的具体逻辑

```java
    @Override
    public void setDefault(Integer uid, Integer aid, String username) {
        Address res = addressMapper.findByAid(aid);
        if(res==null){
            throw new AddressNotFoundException("用户收货地址不存在");
        }
        //先将所有的地址设为非默认地址
        Integer row = addressMapper.updateNoneDefault(uid);
        if(row<1){
            throw new UpdateException("将所有地址设置为非默认地址时出现异常");
        }
        //按照aid将收货地址设置为默认地址
        Integer updateRow = addressMapper.updateDefault(aid, username, new Date());
        if(updateRow!=0){
            throw new UpdateException("设置默认地址时出现异常");
        }
    }
```

---

#### 4.单元测试

---

### 4.6.3 后端-控制层

#### 1.异常处理

将异常加入到全局处理

---

#### 2.请求设计

请求路径：/address/set_default/{aid}

请求参数：@PathVariable("aid")Integer aid,HttpSession session

请求类型：post

响应类型：JsonResult< void>

---

#### 3.处理请求，编写控制层

```java
    /**
     * 处理设置默认地址的请求
     * @param aid 被设置为默认的收货地址id
     * @param session 项目启动时生成的session
     * @return 返回void
     */
    @RequestMapping(value = "/set_default/{aid}",method = RequestMethod.POST)
    public JsonResult<Void> setDefault(@PathVariable("aid")Integer aid,HttpSession session){
        //获取uid，username
        Integer uid = getUserIdFromSession(session);
        String username = getUsernameFromSession(session);
        //设置默认
        addressService.setDefault(uid,aid,username);
        return new JsonResult<>(OK);
    }
```

---

#### 4.  前端页面

```javascript
            //为设置默认按钮绑定事件
            function setDefault(aid) {
                if(confirm("确定要这条收货地址设为默认地址吗？")){
                    $.ajax({
                        url: "/address/set_default/"+aid,
                        type: "POST",
                        dataType: "JSON",
                        success: function(json) {
                            if (json.state == 200) {
                                alert("设置成功");
                                location.reload();
                            } else {
                                alert("设置默认收货地址失败！" + json.message);
                            }
                        },
                        error: function(json) {
                            alert("您的登录信息已经过期，请重新登录！HTTP响应码：" + json.status);
                            location.href = "login.html";
                        }
                    });
                }
            }
```

---

## 4.7 删除地址

### 4.7.1 后端-持久层

---

#### 1.编写sql语句

```sql
//根据aid删除用户收货地址的sql
DELETE FROM t_address WHERE aid = #{aid}
```

---

#### 2.定义Mapper接口抽象方法

```java
    /**
     * 按照aid删除用户的收货地址
     * @param aid 用户的收货地址id
     * @return 返回影响行数
     */
    Integer DeleteAddressByAid(Integer aid);
```

---

#### 3.编写Mapper接口的映射文件

```xml
<!--    Integer DeleteAddressByAid(Integer aid);-->
    <delete id="DeleteAddressByAid">
        DELETE * FROM t_address WHERE aid = #{aid}
    </delete>
```

---

#### 4.单元测试

----

### 4.7.2 后端-业务层

#### 1.异常控制

删除时异常

```java
/**
 * @author zxl
 * @description 数据库删除时异常
 * @date 2022/11/4
 */
public class DeleteException extends ServiceException {}
```

---

#### 2.定义抽象方法

```java
    /**
     * 按照收货地址id删除收货地址
     * @param aid 收货地址id
     */
    void deleteAddressByAid(Integer aid, Integer uid, String username);
```

---

#### 3.编写实现逻辑

```java
    @Override
    public void deleteAddressByAid(Integer aid,Integer uid,String username) {
        //需要先判断用户有多少条地址
        Integer count = addressMapper.userAddressCount(uid);
        //判断用户收货地址是否存在
        Address byAid = addressMapper.findByAid(aid);
        if(byAid==null)throw new AddressNotFoundException("用户收货地址不存在");
        //如果地址多条 而且当前要删除的地址为默认地址
        if(count>1&&byAid.getIsDefault()==1){
            //先删除当前地址
            //按照用户aid删除
            Integer row = addressMapper.DeleteAddressByAid(aid);
            //判断是否出现异常
            if(row!=1)throw new DeleteException("删除用户收货地址时出现未知异常");
            //查询所有的地址，由于查询是按照创造时间排序的
            List<Address> byUid = addressMapper.findByUid(uid);
            //设置默认地址
            addressMapper.updateDefault(aid,username,new Date());
            return;
        }
        //如果只有一条，就不用判断是否需要再次设置默认地址
        //按照用户aid删除
        Integer row = addressMapper.DeleteAddressByAid(aid);
        //判断是否出现异常
        if(row!=1)throw new DeleteException("删除用户收货地址时出现未知异常");
    }
```

---

#### 4.单元测试

---

### 4.7.3 后端-控制层

#### 1.异常处理

将异常加入全局处理

---

#### 2.设计请求

请求路径：/address/delete_address/{aid}

请求参数：@PathVariable("aid")Integer aid,HttpSession session

请求类型：post

响应类型：JsonResult< void>

---

#### 3.处理请求，编写控制方法

```java
    /**
     * 处理删除地址的请求
     * @param aid 需要删除的收货地址的id
     * @return 返回OK
     */
    @RequestMapping(value = "/delete_address/{aid}",method = RequestMethod.POST)
    public JsonResult<Void> deleteAddressByAid(@PathVariable("aid")Integer aid,HttpSession session){
        //获取uid
        Integer uid = getUserIdFromSession(session);
        //获取username
        String username = getUsernameFromSession(session);
        //执行删除
        addressService.deleteAddressByAid(aid,uid,username);
        //执行成功返回数据
        return new JsonResult<>(OK);
    }
```

---

#### 4.前端页面

```javascript
            //为删除按钮定义事件
            function deleteAddress(aid) {
                if(confirm("确定要删除这条收货地址吗?")){
                    $.ajax({
                        url: "/address/delete_address/"+aid,
                        type: "POST",
                        dataType: "JSON",
                        success: function(json) {
                            if (json.state == 200) {
                                location.reload();
                            } else {
                                alert("删除收货地址失败！" + json.message);
                            }
                        },
                        error: function(xhr) {
                            alert("您的登录信息已经过期，请重新登录！HTTP响应码：" + xhr.status);
                            location.href = "login.html";
                        }
                    });
                }
            }
```

---

