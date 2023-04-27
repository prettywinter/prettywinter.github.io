---
title: 无法访问Github解决办法
categories: GitHub
tags: github
abbrlink: '73e96015'
---

在三、四、五、六线城市以及像浪子所在的偏远小山村中想要直接使用浏览器访问 GitHub 是有问题的，输入网址后按下回车键，这个页面都不带刷新的，直接出现 “无法访问该网站” 的错误。解决办法之一就是修改 hosts 文件。

<!-- more -->

hosts 文件不同的操作系统位置不同，如果是 window 系统，他在 `C:\Windows\System32\drivers\etc` 目录下；如果是 Linux 系统，则在 `/etc/hosts`。这里推荐火绒，可以直接打开火绒，点击修改 hosts 文件，方便好用。注意先对 hosts 文件备份，防止修改错误后可以还原。

首先访问 [IPAddress.com](https://www.ipaddress.com/)，右上角输入 github.com 回车查询。将获取到的网址与 github.com 对应写入到 hosts 文件中。这样基本就可以了。

hosts 文件保存最好先以管理员身份启动记事本后再打开，防止权限不足。
