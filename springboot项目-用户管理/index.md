# SpringBoot项目-用户管理

# 2环境搭建

## 2.1基本环境:

1.JDK1.8  
2.Maven 3.6.1    
3.Mysql 8.0.28  
4.IDEA 2019.2.4  

___  

# 3.用户管理

## 3.1 创建数据库

```sql
CREATE DATABASE IF NOT EXISTS `computer_store` CHARACTER SET 'utf8';
```

___

## 3.2 创建数据表

```sql
CREATE TABLE t_user (
    uid INT AUTO_INCREMENT COMMENT '用户id',
    username VARCHAR(20) NOT NULL UNIQUE COMMENT '用户名',
    password CHAR(32) NOT NULL COMMENT '密码',
    salt CHAR(36) COMMENT '盐值',
    phone VARCHAR(20) COMMENT '电话号码',
    email VARCHAR(30) COMMENT '电子邮箱',
    gender INT COMMENT '性别:0-女，1-男',
    avatar VARCHAR(50) COMMENT '头像',
    is_delete INT COMMENT '是否删除：0-未删除，1-已删除',
    created_user VARCHAR(20) COMMENT '日志-创建人',
    created_time DATETIME COMMENT '日志-创建时间',
    modified_user VARCHAR(20) COMMENT '日志-最后修改执行人',
    modified_time DATETIME COMMENT '日志-最后修改时间',
    PRIMARY KEY (uid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

考虑到每个表中都有固定的四个字段,可以使用一个java基类表示  

___

## 3.3 创建实体类

```java
//用于与用户四个字段所形成映射关系的基类
public class BaseEntity  implements Serializable {
    private String createdUser;//日志创建人
    private Date createdTime;//日志创建时间
    private String modifiedUser;//日志最后修改人
    private Date modifiedTime;//日志最后修改时间
}
//对应数据表的User实体类
public class User extends BaseEntity implements Serializable {
    private Integer uid;//用户id
    private String username;//用户名
    private String password;//密码
    private String salt;//盐值
    private String phone;//电话号码
    private String email;//电子邮箱
    private Integer gender;//性别:0-女，1-男
    private String avatar;//头像
    private Integer isDelete;//是否删除：0-未删除，1-已删除
}
```

___

## 3.4 注册

### 3.4.1 后端持久层(使用mybatis)

#### 1 编写sql语句

```sql
#增加用户的sql语句
INSERT INTO t_user(username,password,)VALUES (？，？)
#查询用户是否存在（username在数据库由UNIQUE修饰）
SELECT * FROM t_user WHERE username = ?
```

___

#### 2 定义mapper接口和抽象方法

由于项目可能有多个mapper接口，所以在项目目录下创建一个mappers包，用于管理所有mapper接口  
并在SpringBoot启动类上添加MapperScan注解扫描mapper包或直接在接口说使用@Mapper注解

```java
/*用户模块持久层接口*/
//@Mapper
public interface UserMapper {
    /**
     * 插入用户数据
     * @param user 用户数据
     * @return 返回影响行数
     */
    Integer insert(User user);

    /**
     * 根据用户名查询用户数据（数据库中用户名唯一）
     * @param username 用户名
     * @return 返回用户或者null
     */
    User findByUsername(String username);
}
```

___

#### 3 编写映射文件

项目可能有多个Mapper映射文件，所以需要在项目的resource下创建一个mappers包便于管理。

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace属性:指定当前映射文件和哪个接口映射-->
<mapper namespace="com.zxl.store.mappers.UserMapper">
    <!--Integer insert(User user);-->
    <!--
        useGeneratedKeys 开启某(主键)个字段的值递增
        keyProperty 表示将表中的xxx字段作为主键
    -->
    <insert id="insert" useGeneratedKeys="true" keyProperty="uid">
        INSERT INTO t_user(
        username,password,
        salt,phone,
        email,gender,
        avatar,is_delete,
        created_user,
        created_time,
        modified_user,
        modified_time
        )VALUES (
        #{username},#{password},
        #{salt},#{phone},
        #{email},#{gender},
        #{avatar},#{isDelete},
        #{createdUser},
        #{createdTime},
        #{modifiedUser},
        #{modifiedTime}
        )
    </insert>
    <!--User findByUsername(String username);-->
    <!--
        ResultType 表示查询的结果类型
        ResultMap 当表的字段和类的对象属性字段名称不一致时，来自定义结果的映射规则
    -->
    <select id="findByUsername" resultType="com.zxl.store.pojo.User">
        SELECT * FROM t_user WHERE username = #{username}
    </select>
</mapper>
```

```properties
#针对t_user与User类的名称不对应问题，
#可以在application.properties下针对mybatis开启
mybatis.configuration.map-underscore-to-camel-case=true
```

____

#### 4 将mapper映射文件的位置在yml配置文件中进行对应的设置

将mapper的映射文件路径添加在application.properties配置文件中进行配置

```properties
mybatis.mapper-locations=classpath:mappers/*.xml
```

