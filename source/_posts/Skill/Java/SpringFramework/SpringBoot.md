---
title: Spring Boot
categories:
  - Skill
  - Java
tags:
  - java
  - springboot
abbrlink: a541262a
---

SpringBoot 集成各种第三方库（jar包）及常见问题解决方式。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [一、Spring Boot 项目构建](#一spring-boot-项目构建)
- [二、常用配置](#二常用配置)
  - [1. Swagger（不建议，新项目推荐 Spring Doc）](#1-swagger不建议新项目推荐-spring-doc)
    - [常见错误: 集成 swagger 的同时使用 `ResponseBodyAdvice` 对 controller 的结果做统一封装](#常见错误-集成-swagger-的同时使用-responsebodyadvice-对-controller-的结果做统一封装)
  - [2. 跨域问题](#2-跨域问题)
  - [3. Excel文件导出时中文被转义](#3-excel文件导出时中文被转义)
  - [4. 切面、拦截器、过滤器](#4-切面拦截器过滤器)
    - [4.1 AOP切面](#41-aop切面)
    - [4.2 拦截器](#42-拦截器)
  - [5. 文件的上传下载](#5-文件的上传下载)
  - [6. locback](#6-locback)
  - [Maven打包(打包之前先清理(clean)再打包(package))](#maven打包打包之前先清理clean再打包package)
- [三、集成MP（Nybatis Plus）](#三集成mpnybatis-plus)
  - [1. 打印SQL语句](#1-打印sql语句)
  - [2. MP插入或更新null值](#2-mp插入或更新null值)
  - [3. 使用Wrappers.xxxquery()查询某个字段](#3-使用wrappersxxxquery查询某个字段)

<!-- /code_chunk_output -->

## 一、Spring Boot 项目构建

可以使用阿里云的构建模板：`https://start.aliyun.com/`。

## 二、常用配置

### 1. Swagger（不建议，新项目推荐 Spring Doc）

个人已经不推荐使用 swagger，当然如果你喜欢使用 knife4j 就可以忽略这部分，不需要了解太多。

对于使用 SpringBoot 2.7.x 以上版本，或者遵循 Open Api3.0 的推荐使用 [Spring Doc](https://springdoc.org/)，尤其是准备使用 SpringBoot3.0 的。

而且使用的过程中，你会发现 Spring Doc 的标签属性比 swagger 的清晰（因为doc标签属性少），虽少但是齐全够用。

```xml pom.xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

```java
@Component
@EnableWebMvc
@EnableOpenApi
public class SwaggerConfig {

    @Bean
    public Docket docket() {
        // 使用 seagger 3.0
        return new Docket(DocumentationType.OAS_30)
                .enable(true)
                .groupName("xxx")
                .select()
                // 扫描指定目录的所有接口
                .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
                // 扫描所有路径
                .paths(PathSelectors.ant("/**"))
                .build()
                // 中英文混着写了，方便看清楚
                .apiInfo(new ApiInfo("title", "description",
                "version", "团队服务地址",
                "联系人",
                "协议类型", "协议地址"));
    }
}
```

#### 常见错误: 集成 swagger 的同时使用 `ResponseBodyAdvice` 对 controller 的结果做统一封装

如果在项目中实现 `ResponseBodyAdvice` 接口统一封装 controller 返回的接口，使用 swagger 访问出现 `Unable to infer base url. This is common when using dynamic servlet registration or when the API is behind an API Gateway` 的问题。

那么需要在实现 `ResponseBodyAdvice` 的类注解上额外加入 `@RestControllerAdvice(basePackages = "xxx.xx.controller")` 限定对返回结果封装范围。这样的话就不会拦截 swagger 相关资源的地址。

也可以下面这样修改（浪子没有测试，具体见：https://juejin.cn/post/6921700441038258189）：

```java
@RestControllerAdvice
// @RestControllerAdvice(basePackages = "com.example.controller") // 只对此包中的类进行结果封装
public class ResultResponseHandler implements ResponseBodyAdvice {

    @Override
    public boolean supports(MethodParameter returnType, Class converterT  ype) {
        // 过滤不需要封装的结果
        if (returnType.getParameterType().isAssignableFrom(ResultResp.class)) {
            return false;
        }
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
         if (body instanceof Json || body instanceof UiConfiguration || 
         (body instanceof ArrayList && ((ArrayList) body).get(0) instanceof SwaggerResource)) {
            return body;
         }
        return ResultResp.success(body);
    }
```

### 2. 跨域问题

{% ablock %}
{% tabs %}
<!-- tab 注解方式 -->
{% codeblock 注解方式解决跨域 lang:java %}
// 注解用在 Controller 类中，该类所有方法允许其它域中进行访问
@CrossOrigin
public class xxxController {

}
{% endcodeblock %}
<!-- tab 全局配置 -->
{% codeblock 全局配置解决跨域 lang:java %}
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins("*");
        config.setAllowedHeaders("*");
        config.setAllowedMethods("*");
        // 处理所有请求的跨域配置
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
{% endcodeblock %}
<!-- tab MVC全局配置 -->
{% codeblock MVC配置解决跨域 lang:java %}
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    /**
     * 跨域访问（CORS）
     *
     * @param registry
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowCredentials(false)
                // 所有头
                .allowedHeaders("/**")
                // 所有源
                .allowedOrigins("/**")
                // 所有方法
                .allowedMethods("/**")
                .maxAge(5000);
    }
}
{% endcodeblock %}
{% endtabs %}
{% endablock %}

### 3. [Excel文件导出时中文被转义](https://blog.csdn.net/qq_28869233/article/details/87979552?spm=1035.2023.3001.6557&utm_medium=distribute.pc_relevant_bbs_down.none-task-blog-2~default~OPENSEARCH~default-6.nonecase&depth_1-utm_source=distribute.pc_relevant_bbs_down.none-task-blog-2~default~OPENSEARCH~default-6.nonecase)

```java
String fileName = URLEncode.encode(name, "UTF-8");
response.setHeader("Content-disposition", "attachment;filename="+fileName+";"+"filename*=utf-8''" + fileName);
```

### 4. 切面、拦截器、过滤器

#### 4.1 AOP切面

编写一个 Aop 配置类：
导入 aop 依赖；在 Aop 配置类上加上 @Aspect、@Configuration 注解；编写切面增强方法。

1. 切入点表达式
    - 方法级别的切入点表达式：`execution(* com.XXX.*.*(..))`，第一个 * 号代表可以返回任意类型值。
    - 类级别的切入点表达式：`within(com.XXX.*)`
    - 自定义注解表达式：`@annotation(com.XXX)`
2. 增强方式
    - 前置增强 (@Befor)、后置增强(@After)、环绕增强 (@Arround)、后置返回增强 (@AfterReturning)、异常增强 (@AfterThrowing)
    - 前置和后置都没有返回值，方法参数都是 JointPoint
    - 环绕增强中，需要调用 `proceed()` 才能继续处理业务逻辑(类似拦截器)，该方法返回值为业务的返回值，因此环绕增强的返回类型设置为 Object 比较推荐。
    - 环绕增强的方法参数是 ProceedingJointPoint

#### 4.2 拦截器

1. 实现 HandlerInterceptor 接口；
2. 重写 preHandle、postHandle、afterCompletion 方法，其中 preHandle 方法中返回 true 代表放行，返回 false 代表中断。

preHandle 返回值为 true 时，执行控制器中的方法，当控制器方法执行完成后会返回拦截器中执行拦截器中的 postHandle 方法，postHandle 执行完成之后响应请求。在响应请求完成后会执行 afterCompletion 方法，该方法无论执行 **成功** 或者 **失败** 都会执行。

    ```java CustomerInterceptor.class
    public class CustomerInterceptor implements HandlerInterceptor {
        /**
        * 先执行
        */
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            return HandlerInterceptor.super.preHandle(request, response, handler);
        }

        /**
        * 上面结果为 true 时执行
        */
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
        }

        /**
        * 最后都会执行
        */
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
        }

    }
    ```

3. 配置拦截器：实现 WebMvcConfigurer 接口；重写 addInterceptors 方法，添加编写的拦截器。

   ```java
   @Configuration
    public class WebMvcConfig implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(customerInterceptor());
        }
    }
   ```

> 拦截器只能拦截 controller 相关请求，不能拦截 jsp 静态资源文件；拦截器可以中断请求轨迹；请求之前如果该请求配置了拦截器，请求会先经过拦截器，放行之后执行请求的 controller，controller 执行完成后会回到拦截器继续执行拦截器代码。
> 如果配置了多个拦截器，默认执行的顺序和栈结构是一样的；但是也可以通过 `order()` 方法修改，里面填 int 类型的数字，数字大的优先执行。

### 5. 文件的上传下载

要求表单提交必须是 {% kbd post %} 方式，encType 必须为 `multiPart/form-data`; `multipart file` 默认的上传文件大小(max-request-size)最大为10M。

```java
response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));

// 将文件传到指定的目录
transferTo(targetPath);
// 文件io可以使用 spring 框架自带工具类的 copy 方法
FileCopyUtils.copy(is, os);
```

> 文件的下载在响应之前要设置以 **附件（attachment）** 的形式，否则点击下载时会在浏览器打开。

### 6. locback

使用 logback 框架的时候，如果需要动态的去指定日志存放目录，可以使用以下方法：

```java config/LogHomeConfig.java
// 继承 PropertyDefinerBase 类，重写其中的 getPropertyValue 方法
public class LogHomeConfig extends PropertyDefinerBase {
    @Override
    public String getPropertyValue() {
        // 获取用户名
        String username = System.getProperty("user.name");
        // 获取操作系统
        String os = System.getProperty("os.name");
        // 路径常量可以放到常量类维护，这里说明问题即可
        return os.toLowerCase().contains("window") ? "./logs" : "/home/" + username + "/logs";
    }
}
```

```xml logback.xml
<configuration>
    <!-- 日志存放路径，这里使用了 define 标签，class 定义为重写了 PropertyDefinerBase 类方法的配置类 -->
    <!-- <property name="log.path" define="" value="/home/user/market/logs"> -->
    <define name="log.path" class="com.example.common.logs.LogHomeConfig"/>
    ...
</configuration>
```

这样，就可以根据不同的操作系统生成不同的路径。

### Maven打包(打包之前先清理(clean)再打包(package))

jar 包比较简单，SpringBoot 默认就是 jar 包，这里不在介绍，只说 war 包方式。

1. 修改打包方式，去除内嵌的 tomcat 依赖

    ```xml pom.xml
    <packaging>war</packaging>

    ...

    <!-- 去除内嵌的 tomcat 依赖 -->
    <scope>provided</scope>
    ```

2. 如果整合的是 jsp，那么使用打包的插件时，打包插件必须是 `1.4.2` 的版本，另外，在pom文件中加入以下内容，指定 jsp 文件的打包位置：

    ```xml pom.xml
        <resources>
            <!-- 打包时将 jsp 文件拷贝到 META-INF 目录下 -->
            <resource>
                <!-- 指定 resources 插件处理那个目录下的资源文件 -->
                <directory>src/main/webapp</directory>
                <!-- 指定必须要放在此目录下才能被访问到 -->
                <targetPath>META-INF/resources</targetPath>
                <includes>
                    <include>**/**</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/**</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
            <directory>src/main/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </resource>
        </resources>
    ```

3. 在插件中指定入口类：

    ```xml
    <configuration>
        <fork>true</fork>
        <jvmArgument>-Dfile.encoding=UTF-8</jvmArgument>
        <mainClass>com.XXX.Application.class</mainClass>
    </configuration>
    ```

4. 在启动类中，继承 `SpringBootServletInitiallizer` 类，重写 `configure()` 方法进行配置:

    ```java
    public class xxxApplication extends SpringBootServletInitializer {
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources(xxxApplication.class);
        }
    }
    ```

> **注意：** 打成的 war 包使用外部 Tomcat 部署时，项目配置文件中配置的端口和项目名称会失效。
> 后台方式启动 jar 包：`nohup java -jar jar包名称 &`





## 三、集成MP（Nybatis Plus）

### 1. 打印SQL语句

```yml
# 以下配置二选一
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

logging:
  level:
    com.jz.mapper: debug
```

### 2. MP插入或更新null值

当使用 Mybatis Plus 时，更新数据为 null 值时，即使数据库的字段设置为可以为 null，但是更新或插入时还是数据更新失败。这个就涉及到字段验证策略了。MP 官网也给出了 [解决办法](https://baomidou.com/pages/f84a74/#%E6%8F%92%E5%85%A5%E6%88%96%E6%9B%B4%E6%96%B0%E7%9A%84%E5%AD%97%E6%AE%B5%E6%9C%89-%E7%A9%BA%E5%AD%97%E7%AC%A6%E4%B8%B2-%E6%88%96%E8%80%85-null)

### 3. 使用Wrappers.xxxquery()查询某个字段

有时候，我们只需要查询一张表中的一个或几个字段（同一张表），又不想写 SQL 的情况下，可以使用 `queryWapper.select()` 方式，举个栗子：

```java
public List<Integer> listManageLevel() {
    LambdaQueryWrapper<User> query = Wrappers.lambdaQuery();
    // 使用 select 方式只查询 age 字段
    query.select(User::getAge).groupBy(User::getAge);
    // 如果使用 selectList() 返回的是对象集合
    List<User> users = userMapper.selectList(query);
    // 也可以使用下面的方式，返回一个 Object 集合，注意修改返回值
    // List<Object> ages = sasacMapper.selectObjs(query);

    List<Integer> ages = users.stream()
                            .map(User::getAge)
                            .collect(Collectors.toList());
    return ages;
}
```

> 注意，上面的查询返回的是一个对象集合，还需要使用 map 映射取出字段。当然，你也可以使用注释掉的代码，返回一个 Object 集合，对于单字段来说是比较好的选择。
