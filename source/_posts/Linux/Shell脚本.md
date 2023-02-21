---
title: Shell脚本学习
categories:
  - Linux
  - Shell
tags:
  - linux
  - shell
---

<!-- more -->

## 一、Shell脚本基本命令

shell 文件格式：

```bash
# 必须
#!/bin/bash OR #!/bin/sh
# 代表这个 shell 程序需要键盘接收，要不要都行，一个标识
#filename:score

# 从命令行读取变量名
read SCORE
```

1. if 条件判断语句

    ```bash
    if 条件
    then 
        语句块
    else 
        语句块
    fi
    ```

2. for 循环语句

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

3. while循环语句

    ```bash
    while 条件
    do 
        语句块
    done
    ```

4. until 循环语句

    和while循环同样，不同的是判断循环的条件，while条件为真时循环，until 条件为假时循环。

    ```bash
    until 条件
    do 
        语句块
    done
    ```

**注意：** 条件比较大小时，需要使用以下的内容： 
| 等于  | 大于等于 | 小于等于 | 不等于 | 大于  | 小于  |
| :---: | :------: | :------: | :----: | :---: | :---: |
|  -eq  |   -ge    |   -le    |  -ne   |  -gt  |  -lt  |

另外，定义变量后引用需要加上 `$` 符号，例如 `sum` 变量，输出时 `$sum` 这样使用。

## 判断

`[]` 是 bash 内置命令，两个符号左右都要有空格分隔，否则会出现报错，影响脚本运行逻辑。

if 判断可以说是非常常用的判断了。比较数字的时候需要用以下字符比较，对于有经验的人应该很简单。

| 等于  | 大于等于 | 小于等于 | 不等于 | 大于  | 小于  |
| :---: | :------: | :------: | :----: | :---: | :---: |
|  -eq  |   -ge    |   -le    |  -ne   |  -gt  |  -lt  |


字符串比较(condition = true)：
| 相同            | 不相同           | 不为空  | 长度为0    | 长度非0    |
| --------------- | ---------------- | ------- | ---------- | ---------- |
| [ str1 = str2 ] | [ str1 != str2 ] | [ str ] | [ -z str ] | [ -n str ] |

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