在application.properties中配置数据库连接，否则无法连接数据库
(此处以使用mysql为例子设置，具体数据库请参考自己使用的数据库）

```properties
pring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=123
spring.datasource.url=jdbc:mysql://localhost:3306/store
```

____

#### 5 单元测试

进行单元测试，建议尽可能完成所有测试

___

### 3.4.2 后端-业务层

业务层的包下主要有ex、impl、interface，其中ex放置各种异常处理类，impl中放置interface的实现类  

___

#### 1.规划异常处理机制

考虑到在后端处理业务的过程中，会出现各种异常情况，如执行过程中数据库宕机、用户名重复等。 
虽然java在异常处理机制已经很完善，以上的情况都是抛出RuntimeException异常，对定位异常不够明确。  
因此在业务层的制定中，需要考虑对异常的定义处理。  
在业务层制定一个继承RuntimeException异常的异常类ServiceException，再让具体的异常继承这个异常。  

```java
/*业务层异常的基类*/
public class ServiceException extends RuntimeException {

    public ServiceException() {
        super();
    }

    public ServiceException(String message) {
        super(message);
    }

    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }

    public ServiceException(Throwable cause) {
        super(cause);
    }

    protected ServiceException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

根据业务层不同的 来详细定义具体异常的类型，统一的继承ServiceException异常基类

```java
//用户未找到异常
public class UserNotFoundException extends ServiceException {}
//用户名重复异常
public class UsernameDuplicateException extends ServiceException {}
//插入时未知异常
public class InsertException  extends ServiceException{}
//验证码不匹配异常
public class ValidCodeNotMatchException extends ServiceException {}
```

___

#### 2 定义业务层接口和抽象方法

```java
public interface IUserService {
    /**
     * 处理用户注册
     * @param user 用户信息
     */
    void userRegister(User user);
}
```

#### 3 定义业务层接口的实现类

补全五个字段:
is_Delete:便于逻辑删除  
create_user:注册记录创建人  
create_time:注册日期  
modified_user:增删改操作人  
modified_time:增删改时间  
便于后期数据库管理

```java
@Service
public class IUserServiceImpl implements IUserService {
    @Autowired
    private UserMapper userMapper;

    //处理用户注册
    @Override
    public void userRegister(User user) {
        //首先判断用户名是否在数据库中重复使用
        User res = userMapper.findUserByUsername(user.getUsername());
        //重复的情况下抛出异常
        if(res!=null){
            throw new UsernameDuplicateException("用户名已被注册");
        }
        //加密处理:md5算法
        //串+password+串--->md5，连续加载3次
        //盐值+password+盐值 --->盐值随机字符串
        //记录旧密码
        String oldPass = user.getPassword();
        //使用UUID获取salt
        String salt = UUID.randomUUID().toString().toUpperCase();
        //进行加密操作
        String newPass = getMD5Password(oldPass,salt);
        //对User进行补全
        user.setSalt(salt);
        user.setPassword(newPass);
        //修改逻辑删除判定
        user.setIsDelete(0);
        //补全四个操作字段
        Date currentTime = new Date();
        user.setCreatedTime(currentTime);
        user.setCreatedUser(user.getUsername());
        user.setModifiedTime(currentTime);
        user.setModifiedUser(user.getUsername());

        //调用插入方法 插入用户数据
        Integer row = userMapper.addUser(user);
        //判断插入结果
        if(row!=1){
            throw new InsertException("处理用户注册过程出现未知异常");
        }
    }

    /*md5加密*/
    private String getMD5Password(String password,String salt){
        //加密算法
        //加密之后的密匙
        for (int i = 0; i < 3 ; i++) {
            password = DigestUtils.md5DigestAsHex((salt + password + salt).getBytes()).toUpperCase();
        }
        return password;
    }

}
```

___

#### 4  业务层进行单元测试

___

### 3.4.3 后端-控制层

#### 1 设置返回响应信息给前端的基类

```java
/**
 * @author zxl
 * @description 相应数据给前端
 * @date 2022/10/30
 */
@Data
public class JsonResult<E> {
    //响应状态码 200-成功 4000-用户名重复 5000-数据库或服务器异常
    private int status;
    //响应信息
    private String message;
    //响应数据
    private E data;

    public JsonResult() {
    }

    public JsonResult(int status) {
        this.status = status;
    }

    public JsonResult(Throwable e) {
        this.message = e.getMessage();
    }

    public JsonResult(int status, E data) {
        this.status = status;
        this.data = data;
    }
}
```

___

#### 2 设计请求

请求路径：/user/reg

请求参数：User user,HttpSession session,String code

请求类型：post

响应结果：JsonResult< Void>

#### 3 设计控制层

设计一个BaseController处理全局所有自定义的异常

```java
/*控制层的基类*/
public class BaseController {
    /*操作成功状态码*/
    public static final int OK = 200;
    /**
     * 1.当出现了value内的异常之一，就会将下方的方法作为新的控制器方法进行执行
     *   因此该方法的返回值也同时是返回给前端的页面
     * 2.此外还自动将异常对象传递到此方法的参数列表中，这里使用Throwable e来接收
     **/
    @ExceptionHandler(ServiceException.class) //统一处理抛出的异常
    public JsonResult<Void> handleException(Throwable e){
        JsonResult<Void> result = new JsonResult<>(e);
        if (e instanceof UsernameDuplicateException){
            result.setStatus(4000); //表示用户名重复
            result.setMessage(e.getMessage());
        }else if (e instanceof UserNotFoundException){
            result.setStatus(4001); //表示用户数据不存在
            result.setMessage(e.getMessage());
        }else if (e instanceof InsertException){
            result.setStatus(5000); //数据库或服务器有问题
            result.setMessage(e.getMessage());
        }
        //返回异常处理结果
        return result;
    }
}
```

___

创建一个UserController处理注册请求

UserController继承了BaseController也就间接拥有了BaseController的属性和方法

```java
@RestController
@RequestMapping("users")
public class UserController extends BaseController{
    @Autowired
    private IUserService iUserService;

    //注册用户
    @RequestMapping(value = "reg",method = RequestMethod.POST)
    //@ResponseBody//表示此方法的响应结果以json格式进行数据的响应给到前端
    public JsonResult<Void> reg(User user){
        iUserService.reg(user);
        return new JsonResult<>(OK,"注册成功");
    }
}
```

___

#### 前端页面

前端页面只需要将表单通过ajax异步向后端服务器发送即可(目标文件:register.html)

只需要通过JavaScript给用户注册按钮绑定事件即可

当前功能:

                1.注册信息空缺检测

                2.用户名是否合规检测

                3.添加验证码

                4.验证输入密码是否一致

                5.表单提交

```javascript
    <script type="text/javascript">
        $(document).ready(function () {
            //验证信息和发送ajax注册用户请求
            //给用户注册绑定点击事件
            $("#btn-reg").click(function () {
                let name = $("#username").val();
                let pwd = $("#password").val();
                let rePwd = $("#rePwd").val();
                let codeStr = $("#code").val();
                //去掉验证码前后空格
                codeStr = $.trim(codeStr);
                if (name == "" || pwd == "" || rePwd == "" || codeStr == "") {
                    $("#error-msg").text("请先填写需要注册的信息！");
                    return false;
                }
                //验证用户名是否符合规则
                let nameCheck = /^\w{5,12}$/;
                let username = $("#username").val();
                if (!(nameCheck.test(username))) {
                    $("#error-msg").text("用户名必须是5-12位的字母和数字");
                    return false;
                } else {
                    $("#error-msg").empty()
                }

                //验证密码是否符合规则
                let passCheck = /^\w{5,12}$/;
                let password = $("#password").val();
                if (!passCheck.test(password)) {
                    $("#error-msg").text("密码必须是5-12位的字母和数字");
                    return false;
                } else {
                    $("#error-msg").empty()
                }

                //验证确认密码和密码是否相同
                let rePass = $("#rePwd").val();
                if (rePass !== password) {
                    $("#error-msg").text("密码不一致");
                    return false;
                } else {
                    $("#error-msg").empty()
                }

                $.ajax({
                    url: "/user",
                    type: "post",
                    data: $("#form-reg").serialize(), //获取表单的所有内容
                    dataType: "json",
                    success: function (res) {
                        if (res.status === 200) {
                            alert("注册成功！")
                            location.href = "login.html"
                        } else {
                            $("#error-msg").html(res.message)
                        }
                    },
                    error: function (error) {
                        alert(error.status + "错误,服务器出现故障，请等待攻城狮修复！！")
                    }
                });
            });
        //显示或隐藏密码的方法
        function showPasswordOrNot(eleId,imgId){
            let pwd = document.getElementById(eleId)
            let img = document.getElementById(imgId)
            if (pwd.type == "password"){
                pwd.type = "text";
                img.src = "../images/img/close.jpeg"
            }else {
                pwd.type = "password";
                img.src = "../images/img/open.jpeg"
            }
        }

        //给图片验证码绑定点击事件，刷新验证码
        function reFlashImg(imgId) {
            let kaptcha = document.getElementById(imgId)
            kaptcha.src = "/kaptcha/kaptcha-image?time="+ new Date();
        }
    });
    </script>
```

___

## 3.5 用户登录

### 3.5.1 后端-持久层

持久层可以利用上面写好的sql语句判断用户是否存在

1.sql语句可以使用上面的

2.Mapper接口用上面的

3.Mapper接口的映射文件可以使用上面的

4.单元测试

### 3.5.2 后端-业务层

#### 1.规划异常

创建两个异常类继承ServiceException基类

```java
// 表示用户名不存在的异常
public class UserNotFoundException extends ServiceException {}
//表示密码错误的异常
public class PasswordNotMatchException extends ServiceException {}
```

___

#### 2.编写接口抽象方法

```java
     /**
     * 用户登陆操作
     * @param user 用户信息
     * @return 返回用户
     */
    User userLogin(User user);
```

____

#### 3.实现类内具体的业务处理流程

```java
     //处理用户登陆
    @Override
    public User userLogin(User user) {
        //用户名
        String username = user.getUsername();
        //密码
        String password = user.getPassword();
        //查询用户是否在数据库中
        User res = userMapper.findUserByUsername(username);
        //判断结果为空或者逻辑删除
        if(res==null||(res.getIsDelete()==1)){
            throw new UserNotFoundException("用户数据不存在");
        }
        //密码校验
        String salt = user.getSalt();
        String databasePass = res.getPassword();
        //获取加密密码
        String md5Password = getMD5Password(password, salt);
        //
        if(!(md5Password.equals(databasePass))){
            throw new PasswordNotMatchException("密码错误");
        }
        //密码正确返回查询结果
        //将查询结果中的uid、username、avatar封装到新的user对象中
        User ret = new User();
        user.setUid(res.getUid());
        user.setUsername(res.getUsername());
        user.setAvatar(res.getAvatar());

        return ret;
    }
```

___

#### 进行单元测试

____

### 3.5.3 后端-控制层

#### 1.在全局的BaseController中添加异常处理

____

#### 2.设计请求

请求路径：/user/login

请求参数：User user, HttpSession session,String code

请求类型：get

响应结果：JsonResult< User>

___

#### 3.  处理请求

```java
    //用户登陆
    @GetMapping
    public  JsonResult<User> userLogin(User user,HttpSession session,String code){
        //从session取出验证码
        String validCode = (String) session.getAttribute(Constants.KAPTCHA_SESSION_KEY);
        //判断验证码是否正确
        if (!validCode.equals(code)) {
            throw new ValidCodeNotMatchException("验证码错误,请重试！");
        }
        //执行登陆操作
        User LoginUser = userService.userLogin(user);
        //将用户名和uid保存到session中
        session.setAttribute("uid",LoginUser.getUid());
        session.setAttribute("username", LoginUser.getUsername());
        //返回数据
        return new JsonResult<>(OK,LoginUser);
    }
```

___

#### 4.单元测试

____

### 3.5.4 前端页面

登录成功以及登录成功后需要做的事情：

    需要在登录成功后跳转至首页

    window.location.href = “xxx.html” 这个直接跳转到指定页面

    保存用户信息到session域中

##### 3.5.4.1 保存在前端的会话窗口中，供前端页面使用

```javascript
                    $.ajax({
                        url : "/user",
                        type: "get",
                        data: $("#form-login").serialize(), //获取表单的所有内容
                        dataType: "json",
                        success: function (res) {
                            if (res.status === 200){
                                alert("登录成功！");
                                //前往首页
                                window.location.href="index.html";
                                //将用户信息存入session域中
                                sessionStorage.setItem("user",JSON.stringify(res.data));
                            }else {
                                $("#error-msg").html(res.message)
                            }
                        },
                        error: function (error) {
                            alert(error.status + "错误,服务器出现故障，请等待攻城狮修复！！")
                        }
                    });
```

____

##### 3.5.4.2 保存在工程项目的session中，供整个工程使用

session对象主要存在服务器端，可以用于保存服务器的临时数据的对象，也可用于拦截器的拦截请求

```java
public class BaseController {
    /**
     * Description : 从session中获取用户uid
     * @param session springboot启动时生成的session对象
     **/
    public final Integer getUserIdFromSession(HttpSession session){
        String uidStr = session.getAttribute("uid").toString();
        return Integer.valueOf(uidStr);
    }

    //从session中获取用户username
    public final String getUsernameFromSession(HttpSession session){
        return session.getAttribute("username").toString();
    }
}
```

___

##### 3.5.4.3 拦截器，对每个访问的页面进行拦截判断，没有登录则重定向至登录页面

在interceptor包下自定义拦截器类，实现HandleInterceptor接口，实现此接口的方法

```java
public class LgoinInterceptor implements HandlerInterceptor {
    //在调用所有处理请求的方法之前被自动调用执行的方法

    /**
     * 检测全局Session对象中是否有Uid数据，如果有放行，如果没有重定向到登陆页面
     *
     * @param request  请求对象
     * @param response 响应对象
     * @param handler  处理器
     * @return 如果返回值为true--> 放行  如果false--> 拦截
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        //获取项目工程的session
        HttpSession session = request.getSession();

        if (session.getAttribute("uid") != null) { //说明此时已登录
            return true;
        } else {//说明未登录
                //重定向至登录页面
            response.sendRedirect("/web/login.html");
            return false;
        }
    }

    //ModelAndView对象返回之后被调用的方法
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    //在整个请求所有关联的资源被执行完毕最后所执行的方法
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

___

##### 3.5.4.4 在config包下创建自定义配置类，实现WebMvcConfigurer接口，将拦截器添加到容器中

​指定拦截规则【如果是拦截所有，静态资源也会被拦截，所以要指定白名单和黑名单】

拦截器也要放行接口的请求，不然就报错

```java
@Configuration//加载当前的拦截器并进行注册
//处理器拦截器的注册
public class LoginInterceptorConfigurer implements WebMvcConfigurer {
    //将自定义的拦截器进行注册
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //自定义一个拦截器对象
        HandlerInterceptor interceptor = new LgoinInterceptor();
        //配置白名单
        List<String> patterns = new ArrayList<>();
        patterns.add("/bootstrap3/**");
        patterns.add("/css/**");
        patterns.add("/images/**");
        patterns.add("/js/**");
        patterns.add("/web/register.html");
        patterns.add("/web/login.html");
        patterns.add("/web/index.html");
        patterns.add("/web/product.html");
        patterns.add("/users/**");
        patterns.add("/kaptcha/**");
        patterns.add("/address/**");
        patterns.add("/cart/**");
        patterns.add("/district/**");
        patterns.add("/product/**");
        //向注册器对象添加拦截器
        registry.addInterceptor(interceptor)
                .addPathPatterns("/**")//要拦截的Url是什么
                .excludePathPatterns(patterns);
    }
}
```

___

## 3.6 修改密码

### 3.6.1 后端-持久层

#### 1.编写sql语句

```sql
SELECT * FROM t_user WHERE uid = #{uid}

UPDATE t_user SET password=#{password},
                  modified_time=#{modifiedTime},
                  modified_user=#{modifiedUser}
                  WHERE uid = #{uid}
```

____

#### 2.定义mapper接口的抽象方法

```java
    /**
     * 根据用户的id查询用户数据
     * @param uid 用户Uid
     * @return 返回用户数据或者null
     */
    User findByUid(Integer uid);

    /**
     * 根据用户Uid修改密码
     * @param uid   用户Uid
     * @param password  用户输入的新密码
     * @param modifiedUser  表示修改的执行者
     * @param modifiedTime  表示修改的时间
     * @return
     */
    Integer updatePasswordByUid(Integer uid,
                                String password,
                                String modifiedUser,
                                Date modifiedTime);
```

___

#### 3.编写Mapper接口映射

```xml
    <!--User findByUid(Integer uid);-->
    <select id="findByUid" resultType="com.zxl.store.entity.User">
        SELECT * FROM t_user WHERE uid = #{uid}
    </select>
    <!--    Integer updatePasswordByUid(Integer uid,
                                String password,
                                String modifiedUser,
                                Date modifiedTime);-->
    <update id="updatePasswordByUid" >
        UPDATE t_user SET password=#{password},
                          modified_time=#{modifiedTime},
                          modified_user=#{modifiedUser}
                          WHERE uid = #{uid}
    </update>
```

___

#### 4.测试

___

### 3.6.2 后端-业务层

1.异常处理，创建一个表示密码不匹配的异常，比如原密码不对

2.定义Userservice的抽象方法

3.编写实现类实现接口方法的业务处理逻辑

```java
1.密码不匹配错误
public class PasswordNotMatchException extends ServiceException {}

2.IUserService的抽象方法
    /**
     * 更改用户密码
     * @param uid 用户id
     * @param username 用户名
     * @param oldPassword 用户的旧密码
     * @param newPasswrod 用户的新密码
     */
    void changePasswrod(Integer uid,String username,String oldPassword,String newPasswrod);

3.IUserServiceImpl 实体类的方法
    @Override
    public void changePasswrod(Integer uid, String username, String oldPassword, String newPasswrod) {
        //先查询用户数据是否为空或者逻辑删除
        User res = userMapper.findByUid(uid);
        if(res==null||res.getIsDelete()==1){
            throw new UserNotFoundException("用户数据不存在");
        }
        //数据库密码和旧密码对比
        String dataPassword = res.getPassword();
        String md5Password = getMD5Password(oldPassword, res.getSalt());
        if(!dataPassword.equals(md5Password)){
            throw new PasswordNotMatchException("密码不匹配");
        }
        //插入新密码插入数据库，更新操作人和操作时间
        String newPass = getMD5Password(newPasswrod, res.getSalt());
        Integer row = userMapper.updatePasswordByUid(uid, newPass, username, new Date());
        if(row!=1){
            throw new InsertException("更新数据时产生未知异常");
        }
    }
```

___

### 3.6.3   后端-控制层

#### 1.在全局异常处理机制中添加对业务层异常的处理

##### 2.设计请求

请求路径 /user/change_password

请求参数                        @RequestParam("oldPassword") String oldPassword,  
                                       @RequestParam("newPassword") String newPassword,  
                                       HttpSession session

请求类型 post

响应类型 JsonResult< Void>

##### 3.处理请求

```java
    //用户更改密码
    @RequestMapping(value = "/change_password",method = RequestMethod.POST)
    public JsonResult<Void> changePassword(@RequestParam("oldPassword") String oldPassword,
                                           @RequestParam("newPassword") String newPassword,
                                           HttpSession session){
        //获取Uid
        Integer uid = getUserIdFromSession(session);
        String username = getUsernameFromSession(session);
        userService.changePasswrod(uid,username,oldPassword,newPassword);

        //在用户修改密码之后清除session中保存的密码
        session.setAttribute("uid",null);
        return new JsonResult<>(OK,"修改密码成功");
    }
```

___

### 3.6.4 前端页面

1.在用户修改完密码之后，重新定向为index.html

2.在后端控制层清楚掉uid值，使用户重新登陆

```javascript
        <script type="text/javascript">
            $(document).ready(function () {
                $("#btn-change-password").click(function () {
                    if ($("#oldPwd").val() == "" || $("#newPwd").val() == "" || $("#rePwd").val() == ""){
                        $("#error-msg").text("请填写完信息后再提交！");
                        return false;
                    }
                    //验证密码是否符合规则
                    let passCheck = /^\w{5,12}$/;
                    let password = $("#newPwd").val();
                    if (!passCheck.test(password)){
                        $("#error-msg").text("新密码必须是5-12位的字母和数字");
                        return false;
                    }else {
                        $("#error-msg").empty()
                    }

                    //验证确认密码和密码是否相同
                    let rePass = $("#rePwd").val();
                    if (rePass !== password){
                        $("#error-msg").text("密码不一致");
                        return false;
                    }else {
                        $("#error-msg").empty()
                    }
                    $("#btn-change-password").click(function() {
                        $.ajax({
                            url: "/user/change_password",
                            type: "POST",
                            data: $("#form-change-password").serialize(),
                            dataType: "json",
                            success: function(json) {
                                if (json.state === 200){
                                    alert("密码已更改成功，请重新登录")
                                    //跳转至首页，让用户重新登录
                                    location.href = "login.html"
                                }else {
                                    $("#error-msg").html(json.message)
                                }
                            },
                            error: function (xhr) {
                                alert("您的登录信息已经过期，请重新登录！HTTP响应码：" + xhr.status);
                                location.href = "login.html";
                            }
                        });
                    });
                });
            });
        </script>
```

___

## 3.7 个人资料

#### 3.7.1 后端持久层

___

##### 1.编写Sql

需要更新phone、email、gender、modified_user ,modified_time这五个字段

```sql
//更新的sql语句
update t_user set phone = #{phone},
                  email = #{email},
                  gender = #{gender},
                  modified_user = #{modifiedUser},
                  modified_time = #{modifiedTime}
              where uid = #{uid}
```

___

##### 2.定义Mapper接口抽象方法

```java
    /**
     * 根据用户的id查询用户数据
     *
     * @param uid 用户Uid
     * @return 返回用户数据或者null
     */
    User findByUid(Integer uid);
     /**
     * 更新用户信息
     * @param user 用户数据
     * @return 返回影响的行数
     */
    Integer updateInfoByUid(User user);
```

___

##### 3.编写Mapper接口的映射文件

```xml
       <!--User findByUid(Integer uid);-->
    <select id="findByUid" resultType="com.zxl.store.entity.User">
        SELECT * FROM t_user WHERE uid = #{uid}
    </select>

    <update id="updateInfoByUid">
        UPDATE t_user
        SET
        <if test="phone!=null">phone = #{phone},</if>
        <if test="email!=null">email = #{email},</if>
        <if test="gender!=null">gender = #{gender},</if>
        modified_time = #{modifiedTime},
        modified_user = #{modifiedUser}
        WHERE uid = #{uid}
    </update>
```

___

##### 4.单元测试

____

#### 3.7.2 后端-业务层

##### 1.定义异常（无）

##### 2.定义IUserService接口的抽象方法

##### 3.实现抽象方法，编辑业务逻辑

```java
1.定义异常 （无）
2.定义IUserService接口的抽象方法
    /**
     * 根据用户的id查询用户的数据
     * @param uid 用户id
     * @return 返回查询到的用户 或者 null
     */
    User getByUid(Integer uid);

    /**
     *  更新用户的数据操作
     * @param uid 用户的id
     * @param username 用户名
     * @param user 用户对象数据
     */
    void changeInfo(Integer uid,String username,User user);

3.实现抽象方法，编写业务逻辑
   @Override
    public User getByUid(Integer uid) {
        User res = userMapper.findByUid(uid);
        if(res==null||res.getIsDelete()==1){
            throw new UserNotFoundException("用户数据不存在");
        }
        User usr = new User();
        //防止重要内容泄漏
        usr.setUsername(res.getUsername());
        usr.setPhone(res.getPhone());
        usr.setEmail(res.getEmail());
        usr.setGender(res.getGender());
        return usr;
    }

    /*User对象中的phone/email/gender 手动将uid/username/封装*/
    @Override
    public void changeInfo(Integer uid, String username, User user) {
        User res = userMapper.findByUid(uid);
        if(res==null||res.getIsDelete()==1){
            throw new UserNotFoundException("用户数据不存在");
        }
        user.setUid(uid);
        user.setUsername(username);
        user.setModifiedUser(username);
        user.setModifiedTime(new Date());

        Integer row = userMapper.updateInfoByUid(user);
        if(row!=1){
            throw new InsertException("更新时数据产生未知异常");
        }
    }
```

___

##### 4.单元测试

---

### 3.7.3 后端-控制层

#### 1.无异常，不需要处理

#### 2.设计请求

请求路径 /user/change_info

请求参数 User user,HttpSession session

请求类型 post

响应类型 JsonResult<User>

#### 3.处理请求

```java
    //修改用户信息
    @RequestMapping(value = "/change_info",method = RequestMethod.POST)
    public JsonResult<Void> changeInfo(User user,HttpSession session){
        //User数据只有四部分   用户电话邮箱性别
        System.out.println(user.getUsername()+user.getEmail()+ user.getPhone()+user.getGender());
        //Service内部已经重新写入
        userService.changeInfo(getUserIdFromSession(session),getUsernameFromSession(session),user);
        return new JsonResult<>(OK,"修改信息成功");
    }
    //获取用户信息
    @RequestMapping(value = "/get_by_uid",method = RequestMethod.GET)
    public JsonResult<User> getByUid(HttpSession session){
        Integer uid = getUserIdFromSession(session);

        User user = userService.getByUid(uid);

        //将用户名、id、电话、邮箱、性别进行回传
        User newUser = new User();
        newUser.setUsername(user.getUsername());
        newUser.setUid(user.getUid());
        newUser.setGender(user.getGender());
        newUser.setPhone(user.getPhone());
        newUser.setEmail(user.getEmail());
        newUser.setAvatar(user.getAvatar());
        return new JsonResult<>(OK,newUser);
    }
```

___

### 3.7.4 前端页面

1.第一个ajax请求在用户信息页面加载完成后自动发送，并根据返回值通过js的id选择器

​ 找到对应的元素并修改其属性值

2.第二个ajax请求在用户点击修改按钮之后先提示是否修改，再根据其选择进行处理，

​ 同理根据结果利用js的id选择器找到对应的元素并修改其属性值

```javascript
        <script type="text/javascript">
            $(document).ready(function() {
                $.ajax({
                    url: "/user/get_by_uid",
                    type: "GET",
                    dataType: "json",
                    success: function(json) {
                        if (json.state == 200) {
                            console.log("username=" + json.data.username);
                            console.log("phone=" + json.data.phone);
                            console.log("email=" + json.data.email);
                            console.log("gender=" + json.data.gender);

                            $("#username").val(json.data.username);
                            $("#phone").val(json.data.phone);
                            $("#email").val(json.data.email);

                            let radio = json.data.gender == 0 ? $("#gender-female") : $("#gender-male");
                            radio.prop("checked", "checked");
                        } else {
                            alert("获取用户信息失败！" + json.message);
                        }
                    }
                });

                //给用户更改信息绑定点击事件
                $("#btn-change-info").click(function () {
                    //根据用户选择状态决定是否发生ajax请求
                    if (confirm("确定要修改吗？")){
                        let phone = $("#phone").val();
                        let email = $("#email").val();
                        if (phone == "" || email == ""){
                            $("#error-msg").html("请先填写需要修改的信息！");
                            return false;
                        }
                        let checkPhone = /^[1][3,4,5,7,8][0-9]{9}$/;
                        if (!checkPhone.test(phone)){
                            $("#error-msg").html("手机号不符合要求！");
                            return false;
                        }
                        //验证邮箱是否符合规则
                        let checkEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/;
                        if (!checkEmail.test(email)){
                            $("#error-msg").html("邮箱不符合要求！");
                            return false;
                        }
                        $.ajax({
                            url : "/user/change_info",
                            type: "post",
                            data: $("#form-change-info").serialize(),//获取表单的所有内容
                            dataType: "json",
                            success: function (res) {
                                if (res.status === 200){
                                    alert("修改成功！")
                                    //网页刷新
                                    location.reload();
                                }else {
                                    $("#error-msg").html(res.message);
                                }
                            },
                            error: function (error) {
                                alert("服务器出现故障，请等待攻城狮修复！！")
                            }
                        });
                    }
                })
            });

        </script>
```

____

## 3.8 头像上传

### 3.8.1 后端-持久层

___

##### 1.编写sql

    对应的avatar字段保存头像地址

```sql
        UPDATE t_user
        SET
            avatar=#{avatar},
            modified_user=#{modifiedUser},
            modified_time=#{modifiedTime}

        WHERE
            uid = #{uid}
```

___

##### 2.定义Mapper接口的抽象方法

```java
    /**
     * 根据用户的Uid修改头像
     * @param uid   用户Uid
     * @param avatar  头像数据
     * @param modifiedUser 表示修改的执行者
     * @param modifiedTime 表示修改的时间
     * @return
     */
    Integer updateAvatarByUid(@Param("uid") Integer uid,
                              @Param("avatar") String avatar,
                              @Param("modifiedUser") String modifiedUser,
                              @Param("modifiedTime") Date modifiedTime);
```

___

#### 3.编写Mapper接口的映射文件

```xml
    <update id="updateAvatarByUid">
        UPDATE t_user
        SET
            avatar=#{avatar},
            modified_user=#{modifiedUser},
            modified_time=#{modifiedTime}

        WHERE
            uid = #{uid}
    </update>
```

___

##### 4.单元测试

___

### 3.8.2 后端-业务层

____

##### 1.异常规划 例如用户数据不存在，服务器宕机等

##### 2.定义service层接口抽象方法

##### 3.实现类重写抽象方法，编写业务处理逻辑

```java
2.定义Service层接口抽象方法
    /**
     * 更新用户的头像操作
     * @param uid   用户id
     * @param avatar 用户头像路径
     * @param username  修改的执行者
     */
    void changeAvatar(Integer uid,String avatar,String username);
3.实现类重写抽象方法，编写业务层逻辑
    /**/
    @Override
    public void changeAvatar(Integer uid, String avatar, String username) {
        //查询当前的用户数据是否存在
        User res = userMapper.findByUid(uid);
        if(res==null||res.getIsDelete()==1){
            throw new UserNotFoundException("用户数据不存在");
        }
        Integer row = userMapper.updateAvatarByUid(uid, avatar, username, new Date());
        if(row!=1){
            throw new InsertException("更新时数据产生未知异常");
        }
    }
```

____

##### 4 .单元测试

____

### 3.8.3 后端-控制层

##### 1.处理控制层和业务层异常，将控制层异常加入全局异常处理@ExceptionHandler的值中

考虑到控制层接口前端数据也有可能出现异常，因此控制层也要进行异常控制

创建一个FileUploadException继承RunTimeException，其余异常继承此异常

①文件为空异常

②文件大小超出限制异常

③文件状态异常

④文件类型不符异常

⑤文件读取IO异常

##### 2.设计请求

请求路径：/change_avatar

请求参数：MultipartFile file，Httpsession session

请求类型：post

响应类型：JsonResult< Void>

##### 3.处理请求

①创建一个Controller专门处理文件的上传和下载

②将文件的下载地址保存到数据库

```java
/**
 * @author zxl
 * @description
 * @date 2022/10/30
 */
@RestController
@RequestMapping("/file")
public class FileController extends BaseController{

    @Autowired
    private IUserService userService;

    /*设置上传文件的最大值*/
    public static final int AVATAR_MAX_SIZE=10 * 1024 * 1024;
    /*限制上传文件的类型*/
    public static final List<String> AVATAR_TYPES = new ArrayList<>();
    static {
        AVATAR_TYPES.add("image/jpeg");
        AVATAR_TYPES.add("image/jpg");
        AVATAR_TYPES.add("image/png");
        AVATAR_TYPES.add("image/bmp");
        AVATAR_TYPES.add("image/gif");
    }
    /**
     * MultipartFile接口时SpringMVC提供的一个接口，这个接口为我们包装了
     * 获取文件数据(任何类型的文件File都可以),Springboot整合了SpringMVC
     * 只需要在处理请求的方法参数列表上声明一个参数为MultipartFile即可
     * @param session
     * @param file
     * @return
     */
    @PostMapping
    public JsonResult<String> changeAvatar(HttpSession session,
                                           @RequestParam("file") MultipartFile file){
        //判断文件是否为null
        if(file.isEmpty()){
            throw new FileEmptyException("文件为空");
        }
        //判断文件大小
        if(file.getSize()>AVATAR_MAX_SIZE){
            throw new FileSizeException("文件大小超出限制");
        }

        // 判断上传的文件类型是否超出限制
        String contentType = file.getContentType();
        // boolean contains(Object o)：当前列表若包含某元素，返回结果为true；若不包含该元素，返回结果为false
        if (!AVATAR_TYPES.contains(contentType)) {
            // 是：抛出异常
            throw new FileTypeException("不支持使用该类型的文件作为头像，允许的文件类型：" + AVATAR_TYPES);
        }
        //获取当前文件的绝对路径
        //String parent = session.getServletContext().getRealPath("upload");
        String parent = "/Users/zhaoxinlei/workspace/StoreRebuild/store/src/main/resources/static/avatar";
        System.out.println(parent);
        //保存头像文件的文件夹
        File dir = new File(parent);
        if(!dir.exists()){
            dir.mkdirs();
        }
        //保存头像文件的文件名
        String suffix ="";
        String originalFilename = file.getOriginalFilename();
        int beginIndex = originalFilename.lastIndexOf(".");
        if(beginIndex>0){
            suffix = originalFilename.substring(beginIndex);
        }
        String filename = UUID.randomUUID().toString() + suffix;

        // 创建文件对象，表示保存的头像文件
        File dest = new File(dir,filename);
        //执行保存文件操作
        try{
            file.transferTo(dest);
        }catch (IllegalStateException e){
            //抛出异常
            throw new FileStateException("文件状态异常，可能文件已被移动或者删除");
        }catch (IOException e){
            //抛出异常
            throw new FileUploadIOException("上传文件时读写错误,请稍后重新尝试");
        }
        //从Session中获取Uid和username
        Integer uid = getUserIdFromSession(session);
        String username = getUsernameFromSession(session);
        //将头像写入数据库
        userService.changeAvatar(uid,filename,username);
        //返回成功头像路径
        String avatar = "../avatar/"+filename;
        System.out.println(filename);
        System.out.println(avatar);
        return new JsonResult<>(OK,avatar);
    }
}
```

___

### 3.8.4. 前端页面

```javascript
        <script type="text/javascript">
            $(document).ready(function () {
                //网页加载完成之前自动发送ajax请求
                $.ajax({
                    url: "/user/get_by_uid",
                    type: "get",
                    dataType: "json",
                    success:function (res) {
                        //判断用户是首次注册还是老用户
                        if (res.data.avatar !== null &&res.data.avatar !== ""   ){
                            //设置用户头像
                            $("#img-avatar").attr("src","../avatar/"+res.data.avatar)
                        }else{
                            //设置为默认头像
                            $("#img-avatar").attr("src","../images/index/user.jpg")
                        }
                    },
                    error:function (err) {
                        alert(err.message())
                    }
                })
                $("#btn-change-avatar").click(function() {
                    $.ajax({
                        url: "/file",
                        type: "POST",
                        data: new FormData($("#form-change-avatar")[0]),
                        dataType: "JSON",
                        processData: false, // processData处理数据
                        contentType: false, // contentType发送数据的格式
                        success: function(json) {
                            if (json.state == 200) {
                                $("#img-avatar").prop("src", json.message);
                                console.log($("#img-avatar").prop("src"));
                            } else {
                                alert("修改失败！" + json.message);
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

----

