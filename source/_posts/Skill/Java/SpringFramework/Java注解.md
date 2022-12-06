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

<!-- /code_chunk_output -->

## Spring 注解

![Spring注解](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/Spring注解总结.png "Spring注解总结")

## 一些注解说明

@ControllerAdvice 是 Spring3.2 提供的新注解，是一个 Controller 的增强器，可对 controller 中 @RequestMapping 注解的方法加一些逻辑处理。

@ControllerAdvice 注解常与 @ExceptionHandler 注解联合用于自定义全局异常处理；也常被用来对 controller 层响应给用户的数据做统一处理后返回客户端。
全局异常处理就不说了，说一下后者：

```java
// @RestControllerAdvice 
@ControllerAdvice(basePackages = "com.controller") // 只对此包中的类进行结果封装，集成 swagger 的时候有个小坑
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

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        if (body instanceof ResultResp) {
            return body;
        }
        if (body instanceof String) {
            response.getHeaders().setContentType(MediaType.APPLICATION_JSON_UTF8);
            try {
                return new ObjectMapper().writeValueAsString(body);
            } catch (JsonProcessingException e) {
                throw new RuntimeException(e);
            }
        }
        return ResultResp.success(body);
    }
}
```
也可以实现 `ResponseBodyAdvice` 接口，对使用了 `@Controller` 注解的 controller 层响应到客户端的数据做统一处理后返回给客户端。

> @RestControllerAdvice 与 @ControllerAdvice 类似。

@ExceptionHandler 用来指明异常类
@ResponseBody 底层集成了 jaskson，所以可以把控制器方法返回的对象等信息转换为 json 字符串并响应请求。
@RequestBody 将请求中 json 格式字符串自动转为 Java 中对象(复杂对象)

@Component 衍生注解（本质就是 @Component 注解）：

- @Repository/@Mapper DAO类型
- @Service  Service类型
- @Controller Controller 类型
提供衍生注解更加准确的表达一个类型的作用。

@Async：Spring 提供了 异步执行的代码的功能,能让我们以多线程的方式执行,但是spring 中自带的 @Async 注解执行异步时并没有使用线程池的概念, 导致如果同时执行多个任务可能出现把系统资源耗尽的情况。对此,spingBoot 一做好了优化，默认使用线程池执行任务。在springboot 项目中需要异步执行的代码上加上注解 @Async,同时在启动类上加上@EnableAsync即可，如下：

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
