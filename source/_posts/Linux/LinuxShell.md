---
title: Linux Shell
categories:
  - Linux
  - Shell
tags:
  - linux
  - shell
abbrlink: dccc50f5
---

Linux 的 Shell 比较好用，有很多的命令，使用 Shell 脚本可以帮助我们节约时间。下面一起来简单了解一下吧。附带一个 Linux 的学习网站：https://www.linuxshelltips.com/，本文比较垃圾，看这个比较好。

<!-- more -->

## 一、Shell脚本

shell 文件格式：

```bash
# 运行shell脚本的程序，通常使用Bourne Again Shell（/bin/bash） Bourne Shell（/bin/sh）
#!/bin/bash OR #!/bin/sh
# 代表这个 shell 程序需要键盘接收，要不要都行，一个标识
#filename:score

# 变量名称 SCORE，从命令行接收变量值赋值给该变量
read SCORE
...
```

### 1. 必备知识

条件比较大小时，需要使用以下的格式： 

| 等于  | 大于等于 | 小于等于 | 不等于 | 大于  | 小于  |
| :---: | :------: | :------: | :----: | :---: | :---: |
|  -eq  |   -ge    |   -le    |  -ne   |  -gt  |  -lt  |


| 参数  |                                                   说明                                                    |
| :---: | :-------------------------------------------------------------------------------------------------------: |
|  $?   |                       显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。                       |
|  $#   |                                          传递到脚本的参数个数。                                           |
|  \$*  | 以一个单字符串显示所有向脚本传递的参数。如"\$*"用「"」括起来的情况、以"\$1 \$2 … \$n"的形式输出所有参数。 |
|  \$$  |                                         脚本运行的当前进程ID号。                                          |
|  $-   |                               显示Shell使用的当前选项，与set命令功能相同。                                |

### 2. 循环

1. for 循环语句

    运算时使用 let

    ```bash
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

2. while 循环语句

    ```bash
    while 条件
    do 
        语句块
    done
    ```

3. until 循环语句

    和while循环同样，不同的是判断循环的条件，while条件为真时循环，until 条件为假时循环。

    ```bash
    until 条件
    do 
        语句块
    done
    ```

### 3. 判断

shell 脚本的判断语句使用中括号，两端需要加入空格。比如下面这段代码(文件名为：`test.sh`)：

```bash test.sh
#!/bin/bash

if [ 1 -ge 2 ];then
  echo "yes"
else
  echo "no"
fi
```

如果 1 之前和 2 之后没有空格，将会报错，并且永远执行 **else** 分支：

```bash
./test.sh: line 3: [1: command not found
```

## 二、脚本示例

```bash
#!/bin/bash

# 使用超级管理员的命令需要输入密码，这里先输入一下 yourpassword 替换为超级管理员的密码
sudo -S -v << EOF
yourpassword
EOF

echo -n "请输入创建的文件夹："
read name

if [ ! -d "$name" ];then
    echo "$name 文件夹不存在,正在创建"
    sudo mkdir /$name
else 
    echo "/$name 文件夹已存在"

sudo mkdir /$name/test

if [ $? eq 0 ];then
    echo "文件夹创建成功"
else 
    echo "创建失败"

sudo touch /$name/test/111.txt 
# 对文件写入内容
echo "自古星耀晦明时，不持太阿误剑诗" | tee /$name/test/111.txt
# -a 追加内容
echo "无边落木萧萧下，不尽长江滚滚滚来" | tee -a /$name/test/111.txt
```

> Shell 脚本对 > 或者 >> 符号有不同的处理，如果使用这种方式会报错；这里使用了 tee 命令来进行文本输入。
