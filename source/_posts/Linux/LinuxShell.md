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

Linux 的 Shell 比较好用，有很多的命令，使用 Shell 脚本可以帮助我们节约时间。本文比较垃圾，看这个 https://www.geeksforgeeks.org/how-to-create-a-shell-script-in-linux/ 比较好。

<!-- more -->

## 一、关于 Shell

Linux中有许多可用的Shell，例如The bourne shell（sh），The Korn Shell（ksh）和 GNU Bourne-Again Shell（bash）。为 sh shell 编写的脚本称为 shell 脚本，它们可以由 ksh 和 bash shell 解释。ksh 和 Bash 是原始 sh shell 的改进版本，它们比 sh 具有更多的功能。Bash 通常是大多数 Linux 发行版中的默认 shell，专门为 bash shell 编写的脚本称为 bash 脚本。

您可以指定脚本将使用哪个 shell，即使脚本是从另一个 shell 终端执行的。为此，请在脚本文件顶部添加“#！”，后跟所选 shell 的绝对路径。要将 bash 指定为解释器，请在 shell 脚本顶部添加以下行。

```bash
#!/bin/bash
```

### 1. 整数比较

| 操作符 | 描述     |
| ------ | -------- |
| -eq    | 等于     |
| -ne    | 不等于   |
| -gt    | 大于     |
| -ge    | 大于等于 |
| -lt    | 小于     |
| -le    | 小于等于 |

### 2. 字符串比较

| 操作符 | 描述                        |
| ------ | --------------------------- |
| ==     | 等于                        |
| !=     | 不等于                      |
| \<     | 小于，按 ASCII 字母顺序排列 |
| \>     | 大于，按 ASCII 字母顺序排列 |

> 在 `<` 和 `>` 之前添加一个 \，因为它们在 `[ ]` 结构中键入时需要转义。

### 3. 几个内置的 shell 变量

| 参数  |                                                                 说明                                                                  |
| :---: | :-----------------------------------------------------------------------------------------------------------------------------------: |
|  $?   | 每当命令结束并将控件返回到父进程时，它都会返回介于 0 和 255 之间的退出代码。退出代码 0 表示命令成功，任何其他退出代码表示命令不成功。 |
|  $#   |                                                        传递到脚本的参数个数。                                                         |
|  \$*  |               以一个单字符串显示所有向脚本传递的参数。如"\$*"用「"」括起来的情况、以"\$1 \$2 … \$n"的形式输出所有参数。               |
|  \$$  |                                                       脚本运行的当前进程ID号。                                                        |
|  $-   |                                             显示Shell使用的当前选项，与set命令功能相同。                                              |

在 shell 中，有 `[]` 和 `[[]]` 两种条件测试表达式，它们有一些区别：

1. `[]` 是 shell 自带的 test 命令的一种表现形式，而 `[[ ]]` 可以看作是内建的测试功能，是相对于传统的 test 命令的扩展强化版本。
2. `[]` 只是指针对常规的字符串匹配和文件匹配的，`[[ ]]` 支持字符串、文件、模式匹配、类型匹配等。
3. 在 `[]` 中的变量在没有使用双引号包括时会被扩展和分割，而在 `[[ ]]` 中不会发生这种情况。
4. 在 `[]` 中，|| 和 && 代表或和和逻辑运算，而 `[[ ]]` 中，还有两个额外的操作符：`<` 和 `>` 可用于字符串比较和整数比较。
5. `[]` 支持一些正则表达式的特殊字符，`[[ ]]` 支持更多的扩展的正则表达式特性。

因为 `[[ ]]` 支持的东西更多更强大，建议优先使用 `[[ ]]` 语法。但需要注意，`[[ ]]` 只在 bash 及其衍生版本支持，像 sh，dash 等不支持。

比如寻找文件名是否以某个后缀结尾，可以使用以下语法：

```bash
if [[ $filename =~ \.zip$ ]];then
    echo "文件以 zip 结尾"
else
    echo "文件不以 zip 结尾"
fi
```

### 4. 循环语法

#### 1. for 循环语句

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

#### 2. while 循环语句

```bash
while 条件
do 
    语句块
done
```

#### 3. until 循环语句

    和while循环同样，不同的是判断循环的条件，while条件为真时循环，until 条件为假时循环。

```bash
until 条件
do 
    语句块
done
```

### 5. 判断语法

shell 脚本的判断语句使用中括号，两端需要加入空格。比如下面这段代码(文件名为：`test.sh`)：

```bash test.sh
#!/bin/bash

if [ 1 -ge 2 ];then
  echo "yes"
else
  echo "no"
fi
```

在 [ 和 ] 之前键入一个空格，同时指定要检查的条件，否则会出现错误。

```bash
./test.sh: line 3: [1: command not found
```

## 二、脚本示例

如果你的脚本是在 Window 下写完上传到 Linux 中的，那么文件的换行符可能有问题，可以使用以下命令进行转换:
```bash
# 两个命令二选一
tr -d '\r' < 文件名 > 新文件名
sed -i 's/\r//g' 文件名
```

```bash
#!/bin/bash

# 使用超级管理员的命令需要输入密码，这里先输入一下，yourpassword 替换为超级管理员的密码
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

> Shell 脚本对 bash 中的 > 或者 >> 有不同的处理，如果 shell 中使用这种方式将会报错；
> 
> 因此这里使用了 tee 命令来进行文本输入追加。
