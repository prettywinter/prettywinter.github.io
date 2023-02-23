---
title: Window问题集锦
categories: Window
abbrlink: f085fb33
---

Windows 使用小窍门，可以帮助你提高效率。

<!-- more -->

## 一、快捷方式

| 目标     | 操作                             |
| -------- | -------------------------------- |
| 控制面板 | win + r 输入 `appwiz.cpl` 回车   |
| 系统变量 | win + r 输入 `sysdm.cpl` 回车    |
| 磁盘管理 | win + r 输入 `diskmgmt.msc` 回车 |
| 服务     | win + r 输入 `services.msc` 回车 |
| 电脑信息 | win + r 输入 `winver` 回车       |

## 二、删除开机多余启动项

{% kbd Win %} + {% kbd R %}，输入 `msconfig`，点击确定，选择 “引导” 标签，选择 **非当前OS** 记录，点击删除并应用，按照提示重启即可。

管理员方式进入 ps：

```bash
# 查看启动项
efibootmgr
# 删除对应的项
efibootmgr -b 0001(序号) -B
```

## 三、PowerShell运行脚本

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

## 附：WSL 相关问题

用不习惯，浪子已弃坑。帮助命令 `wsl -h`，感觉还是比较清楚的。

### 安装后root用户密码

安装后默认是没有 root 用户密码的，使用 `sudo passwd` 去指定。

### docker

WSL 中使用 docker 必须下载 [Docker Desktop](https://docs.docker.com/desktop/windows/wsl/)。

#### Docker-Desktop启动ElasticSearch失败

在 Windows 下，使用 Docker 启动 ES 服务可能会遇到内存不足的问题，调整的方式是通过命令行的 `wsl` 命令进入 docker-desktop 的终端，然后通过 `sysctl` 命令调整系统参数。

```bash
# 使用 cmd 进入 docker
wsl -d docker-desktop
# 调整相应的参数，docker 启动 ES 服务时会有打印，修改为指定的即可
sysctl -w vm.max_map_count=262144
```

也可以编辑文件 `vi /etc/sysctl.conf` 加入 `vm.max_map_count=262144`，保存退出执行 `sysctl -p` 后重启 es 服务即可。


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
