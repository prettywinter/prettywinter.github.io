---
wiki: Git
layout: wiki
title: Git 进阶
order: 3
---

## 一、 Git Submodule

`git submodule` 是一个很好的多项目使用共同类库的工具，它允许类库项目做为 repository，子项目做为一个单独的git项目存在父项目中，子项目可以有自己的独立的commit，push，pull。而父项目以Submodule的形式包含子项目，父项目可以指定子项目header，父项目中会的提交信息包含Submodule的信息，再clone父项目的时候可以把Submodule初始化。

说人话就是一个 Git 仓库内部包含其它 Git 仓库，把外面的成为 Git 仓库，里面的称之为 Git 子仓库/子模块。常用的命令也是添加、更新、删除。

如果是初次添加子模块：

```bash
git submodule add [repository-url] [custom-directory-path]
```

如果是 clone 一个含有 submodule 的仓库，直接 clone 后子模块是一个空的文件夹，当然也支持直接 clone 子模块：

```bash
# 直接 clone 仓库以及子仓库
git clone --recurse-submodules [repository-url]

# clone 后单独 clone 子模块
git clone [repository-url]
cd [repository-path]/[submodule-path]
git submodule init && git submodule update

# init 和 update 也可以使用一个命令
git submodule update --init --recursive
```

### 1. 删除submodule

git 并不支持直接删除 submodule，需要手动删除对应的文件:

```bash
cd [repository-path]
git submodule deinit [submodule-path]
git rm -r [submodule-path]
git commit -m "delete submodule"
git push
```
