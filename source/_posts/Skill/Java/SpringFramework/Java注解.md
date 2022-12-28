---
title: Java使用的注解整理
categories:
  - Skill
  - Java
tags:
  - java
  - 注解
abbrlink: 11df8ded
---

注解的相关说明都扔到这里吧。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true} -->

<!-- code_chunk_output -->

- [Spring 注解](#spring-注解)
- [一些注解说明](#一些注解说明)
  - [@SpringBootApplication](#springbootapplication)
  - [@Component 衍生注解（本质就是 @Component 注解）：](#component-衍生注解本质就是-component-注解)
  - [@MapperScan](#mapperscan)
  - [@ControllerAdvice、@RestControllerAdvice](#controlleradvicerestcontrolleradvice)
  - [@ResponseBody、@RequestBody、@Controller](#responsebodyrequestbodycontroller)
  - [@Async](#async)

<!-- /code_chunk_output -->

## Spring 注解

![Spring注解](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/Spring注解总结.png "Spring注解总结")

## 一些注解说明

### @SpringBootApplication

@SpringBootApplication 是一个组合注解，包含多个注解；

```java
// 指定注解的范围
@Target
// 指定注解什么时候有效
@Retention
// 自动配置 spring springmvc 相关环境
@SpringBootConfiguration
// 开启自动配置，自动配置核心注解，自动配置 spring 相关环境，引入第三方技术自动配置其环境，mybatis-springboot、redis-springboot 等等
@EnableAutoConfiguration
// 组件扫描，默认扫描当前包及子包，根据注解发挥注解作用。
@ComponentScan
```

### @Component 衍生注解（本质就是 @Component 注解）：

- @Repository/@Mapper DAO类型
- @Service  Service类型
- @Controller Controller 类型

是的，你没看错，上面的都是 @Component 的衍生注解，使用它们可以更加准确的表达一个类型的作用。
顺带一提：@Configuration 可以创建多个 Bean 对象，而 @Component 只能创建单个 Bean 对象。

### @MapperScan

该注解是 Mybatis 提供的，作用是扫描 Dao 层接口，交给 Spring 工厂去创建对象（和在 Dao 接口加 @Mapper 注解效果一样），这个相当于扫描全部，不需要在每个类标识。

### @ControllerAdvice、@RestControllerAdvice

顾名思义，这两个类都是对 controller 层做增强处理。这两个注解可以用来对 controller 的返回值做统一包装（下面有一个小栗子），也可以和另一个注解 @ExceptionHandler 一起作为全局的系统异常处理。

```java
// @RestControllerAdvice 
@ControllerAdvice(basePackages = "com.xxx.controller") // 只对此包中的类进行结果封装
public class ResultResponseHandler implements ResponseBodyAdvice {

    private static final Logger log = LoggerFactory.getLogger(ResultResponseHandler.class);

    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        // 过滤不需要封装的结果
        if (returnType.getParameterType().isAssignableFrom(ResultResp.class)) {
            return false;
        }
        return true;
    }

    /**
     * 此处处理结果集后响应给客户端
     */
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        if (body instanceof ResultResp) {
            return body;
        }
        return ResultResp.success(body);
    }
}
```

### @ResponseBody、@RequestBody、@Controller

这三个注解通常只在 controller 中使用：`@RestController = @ResponseBody + @Controller`.
所以对于目前常见的前后端分离的项目，一般我们直接使用 @RestController 注解即可。

另外我们已经知道，Spring 中使用了 `jaskon` 作为 json 工具，所以相关的注解底层运作也使用了 jaskon。比如 @ResponseBody、@RequestBody 等。

其中 @ResponseBody 把返回的对象转为 json 格式字符串后响应到客户端；而 @RequestBody 把客户端的 json 字符串参数转为 Java 中的复杂对象。

### @Async

Spring 提供了异步执行的代码的功能,能让我们以多线程的方式执行,但是spring 中自带的 @Async 注解执行异步时并没有使用线程池的概念, 导致如果同时执行多个任务可能出现把系统资源耗尽的情况。对此,spingBoot 做好了优化，默认使用线程池执行任务。

只需要在 SpringBoot 项目中需要异步执行的代码上加上注解 `@Async`,同时在启动类上加上 `@EnableAsync` 即可，如下：

```java
@SpringBootApplication
@EnableAsync
public class AnsyApplication {
    public static void main(String[] args) {
        SpringApplication.run(AnsyApplication.class, args);
    }
}
```

```java
 @Async
 public void test() throws InterruptedException {
     Thread.sleep(1000);
     System.out.println("子线程执行了");
 }
```

测试代码

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = AnsyApplication.class)
public class TestDemo {
    @Autowired
    AnsyService service;
    @Test
    public void testDemo() throws Exception{
        service.test();
        System.out.println("主线程执行了");
         //阻塞当前的主线程，等待异步执行的结束
       Thread.sleep(2000);
    }
}
```
