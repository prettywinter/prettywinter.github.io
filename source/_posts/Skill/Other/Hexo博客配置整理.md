---
title: Hexo博客配置整理
categories:
  - Hexo
tags:
  - hexo
  - 配置
abbrlink: 9b42c073
---

使用 Hexo + Github/其他平台提供的 Pages 服务打造自己简单的博客的话，还是需要安装一些插件来辅助的，不仅可以美化排版，还能提升用户体验。一起看看有哪些用着不错的插件吧~

<!-- more -->

## 一、插件

{% link https://hexo.io/docs/one-command-deployment hexo官网部署插件 %}

其它推荐：https://loafing.cn/posts/hexo-tags.html

### 1. hexo d

使用该命令需要安装插件：{% copy npm install --save hexo-deployer-git %}

### 2. 使用永久链接插件解决中文转义

如果我们写的文章标题是中文的话，那么在进行链接的分享时是会进行转义的（**~~不是乱码~~**），至于为什么要转义，请自行百度。如果忍受不了可以使用插件去解决，生成一个永久性的链接。它会让你的博客链接变成类似这样：`https://xxx.xxx.com/posts/8ddf18fb.html`。注意：这款插件并不会让你的中文正常显示，而是生成一串简短的字符串，毕竟中文url在转义后特别长。

安装插件：{% copy npm install hexo-abbrlink --save %}

在 `blog/_config.yml` 中找到对应 `permalink` 标签，进行修改即可：

```yml blog/_config.yml
# permalink: :year/:month/:day/:title/
permalink: p/:abbrlink/
abbrlink: 
alg: crc32  #算法： crc16(default) and crc32
rep: hex    #进制： dec(default) and hex
permalink_defaults:
pretty_urls:
trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
trailing_html: true # Set to false to remove trailing '.html' from permalinks
```

不同算法、进制生成的链接格式如下：
|算法|进制|生成链接示例|
|--|--|--|
|crc16|hex|https://yourname.github.io/p/66c8.html|
|crc16|dec|https://yourname.github.io/p/65535.html|
|crc32|hex|https://yourname.github.io/p/8ddf18fb.html|
|crc32|dec|https://yourname.github.io/p/1690090958.html|

### 3. 生成RSS页面

{% copy npm install -save hexo-generator-feed %}

配置示例：

```yml blog/_config.yml
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

配置完成执行 `hexo cl && hexo g && hexo d`，然后访问 `https://yourname.github.io/rss2.xml` 或者 `https://yourname.github.io/atom.xml` 查看。

### 4. 引入[Mermaid](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)流程图

Hexo 默认的 markdown 不支持 Mermaid 流程图，需要安装依赖来支持：

{% copy width:max npm install --save hexo-filter-mermaid-diagrams %}

在 `_config.yml` 文件中加入以下内容：

```yml blog/_config.yml
# mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "7.1.2" # default v7.1.2
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
    #startOnload: true  // default true
```

之后，还需要在主题文件中加入一些代码，具体的配置可以参考 [官网](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)。如果嫌弃配置麻烦或者实在不想配置，可以使用下面的方法。

另一种解决方案：[hexo-filter-kroki](https://github.com/miao1007/hexo-filter-kroki)，安装完成后，默认情况下，不需要进行任何配置，插件会将您的文本发送到 [Kroki.io](https://kroki.io/) 进行渲染，并将 base64 编码的图像内联在 html 中。在插件作者的仓库中也有高级配置，可以自行查看配置。

```bash
# 懒人推荐
npm install --save hexo-filter-kroki
```

### 5. 集成Github Action自动化部署时，文章的创建、更新时间都变为自动部署的时间问题

1. 安装插件：{% copy npm install hexo-filter-date-from-git --save %}
2. 确保所有的 md 文件名称中不含空格
3. Github Action 的脚本中加入 `fetch-depth: 0`：

```yml .github/workflows/xxx.yml
# Fetch all history for all tags and branches
steps:
- name: Checkout
  uses: actions/checkout@v3
  with:
    fetch-depth: 0
```

## 二、相关问题

### 1. 关于配置文件

`blog/_config.yml` 是使用 `hexo init` 创建项目时，文件夹中的 `_config.yml` 文件。它是整个博客的配置文件，部署仓库的地址、分支等信息在这里配置。
`theme/_config.yml` 是使用 `hexo init` 创建项目时，文件夹中的 `_config.主题名称.yml` 文件（默认的主题名称为 lan）。主题名称为博客文件夹中 `theme/主题文件夹` 的名称。它是博客主题的配置文件，主题支持的渲染模型、文章使用的标签排版等都通过这里配置。

### 2. 在yourname.github.io的其它分支部署

在设置这一项之前确保在 `blog/_config.yml` 中已设置部署的目标分支，然后在 Github 仓库中进行设置。步骤：进入仓库 -> 选择 Settings -> 找到 Pages，如下图：

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/github_pages选择分支作为博客部署目录.png %}

设置完成后保存，重新部署即可发布到指定的分支。

### 3. 部署之后页面白屏

部署之后，打开网址页面白屏，F12 打开控制台如果有页面异常的消息，代表配置（比如主题配置文件加载博客配置等等多种原因）有问题，执行命令 `hexo g --debug` 查看报错信息，修改对应的配置项，修改完之后继续执行 `hexo g --debug`，直到没有报错信息，然后进行部署，等待部署完成访问网站就 OK 啦~
