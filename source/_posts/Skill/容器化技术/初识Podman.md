---
categories:
  - Skill
tags:
  - skill
  - podman
title: 初识 Podman
---

[Podman](https://podman.io/) 和 Docker 一样，也是一个容器化技术的一种实现。它是全开源的。但是它们的底层架构是不同的，Podman 不需要使用 `root` 用户创建容器，也没有 `deamon` 进程，所以 Podman 不支持`--restart` 策略。但是为了迁移方便，Podman 团队采用了和 Docker 类似的命令。

<!-- more -->

## 一、安装

撰写本文的时候 Podman 已更新到 4.4.0 版本，本文所有涉及 Podman 的命令全部都是在此版本的基础上进行的。

```bash
# 1. 安装
yay -S podman podman-compose
# 2. 查看当前用户名称
whoami
# 3. 把用户名称加入到 `/etc/subuid、/etc/subgid` 两个文件中
sudo vim /etc/subuid
sudo vim /etc/subgid
# 4. 测试
podman -v

# 使用搜索命令，会发现没有任何结果。这是因为需要配置一下
podman search nginx

# 编辑配置文件 /etc/containers/registries.conf
# 备份文件
sudo cp /etc/containers/registries.conf /etc/containers/registries.conf.bak
sudo vim /etc/containers/registries.conf
```

进入文本查看时，会发现所有的内容都是被注释掉的，找到以下内容去掉注释并修改，保存退出。

```toml /etc/containers/registries.conf
unqualified-search-registries = ["docker.io"]
  
[[registry]]
prefix = "docker.io"
location = "registry.docker-cn.com"
```
> `unqualified-search-registries` 可以配置多个源，镜像的搜索会从这些源中去寻找。
> `registry##prefix、location` 是镜像拉取时的配置。

举个栗子，用以上配置举例：

```bash
# 搜索 unqualified-search-registries 配置的所有源中所有符合条件的镜像
# 如果你配置了多个，那么你就会在结果中看到有的镜像带有不同的前缀
podman search nginx

# 拉取镜像
podman pull nginx
# 虽然自己写的是上面的命令，但是 Podman 会自动加入配置
# 我们配置了 prefix，实际命令是这样：podman pull docker.io/nginx
# 如果还配置了 location，就会是这样：podman pull registry.docker-cn.com/nginx
```

## 二、命令

命令大多和 Docker 命令一样，只是开头变成了 podman，如果不习惯可以配置 docker 别名（前提是只安装了 podman）。

命令就不详细说了，可以参阅官方文档：https://docs.podman.io/en/latest/Commands.html
