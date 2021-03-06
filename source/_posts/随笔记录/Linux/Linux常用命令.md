---
title: Linux常用命令
categories: Linux
tags: [命令]
abbrlink: 817c7d82
---

Linux 常用命令总结。这些东西还是要常敲的，记住的总会忘记。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [一、几个目录介绍](#一几个目录介绍)
- [二、常用命令](#二常用命令)
- [三、VI编辑器常用命令](#三vi编辑器常用命令)
- [四、权限](#四权限)
- [五、Shell脚本组成命令](#五shell脚本组成命令)
- [六、用户、用户组命令](#六用户用户组命令)
- [七、tar包常用命令](#七tar包常用命令)
- [八、挂载文件命令](#八挂载文件命令)
- [九、任务计划(Crontab)命令](#九任务计划crontab命令)

<!-- /code_chunk_output -->

## 一、几个目录介绍

|目录名称|介绍|
|--|--|
|/home|    用户主目录，子目录名称默认以该用户名命名。   |
|/root：|  root用户主目录。|
|/bin：|   常用的命令文件。|
|/sbin：|  包含系统管理员和root用户所使用的命令文件。   |
|/boot：|  Linux系统的内核文件和引导装载程序文件。      |
|/opt：|   第三方应用程序的安装文件。|
|/etc：|   Linux上的大部分配置文件，建议修改之前先备份。 |
|/usr：|   包含可以供所有用户使用的程序和数据。
|/media：| 系统自动为某些设置(比如u盘等)挂载提供挂载目录。|
|/mnt：|   手动为某些设备(比如硬盘)挂载提供挂载目录。    |
|/dev：|   大部分设备文件。|

## 二、常用命令

```bash
# 创建链接文件，
# -s 创建软链接，相当于 win 下的快捷方式，不加 -s 选项为硬链接。
ln -s 源文件 新的链接文件名
# 显示计算机及操作系统的所有信息
uname -a | cat /proc/version
# 查看内存信息,-g 以G为单位，-m 以MB为单位
free
# 查看磁盘空间,如果查看目录文件的大小，命令后跟指定目录或者文件即可
df -h 或者 du -h
# 查看所有进程
ps -ef
# 查看所有监听端口 
netstat -lntp
# 查看所有已经建立的连接
netstat -antp
# 查看网络统计信息进程
netstat -s
# 查看活动用户
w
# 查看用户登录日志
last
# 查看系统所有用户
cut -d: -f1 /etc/passwd
# 查看系统所有组
cut -d: -f1 /etc/group
# 查看当前用户的计划任务服务
crontab -l
# 列出所有系统服务
chkconfig –list
# 列出所有启动的系统服务程序
chkconfig –list | grep on
```

## 三、VI编辑器常用命令

|命令|说明|
|--|--|
|yy|复制光标所在行；|
|nyy|复制光标所在行开始的 `n` 行,n 代表数字；|
|p|粘贴复制的内容|
|dd|删除光标所在行；|
|ndd|删除光标所在行开始的 `n` 行，n代表数字；|
|i|在当前光标所在处后进行编辑|
|o|在当前光标所在行的下一行新开一行进行编辑|
|O|在光标当前所在行的上一行新开一行插入|
|gg|让光标移动到文件首(第一行的第一个非空白字符处)|
|G|使光标移动到文件尾(最后一行的第一个非空白字符)|
|u|undo，取消上一步操作|
|Ctrl + r|redo，回到 undo 之前|
|set nu/nonu| 显示/不显示行号
|ggdG|清空所有内容|
|:wq|保存并退出，`:x`、`Shift+zz`具有同样效果|
|:q!|强制退出|

## 四、权限

命令前加上 `sudo` 即以超级管理员的权限运行。普通用户的 bash 环境中，命令前是 `$` 符号，而 root 用户是 `#` 符号，

```bash
# 切换到 root 用户
su root
# 如果中间加入 - ，代表连用户的 shell 环境一起切换
su - root
```

|可选项|意义|
|:--:|--|
|u|用户所有者
|g|用户组群
|o|其他用户
|a|所有用户，系统默认值
|+|添加某个权限
|-(减号)|取消某个权限
|=(等号)|赋予给定权限并取消原有权限(如果有的话)
|r|读取权限
|w|写入权限
|x|可执行权限

> 数字表示权限：r=4,w=2,x=1

![linux权限图片](https://www.runoob.com/wp-content/uploads/2014/08/rwx-standard-unix-permission-bits.png, "linux文件权限信息图")

- 给指定的文件添加某个权限：

  ```bash
  # 数字法
  chmod 777 file.sh
  # 字母法
  chmod u+x,g+wx file.sh
  ```

- 更改文件和目录所有者(二选一)：

  ```bash
  chown -R 用户.组群 文件/目录
  chown -R 用户:组群 文件/目录
  ```

`-R` 选项是递归操作，把目录下的所有文件全部修改为当前的权限。

- 只修改所属用户(即用户所有者)

  ```bash
  chown newGroup file
  ```

- 只修改所属用户组

  ```bash
  chown .newGroup file
  chown :newGroup file
  ```

## 五、Shell脚本组成命令

shell 文件格式：

```bash{.line-numbers}
#!/bin/bash
#filename:score  代表这个 shell 程序需要键盘接收

reda SCORE
```

1. if 条件判断语句

    ```bash{.line-numbers}
    if 条件
    then 
        语句块
    else 
        语句块
    fi
    ```

2. for 循环语句

    运算时使用 let

    ```bash{.line-numbers}
    for 条件
    do  
        语句块
    done
    其中：
    for ad in 1 2 3 4
    for ab in `seq 1 4`
    for ((ab = 1; ab < 4; ab++))
    这三种写法的意思都是相同的，需要特别注意的是，
    第二行那个符号不是单引号，而是 Tab 上面，Esc 下面的那个键。因为使用的时候不容易看懂，不推荐使用。
    推荐第三种写法，注意有两个括号。
    ```

3. while循环语句

    ```bash{.line-numbers}
    while 条件
    do 
        语句块
    done
    ```

4. until 循环语句

    和while循环同样，不同的是判断循环的条件，while条件为真时循环，until 条件为假时循环。

    ```bash{.line-numbers}
    until 条件
    do 
        语句块
    done
    ```

**注意：** 条件比较大小时，需要使用以下的内容： 
|等于|大于等于|小于等于|不等于|大于|小于|
|:--:|:--:|:--:|:--:|:--:|:--:|
| -eq|-ge|-le|-ne|-gt|-lt|

另外，定义变量后引用需要加上 `$` 符号，例如 `sum` 变量，输出时 `$sum` 这样使用。

## 六、用户、用户组命令

```bash{.line-numbers}
# 创建用户（如果创建用户的时候没有指定组名，默认会自动创建一个和用户名相同的组名）： 
useradd -d 设置用户主目录 -g 设置用户组群 username

# 修改用户：
# 如果添加了 -m 选项，用户旧目录会移动到新的目录中；如果不存在，就新建
usermod -l 修改账户名称 -d 设置用户主目录 -m 移动主目录内容到新的位置

# 删除用户
# 使用这个命令时记得做好备份，会将用户主目录和用户一并删除，所以主目录下的东西最好备份。
userdel -rf

# 添加组群：
groupadd

# 修改组群名：
groupmod -n newName oldName

# 删除组群：
groupdel

# 给指定的用户设置密码（如果新创建的用户没有密码，那么该用户是无法使用的，类似未激活）
passwd 用户名

# 添加用户到组群
gpasswd -a 用户名 组群名

# 删除组群的指定用户
gpasswd -d 用户名 组群名

# 查看用户属于那些群组
groups 用户名

# 让属于该群组的当前用户以指定的组群身份登录
newgrp 组群名 用户名
```

## 七、tar包常用命令

```bash
# 打包
tar -cvf

# 查看包内容
tar -tvf

# 解包内容
tar -xvf

# 参数解释
-v 显示详细处理信息
-f 指定归档包文件，后面必须跟 tar 包相关文件 xxx.tar.gz

# 以下格式的包只需要在选项中加入以下字母即可
# 例如 tar -zxvf xxx.tar.gz
# gzip(.tar.gz结尾的包)
-z

# bzip2(.tar.bz2结尾的包)
-j

# xz(以.tar.xz结尾的包)
-J

# -C 可以在解压的时候指定解压路径
tar -C 指定路径 -zvxf 目标包文件
```

## 八、挂载文件命令

mount 设备 挂载目录
umount 设备 挂载目录

## 九、任务计划(Crontab)命令

![定时任务说明](https://www.linuxprobe.com/wp-content/uploads/2016/09/crontab.png)
在以上各个字段中，还可以使用以下特殊字符：

|符号|含义|
|--|--|
| \* |代表所有的取值范围内的数字，如月份字段为*，则表示1到12个月；|
| / |代表每一定时间间隔的意思，如分钟字段为 */10，表示每10分钟执行1次。|
| - |代表从某个区间范围，是闭区间。如“2-5”表示“2,3,4,5”，小时字段中0-23/2表示在0~23点范围内每2个小时执行一次。|
| , |分散的数字（不一定连续），如1,2,3,4,7,9。|

注：由于各个地方每周第一天不一样，因此Sunday=0（第一天）或Sunday=7（最后1天）。

**crontab注意点:**
> crontab有2种编辑方式：直接编辑/etc/crontab文件与crontab –e，其中/etc/crontab里的计划任务是系统中的计划任务，而用户的计划任务需要通过crontab –e来编辑；
每次编辑完某个用户的cron设置后，cron自动在/var/spool/cron下生成一个与此用户同名的文件，此用户的cron信息都记录在这个文件中，这个文件是不可以直接编辑的，只可以用crontab -e 来编辑。
crontab中的command尽量使用绝对路径，否则会经常因为路径错误导致任务无法执行。
新创建的cron job不会马上执行，至少要等2分钟才能执行，可重启cron来立即执行。
%在crontab文件中表示“换行”，因此假如脚本或命令含有%,需要使用" \\% "来进行转义。

**Crontab配置实例:**

```bash{.line-numbers}
#每一分钟执行一次command（因cron默认每1分钟扫描一次，因此全为*即可）
*    *    *    *    *  command

#每小时的第3和第15分钟执行command
3,15   *    *    *    *  command

# 每天上午8-11点的第3和15分钟执行command：
3,15  8-11  *  *  *  command

# 每隔2天的上午8-11点的第3和15分钟执行command：
3,15  8-11  */2  *   *  command

# 每个星期一的上午8点到11点的第3和第15分钟执行command
3,15  8-11   *   *  1 command

# 每晚的21:30重启smb
30  21   *   *  *  /etc/init.d/smb restart

# 每月1、10、22日的4 : 45重启smb
45  4  1,10,22  *  *  /etc/init.d/smb restart
```
