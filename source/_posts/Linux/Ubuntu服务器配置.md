---
title: Ubuntu 服务器配置
categories:
  - Linux
  - Ubuntu
tags:
  - linux
  - ubuntu
cover: 'linux,ubuntu'
abbrlink: af7fa1dd
---

Ubuntu-Server 配置，桌面版应该也能用，如果你喜欢命令的话，应该都没问题。

<!-- more -->

## 一、Ubuntu常用命令 

### 1. [apt-get]((https://www.computerhope.com/unix/apt-get.htm))

```bash
# 查看所有的安装软件
dpkg --list
# 安装已经下载好的 deb 安装包
dpkg -i 安装包

# 更新所有可更新的软件包
sudo apt update && sudo apt upgrade

# 卸载指定软件并删除配置文件
sudo apt --purge remove 包名
# 清理不再使用的依赖和库文件
sudo apt autoremove
```

## 二、基本配置

### 1. 时区设置

Ubuntu 可以使用内置的 timedatectl 设置系统的时区。

```bash
# 查看当前的时区
timedatectl
# 查看支持的时区列表
timedatectl list-timezones
# 是否含有中国时区
timedatectl list-timezones | grep Asia/Shanghai
# 设置中国时区
timedatectl set-timezone Asia/Shanghai

# 无需重启，再次查看
timedatectl
```

### 2. 语言设置

```bash
# 查看当前系统的语言配置
locale
# 如果使用的 Ubuntu 22.04 以上使用这个命令将更加简单容易，选择完成之后注销重启
sudo dpkg-reconfigure locales
# 再次查看配置
locale

# 下面的适用旧版本
# 列出所有启用的区域设置
locale -a

# 编辑 locale.gen 文件
vim /etc/locale.gen
# 取消 zh_CN.UTF-8 UTF-8 en_US.UTF-8 UTF-8 两行的注释

# 编辑保存后，执行以下命令
locale-gen
# 显示正在使用的 Locale 和相关的环境变量
locale
# 设置整个系统使用的区域设置
localectl set-locale LANG=zh_CN.UTF-8
# 立即生效
unset LANG
```

> 如果使用上面的立即生效命令仍然有问题，没有生效，请重启计算机。

### 3. 下载工具

Linux 下载工具有很多，比如 Motrix、文件蜈蚣等跨平台的免费开源下载器，而且还有许多命令行下载器，如果喜欢命令行可以使用 wget 或者 curl。

如果你想了解一下 wget VS curl 的区别：可以访问这篇文章：`https://www.geeksforgeeks.org/difference-between-wget-vs-curl/`

```bash
# 查看本机基础信息
curl ipinfo.io
# 本机公网
curl cip.cc
```

### 4. Vim 配置

如果 Vim 配置的好，那么日常编辑文件是非常流畅的。很多大佬用 Vim 写代码，当然，浪子做不到 QAQ。

Vim 的全局配置文件在 `/etc/vim/vimrc` 或者 `/etc/vimrc`，对所有用户生效；用户的配置文件一般在 `~/.vimrc` 中。

我们修改全局的配置，编辑 {% mark /etc/vim/vimrc color:green %} 文件，没有的话可以手动创建一个。下面给了一些简单实用的配置。

```bashrc .vimrc
" 自动缩进
set autoindent
" 语法高亮
set syntax=on
" Tab键的宽度
set tabstop=4
" 突出显示当前行
set cursorline
" 搜索忽略大小写
set ignorecase
" 去掉输入错误的提示声音
set noeb
" 高亮显示匹配的括号
set showmatch
" 总是显示行号
set nu

" 侦测文件类型
filetype on
" 打开文件类型检测, 加了这句才可以用智能补全
set completeopt=longest,menu
"代码补全 
set completeopt=preview,menu
" 增强模式中的命令行自动完成操作
set wildmenu

" 状态行显示的内容（包括文件类型和解码）
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}
set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]
" 总是显示状态行
set laststatus=2
```

## 三、常用软件

### Redis

```bash
wget https://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make && make install
redis-server
```

