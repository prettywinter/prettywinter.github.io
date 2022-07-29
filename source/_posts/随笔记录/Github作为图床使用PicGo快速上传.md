---
title: Github图床使用PicGo实现快捷上传
categories: 配置
tags: [Github, PicGo]
---

使用 Github 作为图床，并使用 PicGo 实现快捷上传。

<!-- more -->

## 一、PicGo是什么？在本机如何使用？

PicGo 是图片上传到其它存储平台的助手。它支持多种图床：weibo, qiniu, tcyun, upyun, github, aliyun, imgur and SM.MS。
我们在写文章的过程中，免不了要在文章内放置图片，如果实在第三方平台写文章（知乎，简书。CSDN等），这个问题就不需要考虑。但如果是自己搭建的博客，或者是自己编写 `md` 文档，那么图片的处理就比较麻烦了，截图、命名、存储等等都是问题。
其中，使用 Github 作为图床是免费的，其它类型的图床有一定的免费空间。本文介绍 PicGo 配置 Github 图床使用。

### 1. 创建 Github Token

登录Github，点击头像，依次选择 **settings -> Developer settings -> Personal access tokens**，选择 `Generate new token`，进入填写如下信息，填写完毕点击最后的绿色按钮 `Generate token` 保存。**注意：生成的 token 只会显示一次，一定要复制后保存下来。** 否则需要重新生成。

{% image https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20220516101750.png %}

### 2. 下载安装[PicGo](https://github.com/Molunerfinn/PicGo/releases)

安装完成启动 PicGo，点击图床设置，选择 **Github图床**，点击 **设为默认图床**，然后进行图床的配置：
{% image https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20220515164925.png %}

1. 设定仓库名：username/repo
2. 设定分支名：main
3. 设定Token：GitHub 平台创建的 token
4. 指定存储路径：指定仓库下的指定分支下的文件夹路径。简单理解就是把图片放在仓库的哪个目录下。
5. 设定自定义域名：图片成功上传后返回的链接，格式为 [自定义设定域名/指定存储路径/上传后的图片名称](https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20220515164925.png) 。浪子这里使用了 jsdelivr 的 CDN 加速链接。

配置完成后就可以在上传区选择需要上传的图片啦。比如打开资源管理器选择图片，或者直接拖动需要上传的图片到上传区区域，PicGO 就会把这些图片上传到指定的 Github 仓库的路径下。

## 二、使用VsCode搭配PicGo插件上传文档图片到Github图床

浪子经常使用 VsCode 写 markdown 文件，之前不经常在文章里放图片，后来放图片的时候感觉真麻烦。不过幸好，VsCode 里也有 PicGo 的插件供我们使用。单纯在 VsCode 中使用这个插件可以不用安装 PicGo。

### 1. VsCode安装PicGo插件

打开 VsCode 的扩展商店搜索 **picgo**，安装图中红框框中插件。

{% image https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20220515164011.png %}

安装完成后使用 `Ctrl + Shift + P` 快捷键，搜索打开 `settings.json` 文件，添写以下内容保存，这里面的内容和上面的配置一样。

```json settings.json
// 安装 PicGo 软件，使用 PicGo 插件只需要配置一个 configPath
// 本机安装了 PicGo 客户端第一次启动后会自动生成 json 文件
// "picgo.configPath": "C:\\Users\\你的用户名\\AppData\\Roaming\\picgo\\data.json",
// 上面的选项如果配置了，下面的无须配置。

// 只使用 VsCode PicGo 插件，需要配置以下信息
// 当前使用图床 可选：weibo, qiniu, tcyun, upyun, github, aliyun, imgur and SM.MS
"picgo.picBed.current": "github|weibo|qiniu|tcyun|upyun|aliyun|imgur|SM.MS",
"picgo.picBed.uploader": "github|weibo|qiniu|tcyun|upyun|aliyun|imgur|SM.MS",
// 设定分支
"picgo.picBed.github.branch": "main",
// 设定仓库
"picgo.picBed.github.repo": "",
// token
"picgo.picBed.github.token": "",
// 自定义返回的链接
"picgo.picBed.github.customUrl": "",
// 上传路径
"picgo.picBed.github.path": "", 
```

{% image https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20220516102624.png %}

### 3. 在VsCode中使用

OS|从剪切板上传|从资源管理器上传图像（打开文件夹选择上传）|从输入框上传图像|
|--|--|--|--|
Windows/Unix|Ctrl + Alt + U|Ctrl + Alt + E|Ctrl + Alt + O|
OsX|Cmd + Opt + U|Cmd + Opt + E|Cmd + Opt + O|
