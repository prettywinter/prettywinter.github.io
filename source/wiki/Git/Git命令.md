---
wiki: Git
layout: wiki
title: Git 命令
order: 2
---

## Git常用命令

Git 的命令使用特别多，详情可以去 [Git官方文档](https://git-scm.com/book/zh/v2) 查阅，这里只罗列一些常用的命令。

工作区：本地计算机文件位置
暂存区：执行 `git add xxx` 后的文件位置
版本索引：执行 `git commit -m` 后
远程库（代码托管平台：Github、gitee等）：执行 `git push` 后文件位置

### 1. 配置命令

```bash
# 配置个人信息，方便远程托管平台统计个人的贡献
$ git config --global user.name "个人名称或昵称"
$ git config --global user.email "你的邮箱"

# 列出所有配置信息：
$ git config --list
# 取消某一项全局配置
$ git config --global --unset user.name


################### 下面的配不配都不重要，记住上传的文件都是 LF 就行 #################
# core.autocrlf 这个设置主要针对换行符问题，使用新版的 Git 一般不用设置，这个不会有大问题，只是格式问题。
# Window 设置 (检入时自动转换为LF，检出时LF转为CRLF)
$ git config --global core.autocrlf true
# Mac、Linux 设置 (检入时CRLF替换为LF，检出时无操作)
$ git config --global core.autocrlf input

# 更新 git 到最新版本（window适用）
git update-git-for-windows
```

> 检入：提交，检出: 下载

### 2. 添加、删除、提交、推送、比较

```bash
# 将当前目录所有文件添加到git暂存区，也可以一次提交多个文件，使用空格分开
$ git add .
# 提交到版本库并备注提交信息
$ git commit -m "my first commit"
# 如果第一次提交的内容写的不好或者还有新的相关文件未提交，可以使用下面的命令覆盖之前一次的 commit
$ git commit --amend -m "override first commit"
# 推送到远程仓库,如果本地和远程的分支名称相同，只写一个分支名称即可
# 第一个 master 是本地分支，后面的一个是远程分支
$ git push origin [local-branch]:[remote-branch]

# 移除文件
# 从暂存区移除文件，会把工作区相应文件一并删除，回收站无法还原。如果该文件没有提交，则无法删除，可以使用 -f 选项强制删除。
$ git rm [file]
# 只删除暂存区文件，保留工作区文件            
$ git rm --cached [file]
# 删除提交到暂存区的内容
$ git rm --cache [file]

# 查看提交历史：
$ git log
# 图形化显示
$ git log --graph 
# 单行显示、短格式、长格式 oneline，short，full
$ git log --pretty=oneline/short/full

# 撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交：
$ git reset --hard 版本ID
# 回退到某个版本：
$ git reset --soft 版本库ID

# 文件比较
# 查看工作目录与暂存区的不同
$ git diff
```

### 3. 分支

```bash
# 切换分支
$ git checkout [branch] / git switch [branch]
# 删除本地分支
$ git branch -d [branch]
# 查看本地所有分支
$ git branch -a
# 查看远程分支
$ git branch -r
# 查看本地与远程分支的一映射关系
$ git branch -vv
# 重命名本地分支
$ git branch -m [old-branch] [new-branch]
```

### 4. 远程操作

```bash
# 克隆远程库到本地
$ git clone https://github.com/个性地址/仓库名称.git

# 克隆指定的分支
$ git clone -b 分支名称 仓库地址

# 关联远程仓库
$ git remote add origin https://gitee.com/用户个性地址/HelloGitee.git
# 取消远程关联
$ git remote remove origin
# 查看关联的远程库地址
$ git remote -v

# 关联上游仓库
$ git remote add upstream 仓库地址
# 取消关联上游仓库
$ git remote remove upstream

# 删除远程分支 --delete 简写为 -d
$ git push origin -d branch_name
# 将本地分支推送到远程分支并建立映射，如果远程不存在将自动创建
$ git push -u origin 分支名称
```

### 4. 一张图总结

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/git_command.jpg git常用命令小结 %}
