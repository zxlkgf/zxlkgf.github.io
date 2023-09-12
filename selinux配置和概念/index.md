# SELinux配置和概念


## 1.概述
### 1.1 概念
作为 Android 安全模型的一部分，Android 使用安全增强型 Linux (SELinux) 对所有进程强制执行强制访问控制 (MAC)，甚至包括以 Root/超级用户权限运行的进程（Linux 功能）。借助 SELinux，Android 可以更好地保护和限制系统服务、控制对应用数据和系统日志的访问、降低恶意软件的影响，并保护用户免遭移动设备上的代码可能存在的缺陷的影响。

SELinux 按照默认拒绝的原则运行：任何未经明确允许的行为都会被拒绝。SELinux 可按两种全局模式运行：
宽容模式：权限拒绝事件会被记录下来，但不会被强制执行。
强制模式：权限拒绝事件会被记录下来并强制执行。

可以通过
adb shell set enforece 0 ，将其设置为宽容模式
adb shell set enforece 1 ，将其设置为强制模式

--- 

### 1.2 MAC 和 DAC
DEC系统(自主访问控制),其存在所有权的概念，即特定的资源所有者可以控制该资源关联的访问权限。这种系统通常比较粗放。并且容易出现无意中提权的问题。MAC系统则会在每次收到访问请求时都先咨询核心机构，再做出决定。

### 1.3 类型,属性,别名和规则
Android 依靠 SELinux 的类型强制执行 (TE) 组件来实施其政策。
这表示所有对象（例如文件、进程或套接字）都具有相关联的类型。
例如，默认情况下，应用的类型为 untrusted_app。对于进程而言，其类型也称为域。可以使用一个或多个属性为类型添加注解。属性可用于同时指代多种类型。

对象会映射到类（例如文件、目录、符号链接、socket套接字），并且每个类的不同访问权限类型由权限表示。
例如，file类存在权限open。虽然类型和属性作为Android SELinux政策的一部分会进行定期更新，但权限和类是静态定义的，并且作为新Linux版本的一部分也很少进行更新。

政策规则采用以下格式：allow source target:class permissions;，其中：
source - 规则主题的类型（或属性）。谁正在请求访问权限？
目标 - 对象的类型（或属性）。对哪些内容提出了访问权限请求？
类 - 要访问的对象（例如，文件、套接字）的类型
权限 - 要执行的操作（或一组操作，例如读取、写入）
 
例如 我们可以在logcat内对tag:avc进行过滤，来看到不被允许的操作
```text
denied { set } for property=persist.vendor.vt.video_conference_support pid=12424 uid=1000 gid=1000 scontext=u:r:system_app:s0 tcontext=u:object_r:vendor_mtk_vendor_vt_prop:s0 tclass=property_service permissive=0'
```

可以看到上述avc输出：
source : u:r:system_app:s0 
tcontext : u:object_r:vendor_mtk_vendor_vt_prop:s0
tclass: property_service 
当前的SELinux的状态 permissive = 0 强制模式

---

#### 1.3.1 类型声明
区别客体类别。这里所说的类型指的是域类型和客体类型。
语法：type 类型名(例如system_prop)
    type 类型名 [属性集,] [属性集,].....
SELinux没有预定的类型，所以我们需要声明我们所需的类型。

---

#### 1.3.2 属性
规则默认是拒绝所有访问的，每一个访问都需要被声明。
如果我们想要让程序访问所有的文件，首先我们需要创建一个对应的域类型(system_app)，并允许它访问任何类型的文件。
```text
type system_app;  //声明域类型
allow system_app xxx_prop_t : file {read write open close}; 
//允许该域类型对xxx_prop具有读写打开关闭操作。(单个权限不需要{})
allow system_app http_file_t : file {read write open close};
```

然而system_app需要对多文件进行访问权限操作，这样我们就需要编写很多allow语句，所以我们需要使用属性。
attribute直接翻译过来叫属性， 主要用于在声明type的时候进行关联。 属性类似一个组，type关联属性之后， 新增的type就可以加入到这个属性组中，这个属性(组)拥有的权限， 新增的type也就自然拥有了这个权限），举例：
```text
type xxx_prop_t , coredomain, domain;
type http_file_t , domain, coredomain;
{ xxx_prop_t , http_file_t }都关联(加入)了domain属性组
```

这样我们需要让system_app对所有文件进行{read,write,open,close}操作的时候
可以直接使用如下语法：
```text
allow system_app domain: file {read write open close};
```

---

#### 1.3.3 宏
特别是对于文件访问权限，有很多种权限需要考虑。例如，read 权限不足以打开相应文件或对其调用 stat。为了简化规则定义，Android 提供了一组宏来处理最常见的情况。例如，若要添加 open 等缺少的权限，可以将上述规则改写为： 
allow appdomain app_data_file:file rw_file_perms;
如需查看实用宏的更多示例，请参阅 global_macros 和 te_macros 

