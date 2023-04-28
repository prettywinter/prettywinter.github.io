---
wiki: Git
layout: wiki
title: Git 提交信息
order: 4
---

Git 提交信息规范是一个好的习惯，这样团队合作的时候，如果出问题可以先看 git log，但是如果提交的信息不明确却就……所以我们可以使用工具辅助在提交的时候填写相关内容。

## 一、commitizen简介

commitizen：基于 Node.js 的 `git commit` 命令行工具，辅助生成标准化规范化的 commit message。

cz-git、git-cz、cz-customizable、cz-conventional-changelog 等是 commitizen 的适配器：提交时选择的交互式页面，依赖于 commitizen 工作。你可以全部安装体验，找到自己喜欢的使用，使用哪个适配器在 `package.json` 中配置即可。

本文选用 cz-git 进行安装。

[cz-git](https://cz-git.qbb.sh/zh/) Github 地址：{% copy https://github.com/Zhengqbbb/cz-git %}

## 二、安装

### 1. 本地安装cz-git

```bash
npm install -g commitizen
npm install -D cz-git
```

```json package.json
{
    ...
  "config": {
    "commitizen": {
      "path": "./node_module/cz-git|git-cz|cz-customizable|cz-conventional-changelog"
    }
  }
}
```

### 2. 全局安装cz-git

```bash
npm install -g commitizen cz-git
```

```json package.json
{
    ...
  "config": {
    "commitizen": {
      "path": "cz-git|git-cz|cz-customizable|cz-conventional-changelog"
    }
  }
}
```

### 3. 使用

```bash
cz
```

多人合作避免 cz-git 版本不一致问题可以选用本地安装，使团队成员使用同一个版本，个人可以直接使用全局安装。

## 三、[czg](https://cz-git.qbb.sh/zh/cli/)

czg 的安装、使用非常方便，它也可以进行自定义配置，但是省去了很多的麻烦，浪子非常推荐使用，具体内容可以查看 [官方文档](https://cz-git.qbb.sh/zh/cli/)。

无需前置配置，无需适配器，没有额外的步骤，你可以使用 `npx | npm 脚本 | 全局下载...` 在你的任何项目中快速启动。
可定制化: 内部包含 cz-git 的核心，继承了 cz-git 的所有特性，具有相同的行为，配置加载... 你可以根据自己的需要配置的 CLI 的行为。

```bash
# 全局安装
npm i -g czg
# 特性以及帮助文档
czg --help
```
