---
title: NodeJs
categories: skill
tags: Node
---

NodeJs

<!-- more -->

{% link http://nodejs.cn/learn/ Node文档 %}

## 一、常用命令

```shell
# node version
node -v
# 后台启动 推荐使用 pm2
nohup 脚本命令 &

# 查看npm版本
npm -v

# 全局安装
npm install <package-name> -g

# 所在项目安装
npm install <package-name>

# 更新单个指定包
npm update <package-name>

# 更新所有可以更新的包
npm update

# 卸载
npm uninstall <package-name>

# 搜索
npm search <package-name>

# 查看全局安装模块信息，不带 -g 代表查询安装的所有模块信息
npm list -g
npm ls

# 列出全局安装的模块 带上 --depth 0 不深入到包的支点 更简洁
npm list -g --depth 0

# 查看 npm 配置
npm config list

# 检查可更新插件
npm outdated

# 设置全局包依赖镜像地址
npm config set registry https://registry.npmmirror.com

# 回退npm到指定版本
npm install npm@6.14 -g

# 升级npm到最新版本
npm install npm -g

# 显示 npm install -g 全局安装的位置
npm root -g 

# 移除项目未使用的模块
npm prune
```

> npm install 可简写为 npm i
> npm run 相关：除了 npm run start 和 npm run test，其它的命令都不能省略 run。

## 二、Yarn

npm 是 Node 自带的包管理工具，使用起来很方便。也可以使用 yarn，yarn 是 facebook 发布的一款取代npm的包管理工具，[详情请戳](https://yarn.bootcss.com/) [yarn官网介绍](https://yarnpkg.com/getting-started/usage)

```bash
# 安装 yarn 包管理工具
npm install -g yarn
# 查看 yarn 版本
yarn --version
# 查询源
yarn config get registry

# 更换国内源
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```

## 三、pnpm

pnpm 是微软发布的一款 Node 平台的包管理工具，用的人数也挺多的。默认使用 npm 的安装源。

{% link https://pnpm.io/ 官方文档 %}
