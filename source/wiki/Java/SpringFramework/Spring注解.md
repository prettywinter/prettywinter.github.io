---
layout: wiki
wiki: Java
title: 注解
order: 52
---

注解的相关说明都扔到这里吧。

<!-- more -->

## Spring 注解

![Spring注解](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/Spring注解总结.png "Spring注解总结")

### 1. @SpringBootApplication

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

### 2.@Component

- @Repository/@Mapper DAO类型
- @Service  Service类型
- @Controller Controller 类型

上面的注解的本质就是 @Component 注解，使用它们可以更加准确的表达一个类型的作用。
顺带一提：@Configuration 可以创建多个 Bean 对象，而 @Component 只能创建单个 Bean 对象。

### 2. @ControllerAdvice、@RestControllerAdvice

顾名思义，这两个类都是对 controller 层做增强处理。这两个注解可以用来对 controller 的返回值做统一包装（下面有一个小栗子），也可以和另一个注解 @ExceptionHandler 一起作为全局的系统异常处理（项目常用）。

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

### 3. @ResponseBody、@RequestBody、@Controller

这三个注解通常只在 controller 层使用：`@RestController = @ResponseBody + @Controller`.
所以对于目前常见的前后端分离的项目，一般我们直接使用 @RestController 注解即可。

另外我们已经知道，Spring 中使用了 `jaskon` 作为 json 工具，所以相关的注解底层运作也使用了 jaskon。比如 @ResponseBody、@RequestBody 等。

其中 @ResponseBody 把返回的对象转为 json 格式字符串后响应到客户端；而 @RequestBody 把客户端的 json 字符串参数转为 Java 中的复杂对象。

### 4. @Async

Spring 提供了异步执行的代码的功能,能让我们以多线程的方式执行,但是 Spring 中自带的 @Async 注解执行异步时并没有使用线程池的概念, 如果同时执行多个任务可能会把系统资源耗尽的情况。对此，SpingBoot 做好了优化，默认使用线程池执行任务。

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

## 二、Java 注解

### 1. @Nonnull

这个注解好多包中都有，包括 Spring、jetbrains 等。

标记变量、参数、返回值不能为 null，用来提示开发人员注意空指针；与之相反的是 @Nullable，这个注解表示可以为 null，并且为 null 时不会有 NPE 问题。代码中可以使用这两个注解提升代码的可读性、规范性。

需要注意的是，Java 标准库和 Spring 都有这个注解，Spring 中对原来的注解进行了功能增强，Java 新版本的注解不再位于 javax 包，而是 jakarta 包中。

## 三、其它注解

### 1. @MapperScan

该注解是 Mybatis 提供的，作用是扫描 Dao 层接口，交给 Spring 工厂去创建对象（和在 Dao 接口加 @Mapper 注解效果一样），这个相当于扫描指定的包中所有的文件交给 Spring 管理，不需要在每个类中标识。