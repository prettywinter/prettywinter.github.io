---
layout: wiki
wiki: Java
title: SpringBoot
order: 54
---

SpringBoot 用起来的真的简单方便，不需要太多繁琐的配置，实现开箱即用，对于 Java 开发人员来说是莫大的福音。这里整理一下相关的内容。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=false} -->

<!-- code_chunk_output -->

- [一、SpringBoot 项目构建](#一springboot-项目构建)
- [二、SpringBoot整合第三方库](#二springboot整合第三方库)
  - [1. springdoc 或者 smart-doc](#1-springdoc-或者-smart-doc)
  - [2. MP](#2-mp)
  - [3. locback 日志自定义存放文件夹](#3-locback-日志自定义存放文件夹)
- [三、SpringBoot 配置文件](#三springboot-配置文件)

<!-- /code_chunk_output -->

## 一、SpringBoot 项目构建

除了 [Spring](https://start.spring.io) 官网的模板，也可以使用阿里云的构建模板：`https://start.aliyun.com/`，该模板除了速度占优势，也有更多的阿里系常用依赖。

## 二、SpringBoot整合第三方库

### 1. [springdoc](https://springdoc.org) 或者 [smart-doc](https://gitee.com/smart-doc-team/smart-doc)

springdoc 比 swagger 的注解属性更加清晰明了。

smart-doc 只需要写好标准的 JavaDoc，并按照 [官网说明](https://smart-doc-group.github.io/#/zh-cn/start/bestPractice) 编写即可，不需要其它的注解等。

### 2. [MP](https://www.baomidou.com/pages/24112f/)

```yml
# 打印SQL语句 以下配置二选一
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

logging:
  level:
    com.xxx.mapper: debug
```

### 3. locback 日志自定义存放文件夹

logback 是 SpringBoot 默认使用的日志框架。在使用 logback 框架的时候，如果需要动态的去指定日志存放目录，可以重写 Spring 预留的父类中 `PropertyDefinerBase` 的方法，然后在 `logback.xml` 中写入该配置类的完全限定名称即可。

```java config/LogHomeConfig.java
// 继承 PropertyDefinerBase 类，重写其中的 getPropertyValue 方法
public class LogHomeConfig extends PropertyDefinerBase {
    @Override
    public String getPropertyValue() {
        // 获取用户名
        String username = System.getProperty("user.name");
        // 获取操作系统
        String os = System.getProperty("os.name");
        // 路径常量可以放到常量类维护，这里说明问题即可（目录放在jar包的同级目录）
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

这样，就可以根据不同的操作系统存储在不同的路径。

> 关于 slf4 和 Java 中的其它日志框架的关系，引用 [Rust 语言圣经](https://course.rs/logs/log.html) 里的一段话，浪子觉得写的非常好：
>
> slf4j 是 Java 的日志门面库，日志门面不是说排场很大的意思，而是指相应的日志 API 已成为事实上的标准，会被其它日志框架所使用。通过这种统一的门面，开发者就可以不必再拘泥于日志框架的选择，未来大不了再换一个日志框架就是.

## 三、SpringBoot 配置文件

```yml
server:
  port: port

spring:
  application:
    # 应用名称
    name: application-name
  # 数据源
  datasource:
    # hikari 连接池
    type: com.zaxxer.hikari.HikariDataSource
    # PostgreSQL 连接，默认端口：5432
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/postgres?serverTimezone=Asia/Shanghai
    # sqlserver 默认端口号为：1433
    # url: jdbc:microsoft:sqlserver://localhost:1433;DatabaseName=dbname
    # driver-class-name: com.microsoft.jdbc.sqlserver.SQLServerDriver
    # mysql 默认端口号为：3306
    # url: jdbc:mysql://localhost:3306/test?username=root&password=&useUnicode=true&characterEncoding=utf-8&mysqlEncoding=utf-8&serverTimezone=Asia/Shanghai
    # driver-class-name: com.mysql.jdbc.Driver
    # oracle 默认端口号为：1521
    # url: jdbc:oracle:thin:@localhost:1521:orcl
    # driver-class-name: oracle.jdbc.driver.OracleDriver
    hikari:
      username: username
      password: password
      # 最大连接池数量
      maximum-pool-size: 20
      # 配置一个连接在池中最大生存的时间，单位是毫秒
      max-lifetime: 60000
      # 存活时间
      keepalive-time: 6000
  jpa:
    show-sql: true
    hibernate:
      generate_statistics: true
      naming:
        # 相当于 org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy（3.0删除）
        # 驼峰与下划线映射
        physical-strategy: org.hibernate.boot.model.naming.CamelCaseToUnderscoresNamingStrategy
    # jpa 属性配置
    properties:
      hibernate:
        # 格式化控制台打印 SQL
        format_sql: true
#        jdbc:
#          batch_size: 50
#        # 检测批处理是否打开 它可能不生效
#        generate_statistics: true

# Spring 内置日志级别
logging:
  level:
    # 设置某个包的日志打印级别为 debug
    com.leaf.system.repository: debug
```