---
title: GithubPages+Hexo个人博客完善
categories: 教程
tags: [Github Pages, Hexo, 博客]
abbrlink: 61d2e814
---

某次操作中脑回路异常把 Github Pages 的库给删除了……

<!-- more -->

### 1. 使用 hexo d

使用该命令需要安装插件：`npm install --save hexo-deployer-git`

### 2. 使用yourname.github.io的其它分支部署pages

在设置这一项时需要在 Github 仓库中进行设置要部署的分支才可以，不然部署之后访问会看不到页面。
步骤：进入仓库 -> 选择 Settings -> 找到 Pages，如下图：

![assign branch](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/github_pages选择分支作为博客部署目录.png)

设置完成后保存，重新部署即可发布到指定的分支。

### 3. 使用永久链接插件解决中文转义

如果我们写的文章标题是中文的话，那么在进行链接的分享时是会进行转义的（**~~不是乱码~~**），至于为什么要转义，请自行百度。如果忍受不了可以使用插件去解决，生成一个永久性的链接。它会让你的博客链接变成类似这样：`https://xxx.xxx.com/posts/8ddf18fb.html`。注意这款插件并不会让你的中文正常显示，而是生成一串简短的字符串，毕竟中文url在转义后特别长。

安装插件：`npm install hexo-abbrlink --save`

在博客目录的配置文件 `blog/_config.yml` 中找到对应 `permalink` 标签，修改即可。下面是示例：

  ```yml{.line-numbers}
# permalink: :year/:month/:day/:title/
permalink: p/:abbrlink/
abbrlink: 
  alg: crc32  #算法： crc16(default) and crc32
  rep: hex    #进制： dec(default) and hex
permalink_defaults:
pretty_urls:
  trailing_index: false # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
  ```

不同算法、进制生成的链接格式如下：
|算法|进制|生成链接示例|
|--|--|--|
|crc16|hex|https://yourname.github.io/p/66c8.html|
|crc16|dec|https://yourname.github.io/p/65535.html|
|crc32|hex|https://yourname.github.io/p/8ddf18fb.html|
|crc32|dec|https://yourname.github.io/p/1690090958.html|

### 4. 生成 RSS 页面

一般都不需要这个需求，就不讲解了。RSS 就类似与关注、订阅，博客有了更新可以通知并发送给你。
安装插件：`npm install -save hexo-generator-feed`

配置示例（依然是在博客的配置文件中添加，安装完插件可以直接添加到最后）：

```yml{.line-numbers}
# In the front-matter of your post,
# you can optionally add a description, intro or excerpt setting to write a summary for the post.
# Otherwise the summary will default to the excerpt or the first 140 characters of the post.
feed:
  enable: true
  type: 
    - atom
    - rss2
  path: 
    - atom.xml
    - rss2.xml
  limit: 20
  hub:
  content: false
  content_limit: 140
  content_limit_delim: ' '
  order_by: -date
  icon: icon.png
  autodiscovery: true
```

配置完成执行 `hexo cl && hexo g && hexo d`，然后访问 `https://yourname.github.io/rss2.xml` 或者 `https://yourname.github.io/atom.xml` 查看生成的xml。

### 5. 引入[Mermaid](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)流程图

Hexo 默认的 markdown 不支持 Mermaid 流程图的，需要安装依赖来支持：

```bash
npm install --save hexo-filter-mermaid-diagrams
```

在 `_config.yml` 文件中加入以下内容：

```yml{.line-numbers}
mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "7.1.2" # default v7.1.2
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
    #startOnload: true  // default true
```

之后，还需要在主题文件中加入一些代码，具体的配置可以参考 [官网](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)。

如果实在不想配置，或者嫌弃配置麻烦，可以使用下面的一种方法。

可以采取另一种解决方案：[hexo-filter-kroki](https://github.com/miao1007/hexo-filter-kroki)，这个是使用 [Kroki.io](https://kroki.io/) 去生成图，然后把图片返回到你的博客中。

官方解释：安装完成后，默认情况下，不需要进行任何配置，插件会将您的文本发送到 kroki.io 进行渲染，并且 base64 编码的图像内联在 html 中。在作者的仓库中也有高级配置，可以自行查看配置。

```bash
# 懒人推荐
npm install --save hexo-filter-kroki
```

### 6. 集成Github Action自动化部署时，文章的创建、更新时间都变为自动部署的时间问题

1. 安装插件：`npm install hexo-filter-date-from-git --save`
2. 确保所有的 md 文件名称中不含空格
3. Github Action 的脚本中加入 `fetch-depth: 0`：

```yml
# Fetch all history for all tags and branches
steps:
- name: Checkout
  uses: actions/checkout@v3
  with:
    fetch-depth: 0
```
