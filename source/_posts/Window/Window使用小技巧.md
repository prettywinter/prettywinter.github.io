---
title: Window使用小技巧
categories: Window
abbrlink: f085fb33
---

Windows 使用小窍门，可以帮助你提高效率。

<!-- more -->

## 快捷方式

| 目标     | 操作                             |
| -------- | -------------------------------- |
| 控制面板 | win + r 输入 `appwiz.cpl` 回车   |
| 系统变量 | win + r 输入 `sysdm.cpl` 回车    |
| 磁盘管理 | win + r 输入 `diskmgmt.msc` 回车 |
| 服务     | win + r 输入 `services.msc` 回车 |
| 电脑信息 | win + r 输入 `winver` 回车       |

## 删除开机多余启动项

{% kbd Win %} + {% kbd R %}，输入 `msconfig`，点击确定，选择 “引导”，删除非当前OS 选项，点击“应用”，按照提示重启即可。

## 永久关闭自动更新
{% kbd Win %} + {% kbd R %} ，输入 `gpedit.msc` 回车，调出 “本地计算机策略”。依次点击：

```mermaid
graph LR

root(计算机配置) --> r1(管理模板) --> r2(Windows组件) --> r3(Windows更新) --> r4(双击配置自动更新) --> r5(禁用) --> r6(关掉本地策略编辑器 重启电脑)

```

## 删除多余启动项

```bash
# 查看启动项
efibootmgr
# 删除对应的项
efibootmgr -b 0001(序号) -B
```

## PowerShell运行脚本

管理员身份运行ps，

```shell
# 输入以下命令，选择 Y
set-executionpolicy RemoteSigned
```

ps 有四种策略：

| 策略         | 说明                                                                                   |
| ------------ | -------------------------------------------------------------------------------------- |
| Restricted   | 禁止运行任何脚本配置文件（默认）                                                       |
| AllSigned    | 可以运行发布签名的脚本和配置文件，包括本地计算机上编写的脚本                           |
| RemoteSigned | 要求从网络上下载的脚本和配置文件由可信者发布签名，不要求本地计算机上编写的脚本进行签名 |
| Unrestricted | 可以运行未签名的脚本                                                                   |

## Git-Bash中使用tree命令

[下载二进制文件](http://gnuwin32.sourceforge.net/package/tree.htm) 后解压，复制 bin 目录下的 `tree.exe` 文件到 Git 安装目录的 `xxx\Git\usr\bin\` 目录下即可。

## WSL 相关问题

帮助命令 `wsl -h`，感觉还是比较清楚的。

### 安装后root用户密码

安装后默认是没有 root 用户密码的，使用 `sudo passwd` 去指定。

### docker

WSL 中使用 docker 必须下载 [Docker Desktop](https://docs.docker.com/desktop/windows/wsl/)。

### 有关路径存储设置

不论是 WSL 安装的 Linux 发行版还是使用 Docker Desktop，默认的安装路径都在 `C:\User\xxx\AppData\Local`，可以设置为其它路径。

```bash
# 1.查看wsl安装哪些分支
wsl -l -v
# 2.关闭所有的分支及wsl2
wsl --shutdown
# 3.导出相关分支
wsl --export Ubuntu D:\ubuntu.tar
# 4.注销分发并删除根文件系统。
wsl --unregister Ubuntu
# 5.将指定的 tar 文件作为新分发导入
wsl --import Ubuntu D:\WSL\Ubuntu D:\ubuntu.tar
# 之后重启
```
