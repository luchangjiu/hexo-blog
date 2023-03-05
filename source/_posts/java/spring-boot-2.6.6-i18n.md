---
title: springboot i18n 支持国际化
date: 2022-04-25 22:23:00
tags: java
---

# 说明
springboot 使用i18n 国际化语言，可以互相切换

## 依赖

java1.8
springboot 2.6.6 
maven 4.0

## maven 配置

```xml

<parent>
    <artifactId>spring-boot-parent</artifactId>
    <groupId>org.springframework.boot</groupId>
    <version>2.6.6</version>
    <relativePath />
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>


    <!-- 这里我使用 thymeleaf 模板引擎 方便 页面展示 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <!-- 在页面引入jquery  使用i18n 国际化api -->

  <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>webjars-locator-core</artifactId>
    </dependency>

    <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>jquery</artifactId>
        <version>3.6.0</version>
    </dependency>
    <dependency>
        <groupId>org.webjars.bower</groupId>
        <artifactId>jquery-i18n-properties</artifactId>
        <version>1.2.7</version>
    </dependency>


</dependencies>


```

## 项目结构

在resources 目录下新增文件夹 static/i18n

在i18n文件夹上新增的properties 文件

**默认文件**
这个是必须的 可以不填写内容
messages.properties
**中文**

messages_zh_CN.properties

```
helloword=你好 世界

```
**英文**
messages_en_US.properties
```
helloword=hello world

```

需要将编写的文件格式设置为utf-8 以防乱码


## 配置yml
这里仅展示主要的配置,其他配置节点就不写了


```yml

spring:
  messages:
    encoding: utf-8
    basename: static/i18n/messages
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
    mode: HTML5
    encoding: UTF-8
    cache: false

```

## java 代码使用

### 配置语言拦截器切换

```java

import org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver;

import javax.servlet.http.HttpServletRequest;
import java.util.Locale;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer{

	@Bean
	public AcceptHeaderLocaleResolver localeResolver(WebMvcProperties mvcProperties) {
		AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver() {
			@Override
			public Locale resolveLocale(HttpServletRequest request) {
                // 配置 lang 切换语言时请求设置
				String locale = request.getParameter("lang");
				return locale != null
						? org.springframework.util.StringUtils.parseLocaleString(locale)
						: super.resolveLocale(request);
			}
		};
		localeResolver.setDefaultLocale(Locale.CHINA);
		return localeResolver;
	}

}


```

### java 中使用

配置工具类可以方便的使用
配置完成后可以使用  <code> LocalUtil.get("helloworld")</code> 获取到国际化文本内容

```java

import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContextHolder;
import org.springframework.stereotype.Component;

@Component
public class LocalUtil {

    private static MessageSource messageSource;

    public LocalUtil(MessageSource messageSource){
        LocalUtil.messageSource = messageSource;
    }

    public static String get(String key) {
        try {
            return LocalUtil.messageSource.getMessage(key, null, LocaleContextHolder.getLocale());
        }catch (Exception ex) {
            return key;
        }
    }
}


```


### thymeleaf 页面中使用 

Controller 配置index 跳转 页面代码

```java

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping
public class HomeController {

    @RequestMapping("/get-local")
    @ResponseBody
    public String getLocal() {
        return LocalUtil.get("helloworld");
    }

    @RequestMapping
    public String index() {
        return "index";
    }
}


```
配置 index.html 页面代码


```html

<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title th:text="#{helloword}"></title>
</head>
<body>
<h1 th:text="#{helloword}"></h1>
</body>
</html>

```

### 页面jquery 使用i18n
配置index.html 页面代码

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title th:text="#{helloworld}"></title>
    <!-- 引入jquery i18n 相关js -->
    <script th:src="@{/webjars/jquery/jquery.min.js}"></script>
    <script th:src="@{/webjars/jquery-i18n-properties/jquery.i18n.properties.min.js}"></script>

    <script th:inline="javascript">

        // 引入基础路径
        var ROOT = [[${#servletContext.contextPath}]] ;

        // 获取语言和国家
        var LANG_COUNTRY = [[${#locale.language+'_'+#locale.country}]] ;

        // 初始化i18n 插件
        $.i18n.properties({
            path: ROOT + '/i18n/',  //基础访问路径
            name: 'messages', // i18n 文件开头
            language: LANG_COUNTRY,
            encoding: 'utf-8',
            mode: 'map'
        });

        // 初始化 获取i18n 获取的函数
        function getI18N(key) {
            try{
                return $.i18n.prop(key);
            } catch(e) {
                console.log(e);
                return key;
            }
        }

        //输出i18n 文本

        console.log(getI18N('helloworld'));

    </script>
</head>
<body>
<h1 th:text="#{helloworld}"></h1>

</body>
</html>


```



### 启动并测试

启动后 录入地址

java 代码获取测试
访问中文
http://localhost:8080/get-local?lang=zh_CN 
访问英文
http://localhost:8080/get-local?lang=en_US

网页获取测试 

访问中文页面
http://localhost:8080/?lang=zh_CN 
访问英文页面
http://localhost:8080/?lang=en_US


