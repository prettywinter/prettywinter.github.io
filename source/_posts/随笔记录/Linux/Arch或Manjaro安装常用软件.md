---
title: ArchLinux/Manjaro安装一些常用软件
categories: Linux
tags: [Arch, 配置, 软件安装]
cover: https://raw.githubusercontentS.com/prettywinter/dist/main/images/blogcover/arch.jpeg
abbrlink: 8ecfb81b
---

整理一些 ArchLinux 中常用的软件。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [1. AUR 助手 yay](#1-aur-助手-yay)
- [2. 中文输入法](#2-中文输入法)
  - [2.1 非Gnome桌面推荐 fcitx5](#21-非gnome桌面推荐-fcitx5)
  - [2.2 Gnome桌面推荐 ibus](#22-gnome桌面推荐-ibus)
- [3. 字体](#3-字体)
- [4. 网易云音乐](#4-网易云音乐)
- [5. 火焰截图](#5-火焰截图)
- [6. 谷歌浏览器](#6-谷歌浏览器)
- [7. Tim、微信](#7-tim微信)
- [8. 录屏软件](#8-录屏软件)
- [9. 动图录制工具](#9-动图录制工具)
- [10. WPS](#10-wps)
- [11. VSCode](#11-vscode)
- [12. Edge](#12-edge)
- [13. MariaDB](#13-mariadb)
- [14. PostgreSQL#%E5%AE%89%E8%A3%85PostgreSQL)](#14-postgresql)
- [15. VirtualBox](#15-virtualbox)
- [16. termius for linux(微软)](#16-termius-for-linux微软)
- [开发工具](#开发工具)
- [中文环境改变的问题](#中文环境改变的问题)

<!-- /code_chunk_output -->

Arch 安装软件可以去官网搜索，然后安装。
{% link Arch WIKI::https://wiki.archlinux.org/ %}

## 1. AUR 助手 yay

yay 是使用 AUR 的命令，它类似于 CentOS 的 yum，也是 Arch/Manjaro 系统中特有的包管理工具；而 AUR 就类似于第三方维护的软件库，这个也是最吸引人的使用的地方，可以安装WeChat、Tim等 Win 平台下的软件，有专人维护。浪子是非常推荐使用 Arch/Manjaro 的少侠使用这个命令的，pacman 支持的命令 yay 都支持，并且更为丰富。

{% link 详细了解::https://zhuanlan.zhihu.com/p/363666022 %}

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
# 清除缓存
yay -Sc
# 查看安装的软件
yay -Ps
```

## 2. 中文输入法

### 2.1 非Gnome桌面推荐 fcitx5

```bash{.line-numbers}
yay -S fcitx5 fcitx5-chinese-addons fcitx5-gtk fcitx5-qt kcm-fcitx5 fcitx5-material-color fcitx5-lua
```

编辑 `~/.pam_environment` 文件，没有这个文件的话可以手动创建，然后添加以下内容：

```bash{.line-numbers}
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=@im=fcitx
INPUT_METHOD  DEFAULT=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```

### 2.2 Gnome桌面推荐 ibus

```bash{.line-numbers}
sudo pacman -S ibus-rime
```

在 Gnome 的 Manjaro 上 ibus 只需要安装即可，无需配置文件。

如果是 **非Gnome用户** 使用ibus输入法，**需要** 编辑 `~/.xprofile` 文件，添加以下内容：

```bash{.line-numbers}
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -x -d
```

以上文件设置完成后可以使用 source 命令刷新设置，或者重启系统。
{% link 输入法设置参考::https://linuxacme.cn/559 %}

## 3. 字体

```bash{.line-numbers}
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

```bash{.line-numbers}
yay -S netease-cloud-music
```

> 浪子还是推荐直接浏览器解决所有影音需求，看个人喜好喽～

## 5. 火焰截图

```bash{.line-numbers}
yay -S flameshot
```

## 6. 谷歌浏览器

```bash{.line-numbers}
yay -S google-chrome
```

## 7. Tim、微信

```bash{.line-numbers}
# 微信
yay -S deepin-wine-wechat 
# Tim（可能会出现问题）
yay -S deepin-wine-tim
# QQ（浪子没有安装测试，这个在 aur 上安装最多，可自行测试）
yay -S deepin-wine-qq
```

> 不推荐在 Linux 下直接安装，使用虚拟机安装体验较好。

## 8. 录屏软件

```bash{.line-numbers}
yay -S vokoscreen
```

## 9. 动图录制工具

```bash{.line-numbers}
yay -S peek
```

## 10. WPS

```bash
yay -S wps-office-cn
# 安装缺失字体
yay -S ttf-wps-fonts
# 安装中文语言包
yay -S wps-office-mui-zh-cn
```

## 11. VSCode

```bash
yay -S visual-studio-code-bin
```

## 12. Edge

```bash
# 安装微软的 Edge 浏览器，可能下载速度有些慢
yay -S microsoft-edge-stable-bin 
```

## 13. MariaDB

```bash
yay -S mariadb
```

> 普通用户在 kde 桌面的 zsh 终端中执行启动命令可能会弹出验证特殊身份的面板，该密码不是 root 用户的密码。如果验证失败（失败三次会锁定10分钟），执行启动命令推荐切换到 root（su - root） 用户下执行。
> 安装后不要直接启动，先执行以下命令：
> mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
> systemctl start mariadb.service
> mysql -u root -p (默认无密码，直接回车即可)
> 具体细节参考 [MariaDB Wiki](https://wiki.archlinux.org/title/MariaDB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85)

## 14. [PostgreSQL](https://wiki.archlinux.org/title/PostgreSQL_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85PostgreSQL)

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

## 15. VirtualBox

安装 virtualbox 之前需要先确定自己 Linux 系统的内核版本，打开系统设置，系统信息；或者直接输入 `uname -a` 查看系统的内核版本号，在安装的时候选择和自己系统内核版本相同的 module 即可。

```bash
yay -S virtualbox
```

## 16. termius for linux(微软)

简单的说，是和 Xshell、SecureCRT 属于同一类产品。有 Android、Linux、Win 多端。

```bash
yay -S termius
```

## 开发工具

```bash{.line-numbers}
yay -S jdk11-openjdk docker
# 如果有多个版本，设置JDK11为默认，此命令仅 Arch 生效
# sudo archlinux-java set java-11-openjdk
```

## 中文环境改变的问题

{% link 解决办法参考Arch Wiki::https://wiki.archlinux.org/title/Locale_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87) %}

如果是 KDE Plasma 桌面环境，默认你使用了 UTF-8 的编码，通过以下几步后重启就可以看到更改过来了：

```bash{.line-numbers}
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
