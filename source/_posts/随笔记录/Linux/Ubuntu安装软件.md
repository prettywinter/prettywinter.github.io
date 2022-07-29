---
title: Ubuntu安装常用软件
categories: Linux
tags: [Ubuntu, 软件安装]
---

Ubuntu 中提供了方便的包管理工具 [apt-get](https://www.computerhope.com/unix/apt-get.htm)，安装软件什么的非常好用。

<!-- more -->

## apt-get命令

```bash
# 查看所有的安装软件
dpkg --list

# 卸载指定软件并删除配置文件
sudo apt-get --purge remove 包名
# 列出可更新的软件包
apt list --upgradeable
# 清理不再使用的依赖和库文件
sudo apt autoremove
```

如果你想了解一下 wget VS curl 的区别：`https://www.geeksforgeeks.org/difference-between-wget-vs-curl/`

## Redis

```bash
wget https://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make && make install
redis-server
```

## PostgreSQL

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
```

## [Docker](https://docs.docker.com/engine/install/ubuntu/)

```bash
# 卸载旧版本
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker’s official GPG key:
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 设置稳定版仓库
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 更新包索引
sudo apt-get update

####################### 下面二选一 #############################

# 1. 安装最新版本的 Docker Engine-Community 和 containerd ，或者转到下一步安装特定版本：
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 2.
# 查看版本
apt-cache madison docker-ce
# 选择指定的版本安装
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
```

## [btop++](https://github.com/aristocratos/btop)

```bash
# 下载解压安装
wget https://github.com/aristocratos/btop/releases/download/v1.2.7/btop-x86_64-linux-musl.tbz
mkdir btop && tar xvf btop-x86_64-linux-musl.tbz -C btop
cd btop
./install.sh
# 运行
btop
```

## [Nodejs](https://github.com/nodesource/distributions/blob/master/README.md)

```bash
# Using Ubuntu
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## [最新版稳定Git](https://git-scm.com/download/linux)

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install -y git
```

## [open-jdk](https://openjdk.org/install/)

```bash
sudo apt-get install openjdk-8-jdk
```

## [Nginx](https://nginx.org/download/nginx-1.22.0.tar.gz)

```bash
wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.40/pcre2-10.40.tar.gz
wget https://nginx.org/download/nginx-1.22.0.tar.gz
tar -zxvf nginx-1.22.0.tar.gz -C nginx && cd nginx
```
