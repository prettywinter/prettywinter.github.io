---
title: Hexo博客配置
categories: 配置
tags: [Hexo,博客]
---

某次操作中脑回路异常把 Github Pages 的库给删除了……

<!-- more -->

### 1. 使用hexo d

使用该命令需要安装插件：{% copy width:max npm install --save hexo-deployer-git %}

### 2. 使用yourname.github.io的其它分支部署pages

在设置这一项之前确保在 `blog/_config.yml` 中已设置为 pages 分支，然后在 Github 仓库中进行设置部署的分支。步骤：进入仓库 -> 选择 Settings -> 找到 Pages，如下图：

{% image https://raw.sevencdn.com/prettywinter/dist/main/images/doc/github_pages选择分支作为博客部署目录.png %}

设置完成后保存，重新部署即可发布到指定的分支。

### 3. 使用永久链接插件解决中文转义

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

一般不需要这个需求，RSS 就类似与关注、订阅功能。安装插件：
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

### 5. 引入[Mermaid](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)流程图

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

### 6. 集成Github Action自动化部署时，文章的创建、更新时间都变为自动部署的时间问题

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

### 7. 数学公式渲染

Hexo 默认的 markdown 渲染器是不支持公式渲染的，可以进行替换，可以使用的渲染器有很多，这里使用 hexo-renderer-markdown-it-plus 作为公式渲染器。

```bash
# 卸载默认的渲染器
npm uninstall hexo-renderer-marked
# 安装 hexo-renderer-markdown-it-plus
npm install hexo-renderer-markdown-it-plus --save
```

安装完成后，需要配置，一般在主题中有 katex 的设置项，开启即可。没有的话的可以根据 [README](https://github.com/CHENXCHEN/hexo-renderer-markdown-it-plus) 进行配置。
