---
title: Git入门
categories: 配置
tags: [Git]
cover: https://raw.sevencdn.com/prettywinter/dist/main/images/doc/git_command.jpg
---

Git 入门总结。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=true} -->

<!-- code_chunk_output -->

- [Git配置SSH密钥](#git配置ssh密钥)
  - [1. 生成Github用的SSH-Key](#1-生成github用的ssh-key)
  - [2. 生成gitee使用的SSH-Key](#2-生成gitee使用的ssh-key)
  - [3. 配置](#3-配置)
  - [4. 测试是否配置成功](#4-测试是否配置成功)
- [Git常用命令](#git常用命令)
  - [1. 配置命令](#1-配置命令)
  - [2. 远程操作](#2-远程操作)
  - [3. 常用操作](#3-常用操作)
- [一张图](#一张图)
- [Git Submodule](#git-submodule)
  - [删除Submodule](#删除submodule)

<!-- /code_chunk_output -->

有关Git的操作、设置在官方文档中都有说明，可以直接查阅：
{% link https://git-scm.com/book/zh/v2 Git官方文档 %}

## Git配置SSH密钥

Git的官网下载比较慢，可以去 [这里](https://npm.taobao.org/mirrors/git-for-windows/) 选择合适的版本进行下载。

为什么配置SSH密钥？
如果不配置 SSH 密钥的话，在你每次pull、push的时候需要身份认证，这样每次都需要我们手动去输入账户信息，比较耗时。而使用 SSH 的话就省略了输入账户信息的步骤，直接在本地和远程库之间进行操作，用户信息的认证工作交给了 SSH 帮我们进行。

SSH-Key 主要用来和开源代码托管平台、公司内部、个人搭建的存储平台等进行身份验证服务，同时也加密了传输的数据。下面是生成多个密钥的配置示例。

### 1. 生成Github用的SSH-Key

```shell
# Linux Mac
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f ~/.ssh/github_id_rsa
# window
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f C:\Users\用户名\.ssh\github_id_rsa
```

### 2. 生成gitee使用的SSH-Key

```bash
# Linux Mac
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f ~/.ssh/gitee_id_rsa
# window
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f C:\Users\用户名\.ssh\gitee_id_rsa
```

> 以上生成密钥的示例中，在 Git Bash 中使用 `~/.ssh/xxx_id_rsa` 路径就可以；在 Win 系统中如果使用 cmd/power shell/cmder等工具，使用该路径可能会报错“没有该路径文件”，这时候可以使用第二种方式。在下面 config 配置文件中指定路径时同理，一般同 Linux 就可以。
> 2022-08-22 修改为 ed25519 密钥生成。

### 3. 配置

在大多数系统中，默认私钥（~/.ssh/id_rsa 和 ~/.ssh/identity）会自动添加到 SSH 身份验证代理中。 应无需运行 ssh-add path/to/key，除非在生成密钥时覆盖文件名。

在 Linux 系统中比较容易出现 [Permission denied (publickey)](https://docs.github.com/cn/authentication/troubleshooting-ssh/error-permission-denied-publickey) 的错误，这就需要使用 `ssh-add` 把签名的指纹加入到 SSH 的 session 中。但是也会出现问题，综合以上内容，浪子推荐在本地配置 `config` 文件，我们手动创建文件并添加如下内容（其中Host和HostName填写git服务器的域名，IdentityFile指定私钥的路径）

```bash
# github1
# Host 可以自己定义，在此文件中必须唯一，之后在 down 源码时需要使用这个自定义的 host
Host github.com
# HostName 是代码托管平台的域名，例如：code.aliyun.com, github.com, gitee.come 等
HostName github.com
# 设置用户身份认证方式，这里当然使用 公钥 的方式
PreferredAuthentications publickey
# 指定公钥对应的私钥路径
IdentityFile ~/.ssh/github_id_rsa

# github2
Host xyzgithub.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa

# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa
```

如果同一个平台配置了多个不同账号的SSH密钥，那么在进行 clone、remote 等操作时，地址需要做一些改变，否则会找不到而报错。在克隆时 `git@github.com:xx.git` 需要替换为对应的 `Host` 内容。例如使用 github2 的账户关联远程仓库：`git remote add origin git@xyzgithub.com:XXX/repository.git`，这里需要使用我们自定义的 Host 内容。

### 4. 测试是否配置成功

```bash
# 使用 ssh -T 进行测试，成功的话会打印 Hi，username……
$ ssh -T git@gitee.com
$ ssh -T git@github.com
$ ssh -T git@xyzgithub.com
```

以gitee为例，成功的话会返回下图内容：
![sucess](https://images.gitee.com/uploads/images/2018/0814/170837_4c5ef029_551147.png)

## Git常用命令

### 1. 配置命令

```bash
# 配置个人信息，方便远程托管平台统计个人的贡献
$ git config --global user.name "个人名称或昵称"
$ git config --global user.email "你的邮箱"

# 列出所有配置信息：
$ git config --list
# 取消某一项全局配置
$ git config --global --unset user.name

# core.autocrlf 这个设置主要针对换行符问题，使用新版的 Git 一般不用设置，这个不会有大问题，只是格式问题。
# Window 设置 (检入时自动转换为LF，检出时LF转为CRLF)
$ git config --global core.autocrlf true
# Mac、Linux 设置 (检入时CRLF替换为LF，检出时无操作)
$ git config --global core.autocrlf input

# 乱码问题解决
# 文件名显示错误
$ git config --global core.quotepath false
# 输出日志乱码：
$ git config --global i18n.commitEncoding utf-8
$ git config --global i18n.logOutputEncoding utf-8
```

> 检入：提交，检出: 下载

### 2. 远程操作

```bash
# 克隆远程库到本地
$ git clone https://github.com/个性地址/仓库名称.git

# 克隆指定的分支
$ git clone -b 分支名称 仓库地址

# 关联远程仓库
$ git remote add origin https://gitee.com/用户个性地址/HelloGitee.git
# 取消远程关联
$ git remote remove origin
# 关联上游仓库
$ git remote add upstream 仓库地址
# 取消关联上游仓库
$ git remote remove upstream

# 删除远程分支 --delete 简写为 -d
$ git push origin -d branch_name
# 将本地分支推送到远程分支并建立映射，如果远程不存在将自动创建
$ git push -u origin 分支名称
```

### 3. 常用操作

工作区：本地文件
暂存区：执行 git add xxx 添加后的文件
远程库：执行 git push 后

```bash
# 将当前目录所有文件添加到git暂存区，也可以一次提交多个文件，使用空格分开
$ git add .
# 提交到版本库并备注提交信息
$ git commit -m "my first commit" 
# 推送到远程仓库,如果本地和远程的分支名称相同，只写一个分支名称即可
# 第一个 master 是本地分支，后面的一个是远程分支
$ git push origin local_branch:remote_branch

# 切换分支
$ git checkout 分支名 / git switch branch_name
# 删除本地分支
$ git branch -d branch_name
# 查看本地所有分支
$ git branch -a
# 查看远程分支
$ git branch -r
# 查看本地与远程分支的一映射关系
$ git branch -vv
# 重命名本地分支
$ git branch -m old_name new_name

# 文件比较
# 查看工作目录与暂存区的不同
$ git diff

# 移除文件
# 从暂存区移除文件，会把工作区相应文件一并删除，回收站无法还原。如果该文件没有提交，则无法删除，可以使用 -f 选项强制删除。
$ git rm
# 只删除暂存区文件，保留工作区文件            
$ git rm --cached  
# 删除提交到暂存区的内容
$ git rm --cache 文件名

# 查看提交历史：
$ git log
# 图形化显示
$ git log --graph 
# 单行显示、短格式、长格式 oneline，short，full
$ git log --pretty=oneline/short/full

# 将已提交的版本库（commit）、暂存区（add）、工作区（本地）的内容恢复到指定的版本：
$ git reset --hard 版本ID
# 只撤销已提交的版本库、暂存区的内容，不会修改工作区：
$ git reset --mixed 版本ID
# 只撤销已提交的版本库，不会修改暂存区和工作区
$ git reset --soft 版本库ID
```

## 一张图

{% image https://raw.sevencdn.com/prettywinter/dist/main/images/doc/git_command.jpg git常用命令小结 %}

## Git Submodule

`git submodule` 是一个很好的多项目使用共同类库的工具，它允许类库项目做为 repository,子项目做为一个单独的git项目存在父项目中，子项目可以有自己的独立的commit，push，pull。而父项目以Submodule的形式包含子项目，父项目可以指定子项目header，父项目中会的提交信息包含Submodule的信息，再clone父项目的时候可以把Submodule初始化。

`git submodule` 命令用于初始化，更新或检查子模块。

```bash
git submodule [--quiet] add [<options>] [--] <repository> [<path>]
git submodule [--quiet] status [--cached] [--recursive] [--] [<path>…​]
git submodule [--quiet] init [--] [<path>…​]
git submodule [--quiet] deinit [-f|--force] (--all|[--] <path>…​)
git submodule [--quiet] update [<options>] [--] [<path>…​]
git submodule [--quiet] summary [<options>] [--] [<path>…​]
git submodule [--quiet] foreach [--recursive] <command>
git submodule [--quiet] sync [--recursive] [--] [<path>…​]
git submodule [--quiet] absorbgitdirs [--] [<path>…​]
```

如果要 clone 的仓库中包含子模块，那么在拉取仓库的时候使用 `git clone -r ....git` 命令即可获取到子模块。否则，子模块是一个 **空文件夹**，你必须运行两个命令：`git submodule init` 用来初始化本地配置文件，而 `git submodule update` 则从该项目中抓取所有数据并检出父项目中列出的合适的提交。

### 删除Submodule

git 并不支持直接删除 Submodule，需要手动删除对应的文件:

```bash
cd pod-project
git rm --cached pod-library
rm -rf pod-library
rm .gitmodules
# 更改git的配置文件config
vim .git/config
# 删除submodule相关的内容,然后提交到远程服务器
git commit -a -m 'remove pod-library submodule'
```
