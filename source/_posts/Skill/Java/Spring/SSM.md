---
title: Java框架之SSM
categories: skill
tags: [Java, SSM]
---

SSM框架使用整理。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [一、Sping、SpringMVC](#一spingspringmvc)
  - [1.1 事务管理](#11-事务管理)
  - [1.2 Spring AOP](#12-spring-aop)
  - [1.3 文件的上传下载](#13-文件的上传下载)
  - [1.4 拦截器(一般用于用户认证，其它不常用)](#14-拦截器一般用于用户认证其它不常用)
  - [1.5 MVC跳转页面](#15-mvc跳转页面)
  - [1.6 Spring MVC异常处理](#16-spring-mvc异常处理)
- [二、Mybatis](#二mybatis)
  - [2.1 mapper文件和mapper接口同包](#21-mapper文件和mapper接口同包)
  - [2.2 Mybatis框架与数据库的时间关系](#22-mybatis框架与数据库的时间关系)
- [三、MybatisPlus](#三mybatisplus)
  - [3.1 MP分页插件Page](#31-mp分页插件page)
  - [3.2 MP的LambdaQueryWrapper](#32-mp的lambdaquerywrapper)
  - [3.3 MP程序中获取新插入数据的Id](#33-mp程序中获取新插入数据的id)
  - [3.4 MP中的saveBatch()插入的性能问题](#34-mp中的savebatch插入的性能问题)
  - [3.5 MP中不能插入或更新null值](#35-mp中不能插入或更新null值)
- [logback](#logback)

<!-- /code_chunk_output -->

## 一、Sping、SpringMVC

{% link https://spring.io Spring官网 %}

### 1.1 事务管理

Spring 管理事物的方式：

- 编程式事务，在代码中硬编码。(不推荐使用)
- 声明式事务，在配置文件中配置(推荐使用)
  1. 基于XML的声明式事务
  2. 基于注解的声明式事务

### 1.2 Spring AOP

编写一个 Aop 配置类：
导入 aop 依赖；在 Aop 配置类上加上 @Aspect、@Configuration 注解；编写切面增强方法。

1. 切入点表达式
    - 方法级别的切入点表达式：`execution(* com.XXX.*.*(..))`，第一个 * 号代表可以返回任意类型值。
    - 类级别的切入点表达式：`within(com.XXX.*)`
    - 自定义拦截器：`@annotation(com.XXX)`
2. 增强方式
    - 前置增强、后置增强、环绕增强
    - 前置和后置都没有返回值，方法参数都是JointPoint
    - 环绕增强中，需要调用proceed()才能继续处理业务逻辑(类似拦截器)，该方法返回值为业务的返回值，因此环绕增强的返回类型设置为 Object 比较推荐。
    - 环绕增强的方法参数是ProceedingJointPoint

### 1.3 文件的上传下载

要求表单提交必须是 {% kbd post %} 方式，encType 必须为 `multiPart/form-data`; `multipart file` 默认的上传文件大小(max-request-size)最大为10M。

文件的下载在响应之前要设置以附件的形式，否则点击下载时会在浏览器打开。

```java
response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));

// 将文件传到指定的目录
transferTo();
// 文件io可以使用 spring 框架自带的 copy 方法
FileCopyUtils.copy(is, os);
```

### 1.4 拦截器(一般用于用户认证，其它不常用)

编写拦截器：

1. 实现 HandlerInterceptor 接口；
2. 重写 preHandle、postHandle、afterCompletion 方法，其中 preHandle 方法中返回 true 代表放行，返回 false 代表中断。
3. 配置拦截器：实现 WebMvcConfigurer 接口；重写 addInterceptors 方法，添加编写的拦截器。另外，在实现的这个接口中，还有另外一个方法可以解决跨域问题，是一个全局跨域配置。

```java WebMvcConfig.class
// CORS 跨域访问配置
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
```

preHandle 返回值为 true 时，执行控制器中的方法，当控制器方法执行完成后会返回拦截器中执行拦截器中的 postHandle 方法，postHandle 执行完成之后响应请求。在响应请求完成后会执行 afterCompletion 方法，该方法无论执行成功或者失败都会执行。

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

> 拦截器只能拦截 controller 相关请求，不能拦截 jsp 静态资源文件；拦截器可以中断请求轨迹；请求之前如果该请求配置了拦截器，请求会先经过拦截器，放行之后执行请求的 controller，controller 执行完成后会回到拦截器继续执行拦截器代码。
> 如果配置了多个拦截器，默认执行的顺序和栈结构是一样的；但是也可以通过 `order()` 方法修改，里面填 int 类型的数字，数字大的优先执行。

### 1.5 MVC跳转页面

1. 跳转到页面
   - 转发方式：直接 return 逻辑视图名
   - 重定向方式：重定向(redirect)不会经过视图解析器 redirect:/视图全名
2. 跳转到方法
   - 转发跳转相同 controller 的不同方法使用 springMVC 提供的 `forward:` 关键字
   - 重定向使用 `redirect:/controller的RequestMapping/方法的RequestMapping` 
不同 controller 的方法跳转，和相同的一样。

重定向到登录页面：`response.sendRedirect(request.getContextPath() + "/login.jsp")`

### 1.6 Spring MVC异常处理

实现 HandlerExceptionResolver 的接口，重写里面的 resolveException 方法。

## 二、Mybatis

{% link https://mybatis.org/mybatis-3/zh/index.html Mybatis官网 %}

### 2.1 mapper文件和mapper接口同包

除了需要在配置文件中指定 xml 文件的位置，在项目的 {% kbd pom %} 文件中也要加入以下内容：

```xml pom.xml
<build>
  ...
  <resources>
      <!-- mapper接口和文件在同一个包路径下必须配置 -->
      <resource>
          <directory>src/main/java</directory>
          <includes>
              <include>**/*.xml</include>
          </includes>
          <filtering>false</filtering>
      </resource>
  </resources>
  ...
</build>
```

### 2.2 Mybatis框架与数据库的时间关系

|jdbcType|database type|
|--|--|
|DATE|date|
|TIMESTAMP|datetime|

## 三、MybatisPlus

{% link https://baomidou.com/guide/ Mybatis Plus官网 %}

### 3.1 MP分页插件Page

Mybatis Plus自带分页插件接口：IPage，它的下面有一个实现类 Page。
在使用时，可以使用Page创建一个对象实例，设置分页的一些参数；IPage对象用来存放自定义的分页规则和查询条件参数，比如以下简单例子：

{% grid color:green %}
{% tabs %}
<!-- tab UserMapper.java -->
{% codeblock lang:java %}
//mapper中也可以使用 Page ，效果一样的
IPage<User> getUserList(Page<User> page,
                        @param("id") Integer id,
                        @Param("startTime") String startTime,
                        @Param("endTime") String endTime);
{% endcodeblock %}

<!-- tab UserService.java -->
{% codeblock lang:java %}
IPage<User> selectUserList(Integer id, 
                           String startTime,
                           String endTime);
{% endcodeblock %}

<!-- tab UserServiceImpl.java -->
{% codeblock lang:java %}
@Autowired
private UserMapper userMapper;
@Override
public IPage<User> selectUserList(Integer id, String startTime, String endTime) {
    Page<User> page = new Page<>();
    // page.setXXX 设置自定义分页属性

    IPage<User> ipage = userMapper.getUserList(page, id, startTime, endTime);
    return ipage;
}
{% endcodeblock %}

<!-- tab UserController.java -->
{% codeblock lang:java %}
@Autowired
private UserService userService;

@GetMapping("/userList")
public IPage<User> selectUserList(Integer id, String startTime, String endTime) {
    return userService.selectUserList(id, startTime, endTime);
}
{% endcodeblock %}
{% endtabs %}
{% endgrid %}

上面的Page分页参数有两种设置方式如下：

![Page参数](https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/Page参数.png "Page参数")

![Page可设置参数](https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/Page可设置参数.png "Page可设置参数")

### 3.2 MP的LambdaQueryWrapper

Mybatis Plus自带的 LambdaQueryWrapper 可以用来查询对象信息，如下代码：根据openId 查找对应的 customer。在不涉及多表查询时，用此方法比较 nice，个人感觉很好玩。

```java
// 注意，只有指定了 LambdaQueryWrapper<?> 中的 ? 类型后才可以使用函数式接口
LambdaQueryWrapper<Customer> query = Wrappers.lambdaQuery();
query.eq(Customer::getOpenId, openId);
return customerMapper.selectOne(query);
```

### 3.3 MP程序中获取新插入数据的Id

Mybatis 的 mapper.xml 文件中，在插入数据的标签中可以添加 `useGeneratedKeys="true" keyProperty="id"` 这两个属性，插入之后会返会新插入数据的主键 id，之后我们可以使用的这个主键与其他表进行关联。不过需要注意的是，这个 `id` 必须是自增的。

### 3.4 MP中的saveBatch()插入的性能问题

Mybatis Plus 中有批量插入的方法：saveBatch()，它可以保存一个集合。使用这个方法，不用写 sql 语句，但是性能比较差，网上看文章说在数据库（指MySQL）连接（url）后面加入一个配置：`rewriteBatchedStatements=true`，这样可以解决大批量插入时的性能问题，当我第一次看到这个方法查资料时看到一篇文章写的特别好，但是后来浪子找不到了，等到以后找到再补上链接。

### 3.5 MP中不能插入或更新null值

当使用 Mybatis Plus 时，更新数据为 null 值时，会发现，即使数据库的字段可以设置为 null，但是更新或插入时还是数据更新失败。这个就涉及到字段验证策略了。对于这个问题，官网 [常见问题](https://www.mybatis-plus.com/guide/faq.html) 中也有解决方法，[点击查看官网说明](https://www.mybatis-plus.com/guide/faq.html#%E6%8F%92%E5%85%A5%E6%88%96%E6%9B%B4%E6%96%B0%E7%9A%84%E5%AD%97%E6%AE%B5%E6%9C%89-%E7%A9%BA%E5%AD%97%E7%AC%A6%E4%B8%B2-%E6%88%96%E8%80%85-null)

1. 局部配置（在更改的字段加上特定的注解）

    ```java
    // 修改策略，最好是再后面加上该字段在数据库中的类型
    @TableField(updateStrategy = FieldStrategy.IGNORED, jdbcType = JdbcType.INTEGER)
    ```

2. 可以开全局配置，在 `xxx.properties` 或者 `xxx.yml` 文件中注入配置 GlobalConfiguration 属性 fieldStrategy。

## logback

使用 logback 框架的时候，如果需要动态的去指定日志存放目录，可以使用以下方法：

```java config/LogHomeConfig
// 继承 PropertyDefinerBase 类，重写其中的 getPropertyValue 方法
public class LogHomeConfig extends PropertyDefinerBase {
    @Override
    public String getPropertyValue() {
        // 获取用户名
        String username = System.getProperty("user.name");
        // 获取操作系统
        String os = System.getProperty("os.name");
        // 路径常量可以放到常量类维护，这里说明问题即可
        return os.toLowerCase().contains("window") ? "path" : "/home/" + username + "/logs";
    }
}
```

```xml logback.xml
<configuration>
    <!-- 日志存放路径，这里使用了 define 标签，class 定义为重写了 PropertyDefinerBase 类方法的配置类 -->
    <!-- <property name="log.path" define="" value="/home/clf/market/logs"> -->
    <define name="log.path" class="com.example.common.logs.LogHomeConfig"/>
    ...
</configuration>
```

这样，就可以根据不同的操作系统生成不同的路径。