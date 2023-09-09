# Okhttp详解


# 1.OKHttp详解

## 1.1 概述
OkHttp 是一套处理 HTTP 网络请求的依赖库，由 Square 公司设计研发并开源，目前可以在 Java 和 Kotlin 中使用

---

## 1.2 使用

request uri: http://guolin.tech/api/china  

```java
//在gradle.build内添加okhttp
//implementation 'com.squareup.okhttp3:okhttp:4.10.0'

//创建OKHttp client 
OkHttpClient client = new OkHttpClient();
//创建request
Request request = new Request.Builder().url(uri).get().build();
//为client和request创建一个call对象
Call call = client.newCall(request);
//为call添加callback并重写onFailure和onResponse方法
call.enqueue(new Callback() {
            @Override
            public void onFailure(@NonNull Call call, @NonNull IOException e){
            }
            @Override
            public void onResponse(@NonNull Call call, @NonNull Response response) throws IOException {
            }
        });
// 调用runOnUiThread(new Runable{})更新ui
```

---

## 1.3 问题点
### 1.3.1 谷歌要求使用默认加密连接，即Https
报错：java.net.UnknownServiceException: CLEARTEXT communication to guolin.tech not permitted by network security policy

原因: 为保证用户数据和设备的安全，Google针对下一代 Android 系统(Android P) 的应用程序，将要求默认使用加密连接，这意味着 Android P 将禁止 App 使用所有未加密的连接，因此运行 Android P 系统的安卓设备无论是接收或者发送流量，未来都不能明码传输。

具体可参考[谷歌网络安全配置](https://developer.android.google.cn/training/articles/security-config?hl=zh_cn)

解决方法：  
1. APP改用 https 请求
2. targetSdkVersion 降到27以下
3. 更改网络安全配置

方案3的更改网络配置具体操作
1. 在res/xml下创建一个network_security_config.xml，内容如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
        <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

2. 在 AndroidManifest.xml 文件下面配置network_security_config.xml即可
```xml
<application  
....
android:networkSecurityConfig="@xml/network_security_config"   
...   
/>
```

---

### 1.3.2 FATAL EXCEPTION: OkHttp Dispatcher
原因: OkHttp 的Response response.body().string()方法，只能调用一次。按照源码来看,在被调用读取buff内的字符之后，该资源会被关闭，如果二次调用string()就会导致异常。

---

## 1.4 显示的结果
获取json数据。 
```text
[{"id":1,"name":"北京"},{"id":2,"name":"上海"},{"id":3,"name":"天津"},{"id":4,"name":"重庆"},{"id":5,"name":"香港"},{"id":6,"name":"澳门"},{"id":7,"name":"台湾"},{"id":8,"name":"黑龙江"},{"id":9,"name":"吉林"},{"id":10,"name":"辽宁"},{"id":11,"name":"内蒙古"},{"id":12,"name":"河北"},{"id":13,"name":"河南"},{"id":14,"name":"山西"},{"id":15,"name":"山东"},{"id":16,"name":"江苏"},{"id":17,"name":"浙江"},{"id":18,"name":"福建"},{"id":19,"name":"江西"},{"id":20,"name":"安徽"},{"id":21,"name":"湖北"},{"id":22,"name":"湖南"},{"id":23,"name":"广东"},{"id":24,"name":"广西"},{"id":25,"name":"海南"},{"id":26,"name":"贵州"},{"id":27,"name":"云南"},{"id":28,"name":"四川"},{"id":29,"name":"西藏"},{"id":30,"name":"陕西"},{"id":31,"name":"宁夏"},{"id":32,"name":"甘肃"},{"id":33,"name":"青海"},{"id":34,"name":"新疆"}]
```

---
