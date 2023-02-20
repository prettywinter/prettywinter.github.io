---
title: Docker安装Oracle11g
categories:
  - Skill
  - Docker
tags:
  - docker
abbrlink: 6e4c8a2f
---

使用 Docker 安装 Oracle 数据库，注意拉取的镜像，以下操作都是在拉取的镜像的基础上进行的。

<!-- more -->

## 镜像获取

```bash
# 拉取镜像，推荐使用这一个，不然后续无法进行
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
# 运行oracle 11g镜像
docker run -d -p 1521:1521 --name oracle11g registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
# 进入容器
docker exec -it oracle11g bash

```

## 导入环境变量

```bash
# 切换到root用户 密码：helowin
su - root
```

编辑 `vim /etc/profile`

```bash /etc/profile
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH
``` 

## Oracle配置

```bash
# 切换到oracle用户
su - oracle
# 进入oracle目录
cd $ORACLE_HOME
# 启动
$ORACLE_HOME/bin/sqlplus /nolog
# 连接
conn /as sysdba
# 修改system的密码并设置密码的有效时间为无限
alter user system identified by oracle;
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
# 创建 test 用户
create user test identified by test;
# 授权
grant connect,resource,dba to test;
```

## 创建表空间

表空间需要如下设置后才能创建，否则会创建失败。

```bash
show parameter db_create_file;
 
ALTER SYSTEM SET db_create_file_dest = "/home/oracle/app/oracle/oradata"
```