---

### 1.4 安全上下文
调试SELinux政策或为文件添加标签时（通过file_contexts或运行ls -Z），可能会遇到安全上下文（也称为标签） 

安全上下文格式：user:role:type:sensitivity[:categories]
user: 用户，目前SEAndroid上只定义一个SELinux用户，值为u
role:角色，r（适用于主题）和 object_r（适用于对象） 
type: 主体/客体类型，一般定义在具体的te文件中
sensitivity[:categories]:安全级别
sensitivity:敏感度
category:类别。

---

### 1.5 te文件内容的语法规则
rule_name  source_type  target_type : class  permission_set
rule_name: 赋予权限的规则,
包含allow：允许
  dontaudit：不记录某项操作失败
  auditallow：记录操作
  neverallow：检查安全策略中是否存在违反neverallow规则的allow语句。

source_type: 访问target_type的主体或主体集合（域），可自定义
target_type:接受主体访问的客体或客体集合（域），可自定义
class : 客体资源类型，不同的资源类型具有不同访问权限，可自定义、可继承
perm_set：客体予以主体的权限说明。是class中具有的权限的子集

其中source_type和target_type可以按照上述的attribute配置集合。

---

## 2.SELinux的实现
### 2.1 SELinux的核心文件
如果需要集成启动SELinux,需要下载[最新Android内核](https://android.googlesource.com/kernel/common/),并对其中的system/sepolicy目录下的文件进行修改。这些修改后的文件再编译后会包含SELinux内核安全政策，并涵盖上游Android系统。

常情况下，您不能直接修改 system/sepolicy 文件，但您可以添加或修改自己的设备专用政策文件（位于 /device/manufacturer/device-name/sepolicy 目录中）。在 Android 8.0 及更高版本中，您对这些文件所做的更改只会影响供应商目录中的政策 
如需详细了解 Android 8.0 及更高版本中的公共 sepolicy 分离，请参阅在 Android 8.0 及更高版本中自定义 SEPolicy。

在Android8.0及其更高版本中,sepolicy位于AOSP中的以下位置：
system/sepolicy/public: 此目录可以处理privat和vendor之间的接口的所有内容。因此需要对其加以限制。
system/sepolicy/private:此目录下不可以访问vendor下的任何安全策略，其中的内容包含所有system的所有私有内容。
system/sepolicy/vendor: 此目录下不可访问private下的任何安全政策。属于供应商的安全政策。
device/manufacturer/device-name/sepolicy。包含设备专用策略，已经对sepolicy的自定义。

上下文描述文件:
详细上下文参考
查看上下文描述文件
file_contexts //系统中所有file_contexts安全上下文
seapp_contexts //app安全上下文
property_contexts //属性的安全上下文
service_contexts    //service文件安全上下文
genfs_contexts //虚拟文件系统安全上下文

---

### 2.1 构建和编译sepolicy
前提需要下载Android内核代码：  
[最新Android内核](https://android.googlesource.com/kernel/common/)  
参考：  
[构建sepolicy](https://source.android.com/docs/security/features/selinux/build?hl=zh-cn)  
[实现sepolicy](https://source.android.com/docs/security/features/selinux/implement?hl=zh-cn)  

---

## 3,调试验证
SELinux 日志消息中包含“avc:”字样，因此可使用 grep 轻松找到。
您可以通过运行 cat /proc/kmsg 来获取当前的拒绝事件日志，也可以通过运行 cat /sys/fs/pstore/console-ramoops 来获取上次启动时的拒绝事件日志 。

你也可以通过过滤tag:avc 在logcat上查看输出。
```text
avc: denied  { connectto } for  pid=2671 comm="ping" path="/dev/socket/dnsproxyd"
scontext=u:r:shell:s0 tcontext=u:r:netd:s0 tclass=unix_stream_socket
```

该输出的解读如下：
上方的 { connectto } 表示执行的操作。根据它和末尾的 tclass (unix_stream_socket)，您可以大致了解是对什么对象执行什么操作。在此例中，是操作方正在试图连接到 UNIX 信息流套接字。
scontext (u:r:shell:s0) 表示发起相应操作的环境，在此例中是 shell 中运行的某个程序。
tcontext (u:r:netd:s0) 表示操作目标的环境，在此例中是归 netd 所有的某个 unix_stream_socket。
顶部的 comm="ping" 可帮助您了解拒绝事件发生时正在运行的程序。在此示例中，给出的信息非常清晰明了。
 
具体调试策略:请参考[google sepolicy调试](https://source.android.com/docs/security/features/selinux/validate?hl=zh-cn)

---





















