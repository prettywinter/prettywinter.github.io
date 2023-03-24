---
title: CentOS 服务器配置
categories:
  - Linux
  - CentOS
tags:
  - centos
cover: 'https://fastly.jsdelivr.net/gh/prettywinter/dist/images/blogcover/linux.jpeg'
abbrlink: fa20a4d7
---

此篇文章以后随缘更新，这个系统的相关问题网上一般都有答案，浪子个人已不再使用 CentOS 系统。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=true}-->

<!-- code_chunk_output -->

- [CentOS7最小安装](#centos7最小安装)
  - [1、配置网卡](#1配置网卡)
  - [2、关闭防火墙以及 Linux 的一些安全策略](#2关闭防火墙以及-linux-的一些安全策略)
  - [3、配置本地 yum 源](#3配置本地-yum-源)
  - [4、安装常用工具](#4安装常用工具)
  - [5、安装依赖关系](#5安装依赖关系)
  - [6、修改yum源](#6修改yum源)
  - [rpm命令](#rpm命令)
- [安装软件](#安装软件)
  - [1. Redis](#1-redis)
  - [2. Git](#2-git)
  - [3. RabbitMQ](#3-rabbitmq)
  - [4. Python](#4-python)
  - [5. Nginx](#5-nginx)
  - [6. Btop++](#6-btop)
  - [7. FFmpeg](#7-ffmpeg)
- [Linux通用二进制文件安装](#linux通用二进制文件安装)
  - [1. Node](#1-node)
  - [2. MongoDB](#2-mongodb)
  - [3. MySQL](#3-mysql)
    - [密码正确但是进不去 bash 环境](#密码正确但是进不去-bash-环境)
    - [预读处理](#预读处理)
    - [Can‘t connect to local MySQL server through socket ‘/tmp/mysql.sock‘ (2)](#cant-connect-to-local-mysql-server-through-socket-tmpmysqlsock-2)
- [Some Questions](#some-questions)
  - [1. 关于源码编译安装失败](#1-关于源码编译安装失败)
  - [虚拟机](#虚拟机)
  - [防火墙](#防火墙)

<!-- /code_chunk_output -->

## CentOS7最小安装

### 1、配置网卡

先进行网络的连接，编辑网络配置文件（`vi /etc/sysconfig/network-scripts/ifcfg-ens32`），不同的机器最后的文件名称可能不同，一般都是 `ifcfg-` 开头。

```bash
# 空着的部分自定义即可
BOOTPROTO=static
ONBOOT=yes
IPADDR=
NETMASK=
GATEWAY=
DNS1=
```

设置完成后，保存退出，使用命令 `systemctl restart network` 重启网卡。

### 2、关闭防火墙以及 Linux 的一些安全策略

```bash
systemctl stop firewalld
systemctl disable firewalld

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

setenforce 0
```

### 3、配置本地 yum 源

```bash
# 进入目录
cd /etc/yum.repos.d/
# 创建 备份 文件夹
mkdir bak
# 移动该目录下的所有文件到备份文件夹
mv * bak
# 拷贝一份文件进行编辑
cp bak/CentOS-Media.repo /etc/yum.repos.d/CentOS-Media.repo
```

编辑刚才我们拷贝的文件：`vi CentOS-Media.repo`，这就是安装软件时读取的安装源配置，加入以下内容，先使用本地镜像安装。

```bash
[linux]
name=linux
baseurl=file:///media/
gpgcheck=0
enabled=1
```

清除yum缓存：`yum -y clean all`
重建yum缓存：`yum makecache`

### 4、安装常用工具

`yum -y install curl telnet vim wget lrzsz net-tools`

修改vim配置（可以不修改，按照默认的即可，这里仅仅是偏好）

```bash
vim ~/.vimrc
set encoding=utf-8      " 文件编码
set number              " 显示行号
set tabstop=4           " tab宽度为4
set softtabstop=4        " 设置一次可以删除4个空格
set expandtab           " tab转换为空格
set nowrap              " 不自动换行
set showmatch          " 显示括号配对情

syntax on              " 开启语法高亮   
```

### 5、安装依赖关系

`yum -y install gcc gcc-c++ make autoconf wget lrzsz libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel openssl openssl-devel libxslt-devel libevent-devel libtool libtool-ltdl bison gd gd-devel vim-enhanced pcre-devel zip unzip ntpdate sysstat patch bc expect rsync`

### 6、修改yum源

```bash
# 复制文件
cp /etc/yum.repos.d/bak/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
# 进入目录
cd /etc/yum.repos.d/
# 添加 网易 的下载源
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
# 重建缓存
yum makecache
# 更新源
yum -y update
```

如果不想使用 `wget http://mirrors.163.com/.help/CentOS6-Base-163.repo` 的话，可以自己编辑：`/etc/yum.repos.d/CentOS-Base.repo` 这个文件(以下是用的清华源)。编辑完成之后更新库，这时需要网络。

```bash
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### rpm命令

安装：rpm -ivh xxx.rpm
卸载：rpm -evh xxx.rpm
更新：rpm -Uvh xxx.rpm
显示所有已安装软件：rpm -qa
CentOS安装lrzsz工具：sz下载，rz上传

## 安装软件

### 1. Redis

{% link https://redis.io/download/ 官网源码下载 %}

```bash
# 安装编译 redis 需要的工具
sudo yum -y install gcc automake autoconf libtool make
# 进入解压目录，进行编译，直到编译完成
make MALLOC=libc
# 安装到指定路径
make install PREFIX=/usr/local/redis
```

> Redis 的默认的配置文件在源码解压后的目录中

### 2. Git

{% link https://github.com/git/git/tags 官网源码下载 %}

```bash
# 安装编译 Git 源码的工具和依赖
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

# 卸载旧版 Git
yum -y remove git

# 编译 Git 源码
make

# 安装git至指定的路径
make prefix=/usr/local/git install

# 配置环境变量： 编辑文件 vi /etc/profile 
export PATH=$PATH:/usr/local/git/bin

# 刷新环境变量
source /etc/profile

# 查看Git是否安装完成
git --version
```

### 3. RabbitMQ

RabbitMQ 是使用 Erlang 语言编写的中间件，联想一下 Java，我们可以猜到它需要先搭建 Erlang 环境。主要是这个环境需要编译源码，RabbitMQ 本身官网提供了二进制压缩包。

{% link https://www.erlang.org/downloads Erlang源码下载 %}

一、Erlang环境搭建

```bash
# 安装编译 Erlang 的相关依赖
yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto
# 解压源码包
tar -zxvf otp_src_24.1.7.tar.gz

cd otp_src_24.1.7/
# 指定安装目录
./configure --prefix=/usr/local/erlang
# 编译安装
make && make install
# 测试安装是否成功：
cd /usr/local/erlang/bin/
./erl

# 安装成功后配置环境变量 vim /etc/profile
export PATH=$PATH:/usr/local/erlang/bin

# 添加完成保存退出，刷新使其生效
source /etc/profile
```

二、安装Rabbitmq

通过第一步，我们就搭建好了 Erlang 环境，接下来就是安装 RabbitMQ 了，这个还是比较简单的，因为它已经编译好了，我们可以下载直接配置，无需编译。

{% link https://www.rabbitmq.com/install-generic-unix.html 二进制文件包下载 %}

```bash
# 解压
tar -Jxvf rabbitmq-server-generic-unix-3.9.11.tar.xz
# 移动
mv rabbitmq_server-3.9.11 /usr/local/rabbitmq

# 添加环境变量：vim /etc/profile
export PATH=$PATH:/usr/local/rabbitmq/sbin
# 刷新变量
source /etc/profile
```

### 4. Python

Linux 下基本不需要配置 Python 的环境，有个别的 ISO 镜像版本比较老，比如 CentOS7 的 mini ISO 镜像是2.x的，我们可以通过各大系统的包管理工具进行安装，也可以自己通过源码编译安装。

{% link https://www.python.org/downloads/ 源码下载 %}

```bash
# 安装相关工具和依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel

# 解压压缩包
tar -zxvf Python-3.10.2.tgz  
# 进入文件夹
cd Python-3.10.2

# 配置安装位置
./configure prefix=/usr/local/python3

# 编译并安装
make && make install

# 使用 python3 验证是否安装成功
python3 -V

#添加python3的软链接 
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python3 

#添加 pip3 的软链接 
ln -s /usr/local/python3/bin/pip3.8 /usr/bin/pip3
```

### 5. Nginx

{% link https://nginx.org/en/ 官网源码下载 %}

```bash
# 解压
tar -zvxf nginx-1.20.2.tar.gz
# 配置安装路径
./configure --prefix=/usr/local/nginx
# 编译安装
make && make install
# 进入安装目录查看是否安装成功
cd /usr/local/nginx

# 启动 停止 重启
./nginx start
./nginx -s stop
./nginx -s reload
```

### 6. Btop++

Btop++ 是一个 Linux 资源监视器，显示处理器、内存、磁盘、网络和进程的使用情况和统计资料，界面美观，使用简单。这里使用了源码编译安装，但是浪子推荐下载 Github 仓库的二进制包解压运行 install.sh 脚本安装。

{% link https://github.com/aristocratos/btop Github地址 %}
{% link https://gitee.com/mirrors/btop Gitee同步仓库 %}

```bash
# 安装、升级相关依赖工具
yum install coreutils sed build-essential -y
yum install centos-release-scl -y
yum install devtoolset-10 -y
scl enable devtoolset-10 bash 
echo "source /opt/rh/devtoolset-10/enable" >> /etc/profile

# 克隆源码编译安装
git clone https://gitee.com/mirrors/btop.git
cd btop
make && make install
```

### 7. FFmpeg

{% link http://www.ffmpeg.org/ FFmpeg使用 %}

```bash
# 克隆源码 也可以下载 https://github.com/FFmpeg/FFmpeg/releases 相应的包上传
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg

# 安装相关依赖
yum install yasm.x86_64 -y

cd ffmpeg
./configure --enable-ffplay --enable-ffserver --prefix=/usr/local/ffmpeg
# 编译安装
make && make install

# 查看是否安装成功
cd /usr/local/ffmpeg
```

## Linux通用二进制文件安装

通用二进制文件一般使用包管理工具即可快速安装使用，是非常方便的。

### 1. Node

{% link https://nodejs.org/en/download/ 二进制包下载 %}

Node 可以直接下载二进制文件，解压配置环境变量即可。

```bash
# 官网下载二进制压缩包是 xz 结尾的，和下面的镜像站下载的略有不同
tar -Jvxf node-v16.15.0-linux-x64.tar.xz -C /usr/local/
cd /usr/local/
mv node-v16.15.0-linux-x64/ nodejs
ln -s /usr/local/nodejs/bin/node /usr/local/bin
ln -s /usr/local/nodejs/bin/npm /usr/local/bin
```

如果嫌弃官网下载速度慢可以到这里 {% copy width:max https://registry.npmmirror.com/binary.html?path=node/latest-v16.x/ %} 选择合适的版本下载。

### 2. MongoDB

{% link https://www.mongodb.com/try/download/community 二进制包下载 %}

```bash
# 解压
tar -xvzf mongodb-linux-x86_64-rhel62-3.4.22.tgz
mv mongodb-linux-x86_64-rhel62-3.4.22 /usr/local/mongodb

# 创建数据存储目录、工作目录以及日志目录
cd /usr/local/mongodb/
mkdir conf
mkdir data
mkdir logs

# 添加环境变量: vim /etc/profile
export MONGODB_HOME=/usr/local/mongodb  
export PATH=$PATH:$MONGODB_HOME/bin

# 使环境变量生效
source /etc/profile
```

顺带说一下配置，编辑 MongoDB 的配置文件 `mongodb.conf`，修改以下内容：

```bash
# 数据存储目录
dbpath = /usr/local/mongodb/data/db
# 日志存储目录
logpath = /usr/local/mongodb/logs/mongodb.log
# 指定端口号
port = 27017
# 以守护进程的方式启动，即后台运行
fork = true
# 可以连接的地址，开启远程登录
bind_ip = 0.0.0.0
# 启用密码验证，可根据实际情况配置
# auth = true
```

mongodb安装好后第一次启动服务是不需要密码的，也没有任何用户，通过shell命令可直接进入，cd到mongodb目录下的bin文件夹，执行命令 `./mongo` 即可。如果配置了环境变量，可以在任意路径执行 `mongo` 也可以。

```bash
# 指定配置文件启动服务
./mongod --config /usr/local/mongodb/conf/mongodb.conf
# 进入系统数据库
use admin
# 创建 root 用户
db.createUser( {user: "root",pwd: "xxx",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]});
# 查看创建的用户(以下两个命令二选一)
show users;
db.system.users.find();
# 关闭服务
db.shutdownServer();

# 启动时开启密码验证，也可以在配置文件中加入 auth = true 在启动时开启验证，这样就不用 --auth 选项了
./mongod --config /usr/local/mongodb/conf/mongodb.conf --auth
```

### 3. MySQL

MySQL 说实话我觉得通用的二进制文件包安装较为简单，卸载是最简单的，比使用包管理工具安装的简单的多。下面给两个包管理工具安装的参考链接，我没怎么用过，仅作参考。

yum安装：https://zhuanlan.zhihu.com/p/87069388

接下来进入正题，使用通用的 MySQL8.x 版本的二进制压缩包进行安装。至于卸载，就把有关 MySQL 创建的几个文件夹删掉就行了，`/etc/my.cnf` 是默认自带的，卸载的时候删不删都没有问题，如果默认没有这个文件也不必担心，可以手动添加。

```bash
# 检查mysql用户组和用户是否存在，如果没有，则创建
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql
# 创建 mysql 组
groupadd mysql
# 新建 mysql 用户并加入 mysql 群组
useradd -r -g mysql mysql

# 安装所需依赖(需要安装 libaio-devel.x86_64 numactl 这两个依赖)
yum -y install libaio-devel.x86_64 numactl

# 解压二进制文件包到 /usr/local/mysql 目录
tar xxx -C /usr/local/mysql
cd /usr/local/mysql/bin
# 初始化数据，成功初始化后需要记录最后 root@localhost: 后的字符串（初始化失败则不显示），它是后面进入 bash 环境的初始密码
./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql

# vim /etc/my.cnf(没有该文件手动创建) 修改内容
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
port = 3306

# 启动MySQL服务 启动成功会有 Starting MySQL.. SUCCESS! 提示；否则就是启动失败，根据提示查看日志记录定位问题
cd /usr/local/mysql/support-files/
mysql.server start

# 添加软链接并重启服务
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
service mysql restart
# systemctl restart mysql

# 添加开机自启
# 1、将服务文件拷贝到 init.d 下，并重命名为 mysql
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
# 2、赋予可执行权限
chmod +x /etc/init.d/mysqld
# 3、添加服务
chkconfig --add mysqld
# 4、显示服务列表
chkconfig --list
```

至此，安装任务基本完成，下面需要添加用户并分配权限，进入 MySQL 的 bash 环境需要之前进行初始化时生成的密码。

```bash
# 登录 MySQL，密码使用初始化成功时 root@localhost: 后的字符串
mysql -u root -p

# 修改密码 毕竟那么不好记
# 而且如果不修改 它会报 ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement. 的错误
alter user root@'localhost' identified by 'newpassword';
flush privileges;

# 开放远程连接
use mysql;
# 允许所有主机，都可以通过用户为root用户，密码为默认数据库登录密码，进行数据库操作
update user set host='%' where user='root';

# ① 适用于 MySQL 8.0之前的版本，可以直接授权
# grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;

# ② 适用于 MySQL 8.0之后的版本，需要先创建一个用户，再进行授权【推荐方式②】
create user dev@'%' identified by '123456';
grant select,update,delete,insert on *.* to dev@'%' with grant option;
# 刷新权限，这一句很重要，使修改生效，如果没有写，则还是不能进行远程连接。这句表示从mysql数据库的grant表中重新加载权限数据，因为MySQL把权限都放在了cache中，所以，做完修改后需要重新加载。
flush privileges;
```

初始化成功截图：
![初始化成功截图](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/Linux_MySQL初始化成功截图.png)

> 记录日志最末尾位置 `root@localhost:` 后的字符串，此字符串为mysql管理员临时登录密码。

#### 密码正确但是进不去 bash 环境

```bash
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES/NO) 
```

解决方法：

1. 使用 kill 命令停止 mysqld 相关服务
2. cd /usr/local/mysql/bin/，运行命令： `mysqld_safe --user=mysql --skip-grant-tables --skip-networking &`
3. 使用密码登录数据库 mysql -u root -p 并切换到 mysql 数据库
4. 执行命令：`update user set host='%' where user='root';`

#### 预读处理

```bash
mysql> use dbname;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A 
```

这个问题是之前使用的 apt-get 管理工具安装的 MySQL 出现的，原因是因爲數據庫採用了預讀處理。解决办法就是在我们进入MySQL的bash环境时，需要加入 `-A` 参数，不让其预读数据库信息，`mysql -u root -p -A`。如果覺得每次进入 bash 环境都要添加参数比较麻烦，也可以在 **`my.cnf`** 文件里加上如下內容：

```bash
[mysql]
no-auto-rehash
```

#### Can‘t connect to local MySQL server through socket ‘/tmp/mysql.sock‘ (2)

这是 `my.cnf` 的配置问题，下面的三块内容必须都要设置，不然就会使用默认的 socket，位于 `/tmp/mysql.sock` 目录，因此我们最好配置一下：

```conf /etc/my.cnf
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/mysql.sock
port=3306
 
[client]
default-character-set=utf8
socket=/usr/local/mysql/mysql.sock
 
[mysql]
default-character-set=utf8
socket=/usr/local/mysql/mysql.sock
```

或者我们打一个软链接：

``` bash
ln -s /usr/local/mysql/mysql.sock /tmp/mysql.sock
```

## Some Questions

### 1. 关于源码编译安装失败

如果源码编译失败，先确认所需依赖是否全部成功安装，然后清除上一次编译的缓存，之后再次编译，不然会一直失败。[参考网址](https://blog.csdn.net/wcnmlgb888/article/details/82713106)

```bash
# 清除上一次编译失败的缓存
make distclean
# 再次编译
make
```

### 虚拟机

centos7问题描述：
用的好好的虚拟机，之前内网都通，突然xshell连不上虚拟机了也连不上外网了，这时候怎么办呢？
> 解决方法：
1.将networkmanager服务停掉
systemctl stop NetworkManager
systemctl disable NetworkManager
2.重启网卡
systemctl restart network
如上操作，就可以啦

{% link https://blog.csdn.net/weixin_44695793/article/details/108089356 参考博客地址 %}
<!-- [原博客地址](https://blog.csdn.net/weixin_44695793/article/details/108089356) -->

**TIPS：** 如果下面的命令不是使用超级用户执行的话，可能不能安装成功，这个时候可以添加 `sudo` 选项进行重试。


### 防火墙

CentOS 7 使用的 firewall 
开放端口
firewall-cmd --list-port
开放端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent
重新加载
firewall-cmd --reload
禁用端口
firewall-cmd --zone=public --remove-port=8083/tcp --permanent



Centos 6 iptables
3.1 iptables的基本使用
启动：service iptables start

关闭：service iptables stop

查看状态：service iptables status

开机禁用：chkconfig iptables off

开机启用：chkconfig iptables on

3.2 开放指定端口语句
1）允许本地回环接口（即运行本机访问本机）：iptables -A INPUT -i lo -j ACCEPT

注：-A和-I参数分别为添加到规则末尾和规则最前面。

2）允许已建立的或相关联的通行：iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

3）允许所有本机向外的访问：iptables -P INPUT ACCEPTiptables -A OUTPUT -j ACCEPT

4）允许访问22端口：

iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -s 10.159.1.0/24 --dport 22 -j ACCEPT  
注：-s后可以跟 IP 段或指定 IP 地址，如果有其他端口的话，规则也类似。

5）允许ping：iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

6）禁止其他未允许的规则访问：

iptables -A INPUT -j REJECT 

iptables -A FORWARD -j REJECT

注：如果 22 端口未加入允许规则，SSH链接会直接断开。

3.3 屏蔽 IP
1）屏蔽单个IP：iptables -I INPUT -s 123.45.6.7 -j DROP

2）封整个段即从123.0.0.1到123.255.255.254：iptables -I INPUT -s 123.0.0.0/8 -j DROP

3）封IP段即从123.45.0.1到123.45.255.254：iptables -I INPUT -s 124.45.0.0/16 -j DROP

4）封IP段即从123.45.6.1到123.45.6.254：iptables -I INPUT -s 123.45.6.0/24 -j DROP

3.4 iptables 规则
查看已有规则：iptables -L -n

N：只显示IP地址和端口号，不将 IP 解析为域名

将所有 iptables 以序号标记显示，执行：iptables -L -n --line-numbers

添加规则

iptables -A和iptables -I

1）iptables -A

添加的规则是添加在最后面。如针对 INPUT 链增加一条规则，接收从 eth0 口进入且源地址为192.168.0.0/16网段发往本机的数据：

iptables -A INPUT -i eth0 -s 192.168.0.0/16 -j ACCEPT

2）iptables -I 

添加的规则默认添加至第一条。如果要指定插入规则的位置，则使用iptables -I时指定位置序号即可。

删除规则

如果删除指定规则，使用iptables -D命令。命令后可接序号或iptables -D接详细定义。

如果想把所有规则都清除掉，可使用iptables -F。

3.5 规则的保存与恢复
备份iptables rules

使用 iptables-save 命令，如：iptables-save > /etc/sysconfig/iptables.save

恢复iptables rules

使用 iptables 命令，如：iptables-restore < /etc/sysconfig/iptables.save

iptables 配置保存

做的配置修改，在设备重启后，配置将丢失。可使用service iptables save进行保存。

重启 iptables 的服务使其生效：service iptables save   

添加规则后保存重启生效：service iptables restart


3.6 开放端口步骤
3.6.1 方法一：通过命令行
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
service iptables save
service iptables restart
3.6.2 方法二：编辑配置文件
iptables的配置文件为/etc/sysconfig/iptables

编辑配置文件：vi /etc/sysconfig/iptables

1）配置文件中添加：

-A INPUT -p tcp --dport 80 -j ACCEPT
2）执行service iptables restart，重启生效。