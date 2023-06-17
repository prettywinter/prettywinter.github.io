---
wiki: Git
layout: wiki
title: Git 配置
order: 1
---

{% link https://git-scm.com/book/zh/v2 Git官方文档 %}

## Git配置多个SSH密钥

Git 的官网下载比较慢，可以去 [淘宝镜像站](https://npm.taobao.org/mirrors/git-for-windows/) 选择合适的版本进行下载。

Git 安装后配置一下基本信息，这样每次的提交记录都会显示该名称，如果参与了项目，可以统计项目贡献等：

```bash
# 配置个人信息，方便远程托管平台统计个人的贡献
$ git config --global user.name "个人名称或昵称"
$ git config --global user.email "你的邮箱"

# 列出所有配置信息：
$ git config --list / git config -l
# 取消某一项全局配置
$ git config --global --unset user.name
```

为什么配置SSH密钥？
避免每次 pull、push 都需要我们手动去输入账户信息。用户信息的认证工作通过 SSH 认证自动帮我们进行。

### 1. 生成Github用的SSH-Key

```bash
# Linux Mac
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f ~/.ssh/github_id_rsa
# Window
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f C:/Users/用户名/.ssh/github_id_rsa
```

### 2. 生成gitee使用的SSH-Key

```bash
# Linux Mac
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f ~/.ssh/gitee_id_rsa
# Window
$ ssh-keygen -t ed25519 -C 'xxx@xx.com' -f C:/Users/用户名/.ssh/gitee_id_rsa
```

> 2022-08-22 修改为 ed25519 密钥生成。

### 3. 配置

在大多数系统中，默认私钥（~/.ssh/id_rsa 和 ~/.ssh/identity）会自动添加到 SSH 身份验证代理中。应无需运行 ssh-add path/to/key，除非在生成密钥时覆盖文件名。

在 Linux 系统中比较容易出现 [Permission denied (publickey)](https://docs.github.com/cn/authentication/troubleshooting-ssh/error-permission-denied-publickey) 的错误，这就需要使用 `ssh-add` 把签名的指纹加入到 SSH 的 session 中。但是也会出现问题，综合以上内容，浪子推荐在本地配置 `config` 文件，我们手动创建文件并添加如下内容（其中 Host 和 HostName 填写 git 服务器的域名，IdentityFile 指定私钥的路径）

```bash .ssh/config
# github1
# Host 可以自己定义，在此文件中必须唯一，之后在 down 源码时需要使用这个自定义的 host
Host github.com
# HostName 是代码托管平台的域名，例如：code.aliyun.com, github.com, gitee.com 等
HostName github.com
# 设置用户身份认证方式，这里当然使用 公钥 的方式
PreferredAuthentications publickey
# 指定公钥对应的私钥路径
IdentityFile ~/.ssh/github1_id_rsa

# github2
Host xyzgithub.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github2_id_rsa

# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa
```

如果同一个平台配置了多个不同账号的SSH密钥，那么在进行 clone、remote 等操作时，地址需要做一些改变，否则会找不到而报错。在克隆时 `git@github.com:xx.git` 需要替换为对应的 `Host` 内容。

例如使用 github2 的账户关联远程仓库：
```bash
git remote add origin git@xyzgithub.com:XXX/repository.git
```

这里需要使用我们自定义的 `Host` 内容。

### 4. 测试是否配置成功

```bash
# 使用 ssh -T 进行测试，成功的话会打印 Hi，username……
$ ssh -T git@gitee.com
$ ssh -T git@github.com
$ ssh -T git@xyzgithub.com
```

以gitee为例，成功的话会返回下图内容：

![sucess](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20221118211700.png)

### 5.Windows 中 Git-Bash 中配置 tree 命令和 ll 命令

#### 5.1 tree

[下载tree的二进制文件](http://gnuwin32.sourceforge.net/package/tree.htm) 后解压，把 `bin` 目录下的 tree.exe 文件复制到 Git 安装目录 `xxx\Git\usr\bin\` 下即可。

#### 5.1 ll

在用户的根目录下 `C:\Users\jhlz` 新建一个 `.bashrc` 文件（如果有的话就不需要了），熟悉的少侠应该这个文件是做什么的以及如何配置。在该文件中加入以下内容，保存退出:

```bash .bashrc
alias ll='ls -al'
```

#### 5.3 配置 git-bash 编码为 utf-8

修改 Git 安装目录下的 `/etc/bash.bashrc` 文件，在最后加入下面的内容，保存退出，重启 git-bash 即可。

```bash D:\xxx\Git\etc\bash.bashrc
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```
