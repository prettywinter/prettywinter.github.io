---
title: ArchLinux/Manjaro安装一些常用软件
categories:
  - Linux
  - Arch
tags:
  - linux
  - arch
  - manjaro
  - 软件安装
cover: 'https://fastly.jsdelivr.net/gh/prettywinter/dist/images/blogcover/arch.jpeg'
---

整理一些 ArchLinux/Manjaro 中常用的软件。浪子更加推荐去 [Arch Wiki](https://wiki.archlinux.org/) 寻找安装方法进行安装，包括在使用中的一些问题也可以在此找到解决办法。

<!-- more -->

## 1. yay

如果是新安装的系统，请先执行 `sudo pacman -Syu` 进行更新。

不管你是使用 Arch 还是 Manjaro，浪子都推荐少侠使用该命令，无须加入 sudo 前缀，重要的是，yay 可以安装 AUR 的安装包，pacman 支持的命令 yay 都支持，并且更为丰富。

{% link https://zhuanlan.zhihu.com/p/363666022 详细了解 %}

yay 常用命令：

```bash
# 安装 yay
sudo pacman -S yay

# 一些常用的命令
# 更新软件库（全部更新）
yay -Syu
# 安装
yay -S <package-name>
# 更新指定包软件
yay -u <package-name>
# 卸载该软件和没有被使用的依赖
yay -Rs <package-name>
# 若要删除当前未安装的所有缓存包和未使用的同步数据库
yay -Sc
# 查找符合条件的安装包
yay -Ss <package-name>
# 查看安装的软件 yay 信息
yay -Ps
# 查看 yay 默认配置
yay -Pd
# 查看 yay 当前设置
yay -Pg
```

## 2. 中文输入法

### 2.1 kde、xfce推荐 [fcitx5+rime](https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

```bash
yay -S fcitx5-im fcitx5-rime fcitx5-chinese-addons
```

编辑 `/etc/environment` 文件，没有这个文件的话可以手动创建，然后添加以下内容，之后重启电脑：

```bash /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

> fictx5 是一个框架，Rime 是一个输入法，非常推荐使用。

### 2.2 Gnome桌面推荐 [ibus](https://wiki.archlinux.org/title/IBus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

```bash
yay -S ibus ibus-libpinyin
```

GNOME 默认使用 IBus， 所以你只需要安装你需要的输入法引擎（但是 ibus 必须安装），并打开设置界面，通过“键盘”中的“输入源”添加。在你添加至少两个输入源后，GNOME 会在托盘中显示输入选择图标。如果如此操作之后你没有成功，很可能你没有完成 locale-gen。默认切换输入法的快捷键是 Super+Space; 请忽视 ibus-setup 中的添加方法，这不会真的添加新的输入法，且 ibus-setup 中的配置不会对输入法生效。

```bash /home/user/.xprofile
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -x -d
```

> 其它问题请参考Arch Wiki：https://wiki.archlinux.org/title/IBus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#GNOME

## 3. 字体

安装好了输入法，当然也要有一些字体来辅助，下面有四款开源的字体，其中，文泉驿和思源黑体支持中文，其他两款只支持英文。

```bash
# 文泉驿字体
yay -S wqy-bitmapfont wqy-microhei wqy-zenhei wqy-microhei-lite

# 思源字体
yay -S noto-fonts-cjk adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts

# Fira Code
yay -S ttf-fira-code woff2-fira-code woff-fira-code

# JetBrains Mono
yay -S ttf-jetbrains-mono
```

## 4. Window的一些软件

Window 下的常用的工具软件，一锅炖了，节省篇幅给主要内容。

```bash
# 网易云音乐
yay -S netease-cloud-music

# Tim（可能会出现问题）
yay -S deepin-wine-tim
# QQ（浪子没有安装测试，这个在 aur 上安装最多，可自行测试）
yay -S deepin-wine-qq

# 微信
yay -S deepin-wine-wechat
# 如果上面的安装失败，可以安装 wechat-uos 试试
yay -S wechat-uos
# electronic 版微信
yay -S electronic-wechat-uos-bin

# 腾讯会议
yay -S wemeet-bin

# 安装 WPS、WPS缺失字体、WPS中文语言包
yay -S wps-office-cn ttf-wps-fonts wps-office-mui-zh-cn

# 安装向日葵
yay -S sunloginclient
# 启动向日葵（两个命令都要执行）
systemctl start runsunloginclient.service
sunloginclient
```

> 浪子还是推荐直接使用浏览器解决所有影音需求，看个人喜好喽～
> Tim、QQ、微信 看你的系统可不可以运行吧，因为机器有差异，能否运行看运气。使用虚拟机安装体验较好（但是我使用 Linux 为什么还要安装虚拟机跑？）。

## 5. 一些小工具

```bash
yay -S flameshot simplescreenrecorder peek
```

|安装包|说明|
|--|--|
|flameshot|火焰截图|
|simplescreenrecorder|录屏软件|
|peek|动图录制工具|

## 6. 开发工具

termius：和 Xshell、SecureCRT 属于同一类产品。有 Android、Linux、Win 多端，分为免费版和付费版。

```bash
# vscode、edge、termius、google
yay -S visual-studio-code-bin
yay -S microsoft-edge-stable-bin
yay -S termius
yay -S google-chrome
# jdk
yay -S jdk8-openjdk jdk17-openjdk
# 如果有多个版本，设置某个 JDK 版本为默认版本
sudo archlinux-java set java-11-openjdk
# docker
yay -S docker docker-compose
```

docker 安装完成后普通用户不能使用相关命令，需要进行一些修改：

```bash
# 创建 docker 用户组
sudo groupadd docker
# 将普通用户加入 docker 组中
sudo gpasswd -a $USER docker
# 更新 docker 组
newgrp docker
# 测试命令
docker ps
```

## 7. nodejs

```bash
# 安装
yay -S nodejs npm
# 设置淘宝源
npm config set registry https://registry.npmmirror.com
# 授权 npm 安装的访问权限（以下是默认路径，如果进行了修改改下路径即可）
sudo chmod 777 -R /usr/bin
sudo chmod 777 -R /usr/lib/node_modules/
```

## 8. VirtualBox

安装 virtualbox 之前需要先确定自己 Linux 系统的内核版本，打开系统设置，系统信息；或者直接输入 `uname -a` 查看系统的内核版本号，在安装的时候选择和自己系统内核版本相同的 module 即可。

```bash
yay -S virtualbox
```

## 9. 终端使用[zsh](https://wiki.archlinux.org/title/Zsh_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))或者git-shell

> 新版本的 xfce 现在也是默认使用 zsh 了，所以此项一般可以忽略。

```bash
# 检查是否安装
zsh
# 如果没有安装那就安装
yay -S zsh zsh-completions

# 使用 git 终端
# 查看可设置终端列表
chsh -l
# 设置终端路径
chsh -s <full-path-to-shell>

# 如果您使用的是 systemd-homed，请运行
homectl update --shell=<full-path-to-shell> user
```

> 其中，full-path-to-shell 是 `chsh -l` 给出的完整路径。
> 提示：chsh 用作参考。如果列表中不存在最近安装的 shell，则可以手动将其添加到此文件中:`/etc/shells`
> 如果您要卸载 zsh，那么请您先更改默认的 shell 之后再进行卸载程序安装包。
> Arch Wiki：https://wiki.archlinux.org/title/Command-line_shell#Changing_your_default_shell

## 数据库

MySQL 的话，直接使用 docker 吧，这么大众化的产品，对吧，自己随便玩玩就行。

### 1. MariaDB

```bash
yay -S mariadb
```

> 普通用户在 kde 桌面的 zsh 终端中执行启动命令可能会弹出验证特殊身份的面板，该密码不是 root 用户的密码。如果验证失败（失败三次会锁定10分钟），此时切换到 root（su - root） 用户下执行。
> 安装后不要直接启动，先执行以下命令：
> mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
> systemctl start mariadb.service
> mysql -u root -p (默认无密码，直接回车即可)
> 具体细节参考 [MariaDB Wiki](https://wiki.archlinux.org/title/MariaDB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85)

### 2. [PostgreSQL](https://wiki.archlinux.org/title/PostgreSQL_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85PostgreSQL)

```bash
yay -S postgresql
```

安装完成后，root 用户直接输入 `su - postgres`，普通用户输入 `sudo -i -u postgres`，以 [postgres@xxxxxx]$作为前置符号，然后启动服务：`systemctl start postgresql.service`即可。

提示： 如果创建一个与你的 Arch 用户 ($USER) 同名的数据库用户，并允许其访问 PostgreSQL 数据库的 shell，那么在使用 PostgreSQL 数据库 shell 的时候无需指定用户登录（这样做会比较方便）。
以 postgres 用户身份, 添加一个新的数据库用户使用 createuser 命令

```bash
createuser --interactive
```

输入要增加的角色名称: 登录 Arch 的用户名。以具备读写权限的用户身份，创建一个新的数据库,使用createdb 命令。从你的 shell (不是 以 postrgres 用户的身份)

```bash
createdb myDatabaseName
```
