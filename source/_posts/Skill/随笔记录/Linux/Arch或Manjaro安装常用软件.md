---
title: ArchLinux/Manjaro安装一些常用软件
categories: Linux
tags: [Arch, 配置, 软件安装]
cover: https://raw.githubusercontentS.com/prettywinter/dist/main/images/blogcover/arch.jpeg
---

整理一些 ArchLinux 中常用的软件。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [1. AUR 助手 yay](#1-aur-助手-yay)
- [2. 中文输入法](#2-中文输入法)
  - [2.1 非Gnome桌面推荐 fcitx5](#21-非gnome桌面推荐-fcitx5)
  - [2.2 Gnome桌面推荐 ibus)](#22-gnome桌面推荐-ibus)
- [3. 字体](#3-字体)
- [4. 网易云音乐](#4-网易云音乐)
- [5. Tim、微信](#5-tim微信)
- [6. 小工具](#6-小工具)
- [7. WPS](#7-wps)
- [8. vscode、edge、termius、google](#8-vscodeedgetermiusgoogle)
- [9. IDEA社区版](#9-idea社区版)
- [10. MariaDB](#10-mariadb)
- [11. PostgreSQL#%E5%AE%89%E8%A3%85PostgreSQL)](#11-postgresql)
- [12. VirtualBox](#12-virtualbox)
- [zsh)或者git-shell](#zsh或者git-shell)
- [开发工具](#开发工具)
- [中文环境改变的问题](#中文环境改变的问题)

<!-- /code_chunk_output -->

Arch 安装软件可以去官网搜索，然后安装。
{% link https://wiki.archlinux.org/ Arch WIKI %}

## 1. AUR 助手 yay

yay 是使用 AUR 的命令，它类似于 CentOS 的 yum，也是 Arch/Manjaro 系统中特有的包管理工具；而 AUR 就类似于第三方维护的软件库，这个也是最吸引人的使用的地方，可以安装WeChat、Tim等 Win 平台下的软件，有专人维护。浪子是非常推荐使用 Arch/Manjaro 的少侠使用这个命令的，pacman 支持的命令 yay 都支持，并且更为丰富。

{% link https://zhuanlan.zhihu.com/p/363666022 详细了解 %}

yay 常用命令：

```bash{.line-numbers}
# 安装 yay
sudo pacman -S yay

# 一些常用的命令
# 更新软件库
yay -Syu
# 安装
yay -S <package-name>
# 更新指定包软件
yay -u package-name
# 卸载该软件和没有被使用的依赖
yay -Rs <package-name>
# 若要删除当前未安装的所有缓存包和未使用的同步数据库
yay -Sc
# 查找符合条件的安装包
yay -Ss <package-name>
# 查看安装的软件
yay -Ps
```

## 2. 中文输入法

### 2.1 非Gnome桌面推荐 fcitx5

```bash
yay -S fcitx5 fcitx5-chinese-addons fcitx5-gtk fcitx5-qt kcm-fcitx5 fcitx5-material-color fcitx5-lua
```

编辑 `~/.pam_environment` 文件，没有这个文件的话可以手动创建，然后添加以下内容：

```bash
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=@im=fcitx
INPUT_METHOD  DEFAULT=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```

### 2.2 Gnome桌面推荐 [ibus](https://wiki.archlinux.org/title/IBus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

```bash
sudo pacman -S ibus ibus-libpinyin
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

## 4. 网易云音乐

```bash
yay -S netease-cloud-music
```

> 浪子还是推荐直接浏览器解决所有影音需求，看个人喜好喽～

## 5. Tim、微信

```bash
# 微信
yay -S deepin-wine-wechat 
# Tim（可能会出现问题）
yay -S deepin-wine-tim
# QQ（浪子没有安装测试，这个在 aur 上安装最多，可自行测试）
yay -S deepin-wine-qq
```

> 不推荐在 Linux 下直接安装，使用虚拟机安装体验较好。

## 6. 小工具

```bash
yay -S flameshot vokoscreen peek
```

|安装包|说明|
|--|--|
|flameshot|火焰截图|
|vokoscreen|录屏软件|
|peek|动图录制工具|

## 7. WPS

```bash
# 安装 WPS、WPS缺失字体、WPS中文语言包
yay -S wps-office-cn ttf-wps-fonts wps-office-mui-zh-cn
```

## 8. vscode、edge、termius、google

termius：和 Xshell、SecureCRT 属于同一类产品。有 Android、Linux、Win 多端，分为免费版和付费版。

```bash
yay -S visual-studio-code-bin microsoft-edge-stable-bin termius google-chrome
```

## 9. IDEA社区版

```bash
yay -S intellij-idea-community-edition
```

## 10. MariaDB

```bash
yay -S mariadb
```

> 普通用户在 kde 桌面的 zsh 终端中执行启动命令可能会弹出验证特殊身份的面板，该密码不是 root 用户的密码。如果验证失败（失败三次会锁定10分钟），执行启动命令推荐切换到 root（su - root） 用户下执行。
> 安装后不要直接启动，先执行以下命令：
> mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
> systemctl start mariadb.service
> mysql -u root -p (默认无密码，直接回车即可)
> 具体细节参考 [MariaDB Wiki](https://wiki.archlinux.org/title/MariaDB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85)

## 11. [PostgreSQL](https://wiki.archlinux.org/title/PostgreSQL_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85PostgreSQL)

```bash
yay -S postgresql
```

安装完成后，root 用户直接输入 `su - postgres`，普通用户输入 `sudo -i -u postgres`，以 [postgres@xxxxxx]$作为前置符号，然后启动服务：`systemctl start postgresql.service`即可。

提示： 如果创建一个与你的 Arch 用户 ($USER) 同名的数据库用户，并允许其访问 PostgreSQL 数据库的 shell，那么在使用 PostgreSQL 数据库 shell 的时候无需指定用户登录（这样做会比较方便）。
以 postgres 用户身份, 添加一个新的数据库用户使用 createuser 命令

```bash
[postgres]$ createuser --interactive
```

输入要增加的角色名称: 登录 Arch 的用户名。以具备读写权限的用户身份，创建一个新的数据库,使用createdb 命令。从你的 shell (不是 以 postrgres 用户的身份)

```bash
createdb myDatabaseName
```

## 12. VirtualBox

安装 virtualbox 之前需要先确定自己 Linux 系统的内核版本，打开系统设置，系统信息；或者直接输入 `uname -a` 查看系统的内核版本号，在安装的时候选择和自己系统内核版本相同的 module 即可。

```bash
yay -S virtualbox
```

## [zsh](https://wiki.archlinux.org/title/Zsh_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))或者git-shell

```bash
# 检查是否安装
zsh
# 安装
yay -S zsh zsh-completions

# 使用 git 终端
# 查看可设置终端列表
chsh -l
# 设置终端路径
chsh -s <full-path-to-shell>
# 如果您使用的是 systemd-homed，请运行
homectl update --shell=full-path-to-shell user
```

> 其中，full-path-to-shell 是 `chsh -l` 给出的完整路径。
> 提示：chsh 用作参考。如果列表中不存在最近安装的 shell，则可以手动将其添加到此文件中:`/etc/shells`
> 如果您要卸载 zsh，那么请您先更改默认的 shell 之后再进行卸载程序安装包。
> Arch Wiki：https://wiki.archlinux.org/title/Command-line_shell#Changing_your_default_shell

## 开发工具

```bash
yay -S jdk8-openjdk jdk17-openjdk docker 	docker-compose
# 如果有多个版本，设置JDK11为默认
sudo archlinux-java set java-11-openjdk
```

## 中文环境改变的问题

{% link https://wiki.archlinux.org/title/Locale_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87) ArchWiki %}

如果是 KDE Plasma 桌面环境，默认你使用了 UTF-8 的编码，通过以下几步后重启就可以看到更改过来了：

```bash
# 删除文件
rm -rf ~/.config/plasma-localerc
# 编辑 /etc/profile 文件，加入以下内容
export LC_ALL='zh_CN-UTF8'
# 添加完成保存退出，并更新设置使其生效
source /etc/profile
# 显示正在使用的 Locale 和相关的环境变量
locale
```

这里只说了 KDE 桌面环境的解决办法，其它的环境在上面的链接里也是有的。如果各位少侠发现自己系统路径下没有 `/etc/locale.conf` 和 `/etc/locale.gen` 文件也不用慌，使用 `locale-gen` 命令后会自动生成，我们只需要编辑这些文件去修改为需要的语言环境就OK。

随时补充……
