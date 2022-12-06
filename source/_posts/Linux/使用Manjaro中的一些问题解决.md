---
title: 使用Manjaro中的一些问题解决
categories:
  - Linux
tags:
  - linux
  - manjaro
cover: 'https://fastly.jsdelivr.net/gh/prettywinter/dist/images/blogcover/arch.jpeg'
---

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
