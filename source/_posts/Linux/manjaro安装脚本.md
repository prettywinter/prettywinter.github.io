---
title: Manjaro安装脚本
categories:
  - Linux
tags:
  - linux
  - manjaro脚本
abbrlink: 5b9e7c7c
---

每次重装 manjaro 后都需要安装一些环境，刚开始还好，后面就变得懒了，写一个脚本帮助完成安装。

<!-- more -->

## 安装脚本

下面的安装脚本是针对 Manjaro-KDE 版本，如果各位少侠是使用的 Gnome 的话，输入法不推荐 fcitx5，推荐使用 Ibus。下面的安装工具可以在我的另一篇文章中找到：

```sh
#!/bin/bash
# yay，下面的所有命令都使用 yay 安装
echo "install yay"
yes yes | sudo pacman -S yay
echo "update system..."
yes yes | yay -Syu
echo "install yay and update system done!"

# fcitx5 输入法
echo "install fcitx5..."
yes yes | yay -S fcitx5-im fcitx5-rime fcitx5-chinese-addons
echo "install fcitx5 done!"
# 输入法写入配置
sudo echo -e "GTK_IM_MODULE=fcitx\nQT_IM_MODULE=fcitx\nXMODIFIERS=@im=fcitx\nSDL_IM_MODULE=fcitx\nGLFW_IM_MODULE=ibus" >> /etc/environment

echo "print /etc/environment content"
sudo cat /etc/environment
echo "print /etc/environment content done!"

# 字体
echo "install fonts..."
yes yes | yay -S wqy-bitmapfont wqy-microhei wqy-zenhei wqy-microhei-lite ttf-fira-code woff2-fira-code woff-fira-code ttf-jetbrains-mono
echo "install fonts done!"

# linux 下好用的工具
echo "install utils..."
yes yes | yay -S vim flameshot simplescreenrecorder
echo "install utils done!"

# window 下常用的工具
echo "install window some utils..."
yes yes | yay -S wemeet-bin electronic-wechat-uos-bin
echo "install window some utils done!"

############################### 开发环境 #################################
echo "devlopment environment begin..."
# 开发相关软件 vscode、termius、edge、goole 浏览器
echo "install some IDE..."
yes yes | yay -S visual-studio-code-bin microsoft-edge-stable-bin termius google-chrome
echo "install IDE done!"

# jdk
echo "install jdk..."
yes yes | yay -S jdk8-openjdk jdk17-openjdk
# 如果有多个版本，设置某个 JDK 版本为默认版本
sudo archlinux-java set java-8-openjdk
java -version
echo "install jdk done!"

# docker
echo "install docker..."
yes yes | yay -S docker docker-compose
# 创建 docker 用户组
sudo groupadd docker
# 将普通用户加入 docker 组中
sudo gpasswd -a $USER docker
# 更新 docker 组
newgrp docker
# 测试命令
docker ps
echo "install docker done!"

# 配置 git，需要修改为自己的信息
echo "git config..."
git config --global user.name "username"
git config --global user.email "email"
echo "git config setting done!"

echo "devlopment environment end!"
```

> 上面的脚本中 Git 的信息配置需要手动修改为自己的信息。

## 运行

创建 `install.sh` 文件，复制并修改上面的内容，然后在 bash 中运行以下命令：

```bash
sh install.sh 2>&1 | tee install.log
```

如果中途有安装失败的情况，可以查看 `install.log` 文件定位错误信息。
