---
wiki: Git
layout: wiki
title: GitHub+PicGo搭建图床
order: 6
---

## 一、PicGo是什么？在本机如何使用？

PicGo 是图片上传到其它存储平台的助手。它支持多种图床：weibo, qiniu, tcyun, upyun, github, aliyun, imgur and SM.MS。

我们在写文章的过程中，免不了要在文章内放置图片，对于自己搭建的博客网站，或者是自己编写 `md` 文档，那么图片的处理就比较麻烦了，截图、命名、存储等都是问题。

## 二、本机安装 [PicGo](https://github.com/Molunerfinn/PicGo/releases)

安装完成启动 PicGo 客户端，点击图床设置，选择 **Github图床**，点击 **设为默认图床**，然后进行图床的配置：

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20220515164925.png %}

1. 设定仓库名：username/repo
2. 设定分支名：main
3. 设定Token：GitHub 平台创建的 token
4. 指定存储路径：指定仓库下的指定分支下的文件夹路径。简单理解就是把图片放在仓库的哪个目录下。
5. 设定自定义域名：图片成功上传后返回的链接，格式为 [自定义设定域名/指定存储路径/上传后的图片名称](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20220515164925.png) 。浪子这里使用了 jsdelivr 的 CDN 加速链接。

配置完成后就可以在上传区选择需要上传的图片啦。比如打开资源管理器选择图片，或者直接拖动需要上传的图片到上传区区域，PicGO 就会把这些图片上传到指定的 Github 仓库的路径下。

## 三、配置 Github

### 1. 创建一个存放图片的仓库

### 2. 建立一个目标分支，默认是 main

### 3. 创建 Github Token

登录Github，点击头像，依次选择 **settings -> Developer settings -> Personal access tokens**，选择 `Generate new token`，进入填写如下信息，填写完毕点击最后的绿色按钮 `Generate token` 保存。**注意：生成的 token 只会显示一次，一定要复制后保存下来。** 否则需要重新生成。

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20220516101750.png %}

上面的步骤都完成之后把相应的信息填写到 PicGo 中，然后就可以使用该软件进行图片上传啦~

## 四、VSCode+PicGo插件

浪子经常使用 VsCode 写 markdown 文件，之前不经常在文章里放图片，后来放图片的时候感觉真麻烦。不过幸好，VsCode 里也有 PicGo 的插件供我们使用。单纯在 VsCode 中使用这个插件不需要安装 PicGo 客户端。

### 1. VsCode安装PicGo插件

打开 VsCode 的扩展商店搜索 **picgo**，安装图中红框框中插件。

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20220515164011.png %}

安装完成后使用 `Ctrl + Shift + P` 快捷键，搜索打开 `settings.json` 文件，添写以下内容保存，这里面的内容和上面的配置类似。

```json settings.json
// 如果在本机安装了 PicGo 客户端，第一次启动后会自动生成 json 文件，然后你只需要配置这一项就 OK 啦
// "picgo.configPath": "C:\\Users\\你的用户名\\AppData\\Roaming\\picgo\\data.json",


// 如果本机没有安装 PicGo 客户端，只安装了 PicGo 插件，需要配置以下信息

// 当前使用图床 可选：weibo, qiniu, tcyun, upyun, github, aliyun, imgur and SM.MS
"picgo.picBed.current": "github|weibo|qiniu|tcyun|upyun|aliyun|imgur|SM.MS",
"picgo.picBed.uploader": "github|weibo|qiniu|tcyun|upyun|aliyun|imgur|SM.MS",
// 设定分支
"picgo.picBed.github.branch": "branch",
// 设定仓库
"picgo.picBed.github.repo": "repo",
// token
"picgo.picBed.github.token": "token",
// 自定义返回的链接
"picgo.picBed.github.customUrl": "return-url",
// 上传路径
"picgo.picBed.github.path": "path", 
```

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20220516102624.png %}

### 2. 在VsCode中使用快捷键

| OS           | 从剪切板上传   | 从资源管理器上传图像（打开文件夹选择上传） | 从输入框上传图像 |
| ------------ | -------------- | ------------------------------------------ | ---------------- |
| Windows/Unix | Ctrl + Alt + U | Ctrl + Alt + E                             | Ctrl + Alt + O   |
| OsX          | Cmd + Opt + U  | Cmd + Opt + E                              | Cmd + Opt + O    |
