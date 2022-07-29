---
title: 使用commitizen规范Git提交
categories: 配置
tags: [Git, commitizen, git-cz]
---

git-cz Github 地址：https://github.com/streamich/git-cz

<!-- more -->

## commitizen简介

commitizen：基于Node.js的 `git commit` 命令行工具，辅助生成标准化规范化的 commit message。

git-cz、cz-customizable、cz-conventional-changelog等适配器：提交时选择的交互式页面，依赖于 commitizen 工作。你可以全部安装体验，找到自己喜欢的使用，使用哪个在 `package.json` 种配置即可。

本文选用 git-cz 进行安装使用。

## 本地安装git-cz

```bash
npm install -g commitizen
npm install --save-dev git-cz
```

```json package.json
{
    ...
  "config": {
    "commitizen": {
      "path": "./node_module/git-cz|cz-customizable|cz-conventional-changelog"
    }
  }
}
```

## 全局安装git-cz

```bash
npm install -g commitizen git-cz
```

```json package.json
{
    ...
  "config": {
    "commitizen": {
      "path": "git-cz|cz-customizable|cz-conventional-changelog"
    }
  }
}
```

## 使用

```bash
git cz
# 或者
cz
# 或者
git-cz
```

多人合作避免 git-cz 版本不一致问题可以选用本地安装，使团队成员使用同一个版本，个人可以直接使用全局安装。
