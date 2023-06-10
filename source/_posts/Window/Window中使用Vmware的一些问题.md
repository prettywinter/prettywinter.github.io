---
title: Window中使用VMware的一些常见问题
categories: Windows
abbrlink: 13fda3d
---

虽然 Win 下的 WSL 广受好评，但是浪子懒得折腾，不折腾使用还是不太方便。还是比较喜欢使用 Vmware 创建 Linux 环境，虽然它很吃内存。。。但也没办法 :relieved:

<!-- more -->

## Vmware 虚拟机配置双网卡

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

## CentOS

1. 用的好好的虚拟机，之前内网都通，后面使用的时候使用 SSH 工具连不上虚拟机了，并且虚拟机也连不上外网

```bash
# 1.停止并关闭 networkmanager 服务
systemctl stop NetworkManager
systemctl disable NetworkManager
# 2.重启网络
systemctl restart network
```

[参考原博客地址](https://blog.csdn.net/weixin_44695793/article/details/108089356)

## 其它问题

### 1. ping

ping 不通也可能是连接了 VPN 的问题。如果在连接 VPN 之前先开启了虚拟机并连接是可以的，当连接了 VPN 之后就不行了。

虚拟机可以ping通本机，本机ping不通虚拟机

如果使用的是 NAT 模式，不在同一个网段也可以 ping 通。因为当你的网络连接变了之后（从连 wifi 到使用网线），本机的 IP 网络也会随之更改，所以需要确认一边。设置完成记得重启。
