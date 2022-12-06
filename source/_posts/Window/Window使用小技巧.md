---
title: Window使用小技巧
categories: Window
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

## Vmware

VMWare虚拟机配置双网卡

使用 VMWare 时，推荐创建的虚拟机配置双网卡，默认的使用 NAT 模式，新添加一个使用 仅主机 模式，然后进行修改配置。这样就算以后连接无数个无线网，电脑依然能上网，但是使用 SSH 远程连接时依然不用修改 Linux 的网络配置文件。实现了一次配置，永久使用的效果。配置也很简单：

在 VMWare 里创建一个虚拟机后，打开改虚拟机的设置，硬件下方的”添加“，然后在弹出的列表里选择”网络适配器“，之后点击新添加的网络适配器，选择”仅主机“。

然后打开网络适配器修改 VMWare Network Adapter VMnet1 的 IP 为简单的地址，Linux 里的网络配置文件的 IP 地址不要和这个一样，因为 VM1 是本机和 Linux 虚拟机通信的 IP 地址，不能一样，推荐 VM1 10.0.0.1，Linux 配置为 10.0.0.2，简单好记。

> **注意：** 如果是新建的虚拟机，在安装之前按照以上方法配置，安装后查看网络配置文件时或许会有两个不同的文件，比如：ens33，ens34。如果是之前已经安装过的单网卡想要配置双网卡，可以把已经存在网络配置文件复制一份，修改文件里面的内容即可。如果新建的虚拟机也是只有一个配置文件，也可以直接复制原有的文件进行修改。

另外，第一次安装后网络服务是正常的，之后换了一个无线网连接就出现了问题，可能是因为 NetworkManager 的问题，可以关闭此服务。然后重启网络。内网相通，但使用 xshell 等终端工具连不上虚拟机也连不上外网也可以尝试使用这个方法。

```bash
# 关闭服务
systemctl stop NetworkManager
# 禁止开机启动
systemctl disable NetworkManager
# chkconfig NetworkManager off

# 重启网络
systemctl restart network
```

{% link https://blog.csdn.net/weistin/article/details/80676955 参考文章地址 %}
