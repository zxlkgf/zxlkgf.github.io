# SpringBoot项目-Kaptcha

# Kaptcha

## 1.1 Kaptcha简介

Kaptcha 是一个扩展自simplecaptcha的验证码库，默认情况下，Kaptcha非常易于设置和使用，并且默认输出会产生一个很难验证的验证码。默认情况下，它生成的验证码看起来与上面的非常相似。如果您想更改输出的外观，则有几个配置选项，并且该框架是模块化的，因此您可以编写自己的变形代码。

参考资料:( [kaptcha验证码使用](https://www.cnblogs.com/cynchanpin/p/6912301.html))

---

## 1.2 Kaptcha详细配置表

---

# 2 Maven依赖

```xml
        <!-- 验证码 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>kaptcha-spring-boot-starter</artifactId>
            <version>1.1.0</version>
        </dependency>
```

---

# 3.创建配置类

```java
package com.zxl.store.config;

import com.google.code.kaptcha.impl.DefaultKaptcha;
import com.google.code.kaptcha.util.Config;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Properties;

/**
 * @author zxl
 * @version 1.0
 * @description: kaptcha配置类
 * @date 2022/10/30
 */
@Slf4j
@Configuration
public class KaptchaConfig {
    //kaptcha
    @Bean
    public DefaultKaptcha getKaptcheCode() {
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        // 创建properties
        Properties properties = new Properties();
        //是否有边框 NO
        properties.setProperty("kaptcha.border", "no");
        //字体颜色 black
        properties.setProperty("kaptcha.textproducer.font.color", "black");
        //图片宽度 100
        properties.setProperty("kaptcha.image.width", "100");
        //图片高度 36
        properties.setProperty("kaptcha.image.height", "36");
        //字体大小 30px
        properties.setProperty("kaptcha.textproducer.font.size", "30");
        //图片样式 阴影
        properties.setProperty("kaptcha.obscurificator.impl", "com.google.code.kaptcha.impl.ShadowGimpy");
        //session key = code
        properties.setProperty("kaptcha.session.key", "code");
        //干扰实现类
        properties.setProperty("kaptcha.noise.impl", "com.google.code.kaptcha.impl.NoNoise");
        //背景渐变颜色 开始颜色
        properties.setProperty("kaptcha.background.clear.from", "232,240,254");
        //背景渐变颜色 结束颜色
        properties.setProperty("kaptcha.background.clear.to", "232,240,254");
        //验证码长度
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        //字体
        properties.setProperty("kaptcha.textproducer.font.names", "彩云,宋体,楷体,微软雅黑");
        //设置参数
        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
}
```

---

# 创建控制类

```java
package com.zxl.store.controller;

import com.google.code.kaptcha.Constants;
import com.google.code.kaptcha.Producer;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.imageio.ImageIO;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.awt.image.BufferedImage;

/**
 * @author zxl
 * @version 1.0
 * @description: kaptcha的控制层,kaptcha调用
 * @date 2022/10/30
 */
@Slf4j
@RestController
@RequestMapping("/kaptcha")
public class KaptchaController {

    @Autowired
    private Producer producer;

    @GetMapping("/kaptcha-image")
    public void getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //禁止server缓存
        response.setDateHeader("Expires", 0);
        //设置标准的http/1.1 no-cache headers
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");
        // 设置IE扩展 HTTP/1.1 no-cache headers (use addHeader)
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");
        // 设置标准 HTTP/1.0 不缓存图片
        response.setHeader("Pragma", "no-cache");
        // 返回一个 jpeg 图片，默认是text/html(输出文档的MIMI类型)
        response.setContentType("image/jpeg");
        // 为图片创建文本
        String capText = producer.createText();
        // 输出验证码
        log.info("******************当前验证码为：{}******************", capText);
        // 将验证码存于session中
        request.getSession().setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);
        // 创建带有文本的图片
        BufferedImage bi = producer.createImage(capText);
        ServletOutputStream out = response.getOutputStream();
        // 向页面输出验证码
        ImageIO.write(bi, "jpg", out);
        try {
            // 清空缓存区
            out.flush();
        } finally {
            // 关闭输出流
            out.close();
        }
    }
}

```



