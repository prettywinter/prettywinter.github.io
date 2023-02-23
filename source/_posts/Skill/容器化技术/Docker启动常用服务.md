---
title: Docker启动常用服务
categories:
  - Skill
  - Docker
tags:
  - docker
cover: 'https://fastly.jsdelivr.net/gh/prettywinter/dist/images/blogcover/docker.jpeg'
abbrlink: 1ca431ad
---

[Docker Hub](https://registry.hub.docker.com/) 上可以搜索并查看镜像以及版本，选择合适的进行拉取/下载。相应的镜像里都有官方的的启动说明可以参考，本文只是水文。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=2 orderedList=false} -->

<!-- code_chunk_output -->

- [1. MySQL](#1-mysql)
- [2. Redis](#2-redis)
- [3. Nginx](#3-nginx)
- [4. RabbitMQ](#4-rabbitmq)
- [5. ElasticSeach And Kibana](#5-elasticseach-and-kibana)

<!-- /code_chunk_output -->

Docker Hub 上的镜像有三种标签是可以信任的：

1. DOCKER OFFICIAL IMAG：Docker 官方镜像，是一组精心策划的 Docker 开源和即插即用解决方案存储库。
2. Verified Publisher：Docker验证镜像，来自 Docker 验证的出版商的高质量镜像。这些产品由商业实体直接发布和维护。这些镜像不受速率限制。
3. Sponsored OSS：Docker 赞助的开源软件镜像。这些镜像由 Docker 通过开源计划赞助的开源项目发布和维护。

## 1. MySQL

```bash
# 创建所需目录
mkdir -p /data/docker-service/mysql/conf.d
mkdir -p /data/docker-service/mysql/data
cd /data/docker-service/mysql
# 拉取镜像
docker pull mysql:8.0.30
# 启动一个简单的基础镜像（只是用来复制配置文件，用完就删除）
docker run --name mysql-test -e MYSQL_ROOT_PASSWORD=root -d mysql:8.0.30
docker cp mysql-test:/etc/my.cnf conf.d/

# 删除镜像
docker rm mysql-test

# 使用自定义的配置文件启动MySQL并且在启动时创建一个数据库
docker run -d -p 3306:3306 --name mysql -v /data/docker-service/mysql/conf.d:/etc/mysql/conf.d -v /data/docker-service/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -v MYSQL_DATABASE=要创建的数据库名称 --restart=always mysql:8.0.30
```

启动完成后，进入 MySQL 的 bash 环境: `docker exec -it mysql bash`，执行下面两条命令开启远程连接：

```bash
# password 根据自身情况修改
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
# 刷新权限
FLUSH PRIVILEGES;
```

如果需要导入之前的数据库备份文件到此容器中，可以使用 `docker cp /data/docker-service/mysql/data/备份文件.sql mysql:/var/lib/mysql`，然后使用 `source 备份文件.sql;` 加载数据到容器。

> 如果本机使用可视化工具（dbeaver、navicat）连接出现以下错误：Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'，请使用 127.0.0.1 连接，不要使用 localhost。

## 2. Redis

Redis默认开启的是快照模式(RDB)，可以开启AOF持久化(最多丢1s内数据)

```bash
# 创建所需目录
mkdir -p /data/docker-service/redis/conf
mkdir -p /data/docker-service/redis/data
cd /data/docker-service/redis

# 此处如果使用自定义配置文件，在 conf 文件中建立 redis.conf 文件添加内容即可

# 指定配置文件并开启AOF持久化后台启动
docker run -p 6379:6379 --name redis -v /data/docker-service/redis/conf:/usr/local/etc/redis -v /data/docker-service/redis/data:/data -d redis:6 redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

## 3. Nginx

```bash
# 创建所需目录
mkdir -p /data/docker-service/nginx/conf.d
mkdir -p /data/docker-service/nginx/html
mkdir -p /data/docker-service/nginx/log
cd /data/docker-service/nginx

# 拉取镜像
docker pull nginx:1.22.1
# 启动一个简单的基础镜像（只是用来复制配置文件，用完就删除）
# 如果自己复制文件内容，此步可以跳过
docker run --name nginx-test -d nginx:1.22.1
docker cp nginx-test:/etc/nginx/nginx.conf ./
docker cp nginx-test:/etc/nginx/conf.d/default.conf conf.d/

# 删除镜像
docker stop nginx-test && docker rm nginx-test

# 启动
docker run --name nginx -p 80:80 -v /data/docker-service/nginx/nginx.conf:/etc/nginx/nginx.conf:ro -v /data/docker-service/nginx/html:/usr/share/nginx/html -v /data/docker-service/nginx/log:/var/log/nginx -v /data/docker-service/nginx/conf.d:/etc/nginx/conf.d -d nginx:1.22.1
```

> 注意：nginx的配置文件必须和版本一致。
> `ro` 代表只读(read only): 外部的改变能够影响内部，内部的改变不会影响外部。

## 4. RabbitMQ

```bash
# 使用自定义配置信息启动
docker run -d --name RabbitMQ -p 15672:15672 -p 5672:5672 -v /data/docker-service/rabbitmq/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf rabbitmq:3.8-management
```

5672 端口是与程序进行通信的，比如Java。rabbitmq:3.8-management 连带服务和管理界面的插件一并启动(默认的账号密码：guest/guest)，而 rabbitmq:3.8 是没有管理界面的。

如果在启动时指定用户名、密码可以加上以下内容。

```bash
-e RABBITMQ_DEFAULT_VHOST=/ems
-e RABBITMQ_DEFAULT_USER=root
-e RABBITMQ_DEFAULT_PASS=root
```

## 5. ElasticSeach And Kibana

ElasticSeach 官方指导教程：https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html
Kibana 官方教程传送：https://www.elastic.co/guide/en/kibana/current/docker.html

> 直接启动 ES 容器会报错，解决方案：
> 1.编辑宿主机 vim /etc/sysctl.conf 加入 `vm.max_map_count=262144`，保存退出
> 2.sysctl -p
