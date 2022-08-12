---
title: Java开发错误总结
categories: skill
tags: [Java]
---

整理一些在Java开发中遇到的一些问题，不定时更新。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [一. Excel文件导出时中文被转义](#一-excel文件导出时中文被转义)
- [二、新版 Spring Security 配置](#二新版-spring-security-配置)
- [三、Swagger3.0配置](#三swagger30配置)

<!-- /code_chunk_output -->

## 一. [Excel文件导出时中文被转义](https://blog.csdn.net/qq_28869233/article/details/87979552?spm=1035.2023.3001.6557&utm_medium=distribute.pc_relevant_bbs_down.none-task-blog-2~default~OPENSEARCH~default-6.nonecase&depth_1-utm_source=distribute.pc_relevant_bbs_down.none-task-blog-2~default~OPENSEARCH~default-6.nonecase)

```java
response.setHeader("Content-disposition", "attachment;filename="+fileNameURL+";"+"filename*=utf-8''"+fileNameURL);
```

Mybatis Plus 框架在格式化日期为 “yyyy-MM” 的形式后，传输到前端的过程中，如果实体类是 Date 类型，数据库是 datetime 类型，那么会出现异常错误。MP 不能格式化成 “yyyy-MM” 的形式然后返回给前端。

这个时候可以自己写自定义转换器来完成。

## 二、新版 Spring Security 配置

新版本的 Security 配置不需要继承 {% kbd WebSecurityConfigurerAdapter %} 类，其配置写法主要是以注册 Bean 的方式为主，具体的使用可以阅读 [官方文档](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter#ldap-authentication)

## 三、Swagger3.0配置

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
// 3.0 版本开启 swagger 的注解，而且需要加入 @EnableWebMvc 注解，否则会报错：
// Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException: Cannot invoke "org.springframework.web.servlet.mvc.condition.PatternsRequestCondition.getPatterns()" because "this.condition" is null
@EnableOpenApi
public class SwaggerConfig {

    @Bean
    public Docket docket() {
        // 使用 seagger 3.0
        return new Docket(DocumentationType.OAS_30)
                .apiInfo(apiInfo())
                .enable(true)
                .groupName("xxx")
                .select()
                // 扫描指定目录的所有接口
                .apis(RequestHandlerSelectors.basePackage("com.market.example.controller"))
                // 扫描所有路径
                .paths(PathSelectors.ant("/**"))
                .build();
    }

    private ApiInfo apiInfo() {
        // 中英文混着写了，方便看清楚
        return new ApiInfo("title", "description",
                "版本", "团队服务地址",
                "联系人",
                "协议类型", "协议地址");
    }
```
