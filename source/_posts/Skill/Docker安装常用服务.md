---
title: Docker安装常用服务
categories: skill
tags: [docker]
abbrlink: 4bb2ae3a
---

使用 Docker 安装常用服务，可以在 [Docker Hub](https://registry.hub.docker.com/) 上搜索镜像以及版本进行拉取/下载。带有 `offical image` 标签的都是官方镜像。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=2 orderedList=false} -->

<!-- code_chunk_output -->

- [1. MySQL](#1-mysql)
  - [使用自定义的配置文件启动MySQL并且在启动时创建一个数据库](#使用自定义的配置文件启动mysql并且在启动时创建一个数据库)
- [2. Redis](#2-redis)
  - [指定配置文件并开启AOF持久化后台启动](#指定配置文件并开启aof持久化后台启动)
- [3. Nginx](#3-nginx)
  - [使用自定义配置文件启动](#使用自定义配置文件启动)
- [4. RabbitMQ](#4-rabbitmq)
  - [使用自定义配置信息启动](#使用自定义配置信息启动)
- [5. MongoDB](#5-mongodb)
- [6. ES](#6-es)
  - [Docker-Desktop启动ElasticSearch失败](#docker-desktop启动elasticsearch失败)
  - [6.1 直接启动](#61-直接启动)
  - [6.2 使用自定义配置启动](#62-使用自定义配置启动)
  - [6.3 加载 IK 分词器启动](#63-加载-ik-分词器启动)
- [7. Kibana](#7-kibana)
  - [指定自定义配置文件启动](#指定自定义配置文件启动)

<!-- /code_chunk_output -->

## 1. MySQL

镜像拉取：`docker pull mysql:5.7.32`

### 使用自定义的配置文件启动MySQL并且在启动时创建一个数据库

```bash
docker run -d -p 3306:3306 --name mysql -v /root/docker/mysql:/etc/mysql/conf.d -v /root/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -v MYSQL_DATABASE=数据库名称 --restart=always mysql:5.7.32
```

> 导入备份数据到容器：docker cp /root/docker/test.sql 容器名称:/path
> 进入 MySQL 的 bash 环境: docker exec -it mysql bash
> 登录数据库，并选择数据库加载数据： mysql -u root -p && use test
> 加载数据：source test.sql;

## 2. Redis

Redis默认开启的是快照模式(RDB)，可以开启AOF持久化(最多丢1s内数据):

### 指定配置文件并开启AOF持久化后台启动

```bash
docker run -p 6379:6379 --name redis -v /home/data/redis/redis.conf:/etc/redis/redis.conf  -v /data/redis/data:/data -d redis:6.2.5 redis-server /etc/redis/redis.conf --appendonly yes
```

## 3. Nginx

### 使用自定义配置文件启动

```bash
docker run --name nginx -d -p 80:80 -v /root/docker/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.20
```

> 注意：nginx的配置文件必须和版本一致。
> `ro` 代表只读(read only): 外部的改变能够影响内部，内部的改变不会影响外部。

复制容器内部的配置文件到宿主机：`docker cp nginx:/etc/nginx/nginx.conf /root/nginx.conf`

## 4. RabbitMQ

### 使用自定义配置信息启动

```bash
docker run -d --name RabbitMQ -p 15672:15672 -p 5672:5672 -v /root/docker/rabbitmq/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf rabbitmq:3.8-management
```

5672 端口是与程序进行通信的，比如Java。rabbitmq:3.8-management 连带服务和管理界面的插件一并启动(默认的账号密码：guest/guest)，而 rabbitmq:3.8 是没有管理界面的。

如果在启动时指定用户名、密码可以加上以下内容。

```bash{.line-numbers}
-e RABBITMQ_DEFAULT_VHOST=/ems
-e RABBITMQ_DEFAULT_USER=root
-e RABBITMQ_DEFAULT_PASS=root
```

## 5. MongoDB

```bash
docker run --name mongo -d -p 27017:27017  mongo:5.0.5
```

## 6. ES

### Docker-Desktop启动ElasticSearch失败

在Windows下，使用 Docker 启动 ES 服务，如果遇到内存不足的问题。调整的方式是通过命令行的 `wsl` 命令进入 docker-desktop 的终端，然后通过 `sysctl` 命令调整系统参数。

```bash
# 使用 cmd 进入 docker
wsl -d docker-desktop
# 调整相应的参数，docker 启动 ES 服务时会有打印，修改为指定的即可
sysctl -w vm.max_map_count=262144
```

也可以编辑文件 `vi /etc/sysctl.conf` 加入 `vm.max_map_count=262144`，保存退出执行 `sysctl -p` 后重启 es 服务即可。

### 6.1 直接启动

```bash
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:7.0
```

> 直接启动 ES 容器会报错，解决方案：
> 1.编辑宿主机 vim /etc/sysctl.conf 加入 `vm.max_map_count=262144`，保存退出
> 2.sysctl -p

### 6.2 使用自定义配置启动

```bash
docker run -d --name es -p 9200:9200 -p 9300:9300 -v /root/docker/es/data:/usr/share/elasticsearch/data -v /root/docker/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:7.0
```

### 6.3 加载 IK 分词器启动

```bash
docker run -d --name es -p 9200:9200 -p 9300:9300 -v /root/docker/es/data:/usr/share/elasticsearch/data -v /root/docker/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /root/docker/es/ik:/usr/share/elasticsearch/plugins/ik elasticsearch:7.0
```

## 7. Kibana

### 指定自定义配置文件启动

```bash
docker run -d --name kibana -p 5601:5601 -v /root/docker/kibana.yml:/usr/share/kibana/config/kibana.yml Kibana:7.0
```