### PostgreSQL

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
```

### [Docker](https://docs.docker.com/engine/install/ubuntu/)

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

### [btop++](https://github.com/aristocratos/btop)

```bash
# 下载解压安装
wget https://github.com/aristocratos/btop/releases/download/v1.2.8/btop-x86_64-linux-musl.tbz
mkdir btop && tar xvf btop-x86_64-linux-musl.tbz -C btop
cd btop
./install.sh
# 运行
btop
```

### [Nodejs](https://github.com/nodesource/distributions/blob/master/README.md)

```bash
# Using Ubuntu
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### [最新版稳定Git](https://git-scm.com/download/linux)

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install -y git
```

### [open-jdk](https://openjdk.org/install/)

```bash
sudo apt-get install openjdk-8-jdk
```

### [Nginx](https://nginx.org/download/nginx-1.22.0.tar.gz)

```bash
wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.40/pcre2-10.40.tar.gz
wget https://nginx.org/download/nginx-1.22.0.tar.gz
tar -zxvf nginx-1.22.0.tar.gz -C nginx && cd nginx
```

> 修改配置文件再次启动报错：`[emerg]: bind() to 0.0.0.0:80 failed (80: Address already in use)`，执行命令：sudo fuser -k 80/tcp，然后 ./nginx

## 四、其它

### 查找apt安装的软件的路径

这里以jdk为例

```bash
which java
# 多次执行，直到最后打印出一些系统信息
file 上一条命令打印的路径
```

### 开放指定端口

随着 Ubuntu 的更新，它的各种软件也在更新，以前用 `firewall`，新版本（22.04）已经默认使用 `ufw`，相比较而言，[UFW](https://help.ubuntu.com/community/UFW) command more easily，不过具体使用这里就不详细写了，可以参考官网的说明，很简单。

```bash
# 启动防火墙
systemctl start firewalld
# --zone 作用域 --add-port 格式 端口/通讯协议 --permanent 永久生效
firewall-cmd --zone=public --add-port=1935/tcp --permanent
# 重启
firewall-cmd --reload
# 查看当前所有tcp端口
netstat -ntlp
# 查看所有1935端口使用情况 
netstat -ntulp | grep 1935

# 启用 ufw
ufw enable
# 禁用 ufw
ufw disable
# 查看状态
ufw status verbose
# 放行端口
ufw allow port/协议（TCP|UDP）
# 关闭端口
ufw deny port/协议
# 允许特定 IP 访问
ufw allow from IP
```

### [ftp服务搭建](https://www.geeksforgeeks.org/how-to-setup-and-configure-an-ftp-server-in-linux/)

[下载](https://pkgs.org/download/vsftpd) 完成后上传至服务器，执行以下命令安装

```bash
dpkg -i xxx.deb
```

安装完成后编辑 `/etc/vsftpd.conf` 文件：

```conf
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
# 新增
pasv_enable=Yes
pasv_min_port=10000
pasv_max_port=10100
allow_writeable_chroot=YES
```

更新防火墙：`sudo ufw allow from any to any port 20,21,10000:10100 proto tcp`

重启服务：`sudo systemctl restart vsftpd`

创建新用户并设置密码：

```bash
sudo useradd -m ftptest
sudo passwd ftptest
New password: 
Retype new password: 
passwd: password updated successfully

 
# 出于安全考虑，限定用户ftpuser不能登录，只能ftp
usermod -s /sbin/nologin ftpuser

# 出于安全考虑，我们将禁用此用户的 ssh 权限。输入

sudo vim /etc/ssh/sshd_config
# 在此文件中添加以下行，然后重启 SSH sudo systemctl restart sshd
DenyUsers ftpuser
```

添加文件(连接 ftp 时展示的目录为登录的用户的主目录，所以为了安全我们使用新建的用户)：

`sudo echo "this is a simple file" > /home/ftptest/test.txt`

然后就可以连接测试了：`ftp://ip`

## 五、一些问题

### 1. Docker

Ubuntu version 22.04
Docker version 23.0.1
Docker Compose version v2.16.0
安装 Docker 工具：snap
错误原因：
```bash
Cannot Connect to the Docker Daemon at ‘unix:///var/run/docker.sock’
```

解决参考：https://appuals.com/cannot-connect-to-the-docker-daemon-at-unix-var-run-docker-sock/

如果不能解决，尝试下面的方法：

```bash
# 停止 docker
systemctl stop docker.socket
systemctl stop docker

# 查看 snap 管理的 docker 状态
snap services
# 停止服务
snap stop docker

# 然后重启
systemctl start docker
```
