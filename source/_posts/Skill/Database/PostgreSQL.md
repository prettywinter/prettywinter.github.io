---
title: PostgreSQL
categories:
  - Skill
  - DB
tags:
  - database
  - postgresql
cover: 'database,postgresql'
abbrlink: cb97b83b
---

PostgreSQL：最先进的开源数据库。

<!-- more -->

## 一、为什么使用PostgreSQL

PostgreSQL 是一个功能强大的开源对象关系数据库系统，它使用并扩展了 SQL 语言，并结合了许多可安全存储和扩展最复杂数据工作负载的特性。PostgreSQL 的起源可以追溯到 1986 年，作为POSTGRES项目的一部分，并在核心平台上进行了 30 多年的积极开发。

PostgreSQL 因其久经考验的架构、可靠性、数据完整性、强大的功能集、可扩展性以及软件背后的开源社区致力于始终如一地提供高性能和创新解决方案而赢得了良好的声誉。PostgreSQL 可在所有主要操作系统一直符合 ACID，并具有强大的附加组件，例如流行的PostGIS地理空间数据库扩展器。毫不奇怪，PostgreSQL 已成为许多人和组织选择的开源关系数据库。

PostgreSQL 具有许多功能，旨在帮助开发人员构建应用程序、帮助管理员保护数据完整性和构建容错环境，并帮助您管理数据，无论数据集大小。除了免费和开源，PostgreSQL 还具有高度可扩展性。例如，您可以定义自己的数据类型、构建自定义函数，甚至使用不同的编程语言而无需重新编译数据库！

PostgreSQL 试图符合SQL 标准，这样的一致性不会与传统特性相矛盾或可能导致糟糕的架构决策。支持 SQL 标准所需的许多功能，但有时语法或功能略有不同。随着时间的推移，可以预期进一步朝着一致性迈进。从 2021 年 9 月发布的第 14 版开始，PostgreSQL 至少符合 SQL:2016 Core 一致性的 179 个强制性特性中的 170 个。在撰写本文时，没有任何关系数据库完全符合此标准。

> 以上内容摘自官网：https://www.postgresql.org/about/

## 二、基础操作

安装完成后切换到 postgres 用户，启动服务，输入 `psql` 进入 bash 环境，或者使用 `psql -d databasename` 进入指定的数据库。或者直接输入 `sudo -u postgres psql` 也是可以进入 bash 环境的。

### 用户bash命令

```bash
# 清空指定用户的密码
$ sudo passwd -d user
# 设置指定用户的密码，根据系统提示进行输入即可
$ sudo -u user passwd
```

### PostgreSQL客户端命令

```bash
# 连接本地数据库
psql -U <username> -W dbname
# 连接到远程数据库
psql -h <host> -U <username> -W dbname

# 修改密码
ALTER USER postgres WITH PASSWORD 'postgres';
# 显示所有数据库
\l
# 显示当前数据库的所有表
\d
# 显示数据库的表空间
\db+
# 进入指定的数据库
\c dbname
# 退出bash
\q
```

## Java连接操作Postgres

### 1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.5.0</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

### 2. 配置

```yml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/test
    username: postgres
    password: postgres

mybatis-plus:
  type-aliases-package: com.basic.entity
  global-config:
    banner: false
```

> 测试略过，使用 Mybatis-plus 测试很简单。

### 3. 数据库类型和Java类型对应关系

|database_type|JavaType|description|
|--|--|--|
|timstamp with time zone|OffsetDateTime|带时区的时间戳|
|timstamp without time zone|LocalDateTime|不带时区的时间戳|
|timstamp without time zone|LocalDateTime|不带时区的时间戳|
