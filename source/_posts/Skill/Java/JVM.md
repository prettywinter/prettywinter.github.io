---
title: JVM
categories:
  - Skill
  - Java
tags:
  - jvm
abbrlink: 66c016fb
---

JVM 简单理解就是运行 Java 等语言的“操作系统”，没有 JVM，Java 程序就无法运行。本篇文章就来整理一下，没有用到的就不写进来了，用过的或者学习的整理进来。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

<!-- code_chunk_output -->

- [一、JVM内存模型](#一jvm内存模型)
- [二、调优命令](#二调优命令)
- [三、常见调优工具](#三常见调优工具)
  - [Arthas](#arthas)
- [四、JVM垃圾收集算法](#四jvm垃圾收集算法)
- [附：JVM指令码](#附jvm指令码)

<!-- /code_chunk_output -->

## 一、JVM内存模型

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/JVM内存模型.png JVM内存模型图 %}

方法区和堆是所有线程共享的内存区域；而Java虚拟机栈、本地方法栈和程序员计数器是运行是线程私有的内存区域。

- Java堆（Heap）,是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。
- 方法区（Method Area）,方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
- 程序计数器（Program Counter Register）,程序计数器（Program Counter Register）是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器。当线程发生切换时，记录上次线程挂起的位置，之后线程切换回来的时候，继续从上次记录的位置开始执行。
- 虚拟机栈（JVM Stacks）,与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
- 本地方法栈（Native Method Stacks）,本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。

> 说到方法区，不得不提一下“永久代”这个概念，尤其是在JDK8以前，许多Java程序员都习惯在HotSpot虚拟机上开发、部署程序，很多人都更愿意把方法区称呼为“永久代”（Permanent Generation），或将两者混为一谈。本质上这两者并不是等价的，因为仅仅是当时的HotSpot虚拟机设计团队选择把收集器的分代设计扩展至方法区，或者说使用永久代来实现方法区而已，这样使得 HotSpot的垃圾收集器能够像管理Java堆一样管理这部分内存，省去专门为方法区编写内存管理代码的工作。但现在回头来看，当年使用永久代来实现方法区的决定并不是一个好主意，这种设计导致了Java应用更容易遇到内存溢出的问题（永久代有-XX：MaxPermSize的上限，即使不设置也有默认大小，而J9和JRockit只要 没有触碰到进程可用内存的上限，例如32位系统中的4GB限制，就不会出问题），而且有极少数方法 （例如String::intern()）会因永久代的原因而导致不同虚拟机下有不同的表现。当Oracle收购BEA获得了JRockit的所有权后，准备把JRockit中的优秀功能，譬如Java Mission Control管理工具，移植到HotSpot虚拟机时，但因为两者对方法区实现的差异而面临诸多困难。**考虑到HotSpot未来的发展，在JDK 6的时候HotSpot开发团队就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区的计划了，到了JDK 7的HotSpot，已经把原本放在永久代的字符串常量池、静态变量等移出，而到了JDK 8，终于完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间（Meta-space）来代替，把JDK 7中永久代还剩余的内容（主要是类型信息）全部移到元空间中**。
> 引自《深入理解Java虚拟机第二版》 周志明

## 二、调优命令

Sun JDK监控和故障处理命令有jps jstat jmap jhat jstack jinfo
jps：JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。
jstat：JVM statistics Monitoring是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
jmap：JVM Memory Map命令用于生成heap dump文件。
jhat：JVM Heap Analysis Tool命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看jstack，用于生成java虚拟机当前时刻的线程快照。
jinfo：JVM Configuration info 这个命令作用是实时查看和调整虚拟机运行参数。
jstack：查找死锁。

```bash
jmap ‐dump:format=b,file=eureka.hprof 14660
# jstack pid -A 后面数字为显示的线程所在行的后面10行，最后的参数是线程id的16进制表示。
jstack 19663|grep -A 10 4cd0

jstat -gc pid

# 打印GC日志方法 %t：时间
java ‐jar ‐Xloggc:./gc‐%t.log ‐XX:+PrintGCDetails ‐XX:+PrintGCDateStamps ‐XX:+PrintGCTimeStamps ‐XX:+PrintGCCause ‐XX:+UseGCLogFileRotation ‐XX:NumberOfGCLogFiles=10 ‐XX:GCLogFileSize=100M xxxx.jar
```
## 三、常见调优工具

常用调优工具分为两类,jdk自带监控工具：jconsole和jvisualvm，第三方有：MAT(Memory Analyzer Tool)、GChisto。
jconsole，Java Monitoring and Management Console是从java5开始，在JDK中自带的java监控和管理控制台，用于对JVM中内存，线程和类等的监控
jvisualvm，jdk自带全能工具，可以分析内存快照、线程快照；监控内存变化、GC变化等。
MAT，Memory Analyzer Tool，一个基于Eclipse的内存分析工具，是一个快速、功能丰富的Java heap分析工具，它可以帮助我们查找内存泄漏和减少内存消耗
GChisto，一款专业分析gc日志的工具

### [Arthas](https://arthas.aliyun.com/doc/)

首先下载到本机，用java -jar运行即可，可以识别机器上所有Java进程。

输入dashboard可以查看整个进程的运行情况，线程、内存、GC、运行环境信息。详细使用参照 [官方文档](https://arthas.aliyun.com/doc/)。

## 四、JVM垃圾收集算法

- 引用计数法
- 根可达算法
- 标记清除算法
- 标记复制算法
- 标记整理算法

## 附：JVM指令码

|JVM指令码|助记符|说明|
|--|--|--|
|0x00| nop| 什么都不做 
|0x01| aconst_null| 将null推送至栈顶 
|0x02| iconst_m1| 将int型-1推送至栈顶 
|0x03| iconst_0| 将int型0推送至栈顶 
|0x04| iconst_1| 将int型1推送至栈顶 
|0x05| iconst_2| 将int型2推送至操作数栈顶 
|0x06| iconst_3| 将int型3推送至栈顶 
|0x07| iconst_4| 将int型4推送至栈顶 
|0x08| iconst_5| 将int型5推送至栈顶 
|0x09| lconst_0| 将long型0推送至栈顶 
|0x0a| lconst_1| 将long型1推送至栈顶 
|0x0b| fconst_0| 将float型0推送至栈顶 
|0x0c| fconst_1| 将float型1推送至栈顶 
|0x0d| fconst_2| 将float型2推送至栈顶 
|0x0e| dconst_0| 将double型0推送至栈顶 
|0x0f| dconst_1| 将double型1推送至栈顶 
|0x10| bipush| 将单字节的常量值(-128~127)推送至栈顶 
|0x11| sipush| 将一个短整型常量值(-32768~32767)推送至栈顶 
|0x12| ldc| 将int, float或String型常量值从常量池中推送至栈顶 
|0x13| ldc_w| 将int, float或String型常量值从常量池中推送至栈顶（宽索引） 
|0x14| ldc2_w| 将long或double型常量值从常量池中推送至栈顶（宽索引） 
|0x15| iload| 将指定的int型本地变量推送至栈顶 
|0x16| lload| 将指定的long型本地变量推送至栈顶 
|0x17| fload| 将指定的float型本地变量推送至栈顶 
|0x18| dload| 将指定的double型本地变量推送至栈顶 
|0x19| aload| 将指定的引用类型本地变量推送至栈顶 
|0x1a| iload_0| 将第一个int型本地变量推送至栈顶 
|0x1b| iload_1| 将第二个int型本地变量推送至栈顶 
|0x1c| iload_2| 将第三个int型本地变量推送至栈顶 
|0x1d| iload_3| 将第四个int型本地变量推送至栈顶 
|0x1e| lload_0| 将第一个long型本地变量推送至栈顶 
|0x1f| lload_1| 将第二个long型本地变量推送至栈顶 
|0x20| lload_2| 将第三个long型本地变量推送至栈顶 
|0x21| lload_3| 将第四个long型本地变量推送至栈顶 
|0x22| fload_0| 将第一个float型本地变量推送至栈顶 
|0x23| fload_1| 将第二个float型本地变量推送至栈顶 
|0x24| fload_2| 将第三个float型本地变量推送至栈顶 
|0x25| fload_3| 将第四个float型本地变量推送至栈顶
|0x26| dload_0| 将第一个double型本地变量推送至栈顶 
|0x27| dload_1| 将第二个double型本地变量推送至栈顶 
|0x28| dload_2| 将第三个double型本地变量推送至栈顶 
|0x29| dload_3| 将第四个double型本地变量推送至栈顶 
|0x2a| aload_0| 将第一个引用类型本地变量推送至栈顶 
|0x2b| aload_1| 将第二个引用类型本地变量推送至栈顶 
|0x2c| aload_2| 将第三个引用类型本地变量推送至栈顶 
|0x2d| aload_3| 将第四个引用类型本地变量推送至栈顶 
|0x2e| iaload| 将int型数组指定索引的值推送至栈顶 
|0x2f| laload| 将long型数组指定索引的值推送至栈顶 
|0x30| faload| 将float型数组指定索引的值推送至栈顶 
|0x31| daload| 将double型数组指定索引的值推送至栈顶 
|0x32| aaload| 将引用型数组指定索引的值推送至栈顶 
|0x33| baload| 将boolean或byte型数组指定索引的值推送至栈顶 
|0x34| caload| 将char型数组指定索引的值推送至栈顶 
|0x35| saload| 将short型数组指定索引的值推送至栈顶 
|0x36| istore| 将栈顶int型数值存入指定本地变量 
|0x37| lstore| 将栈顶long型数值存入指定本地变量 
|0x38| fstore| 将栈顶float型数值存入指定本地变量 
|0x39| dstore| 将栈顶double型数值存入指定本地变量 
|0x3a| astore| 将栈顶引用型数值存入指定本地变量 
|0x3b| istore_0| 将栈顶int型数值存入第一个本地变量 
|0x3c| istore_1| 将栈顶int型数值存入第二个本地变量 
|0x3d| istore_2| 将栈顶int型数值存入第三个本地变量 
|0x3e| istore_3| 将栈顶int型数值存入第四个本地变量 
|0x3f| lstore_0| 将栈顶long型数值存入第一个本地变量 
|0x40| lstore_1| 将栈顶long型数值存入第二个本地变量 
|0x41| lstore_2| 将栈顶long型数值存入第三个本地变量 
|0x42| lstore_3| 将栈顶long型数值存入第四个本地变量 
|0x43| fstore_0| 将栈顶float型数值存入第一个本地变量 
|0x44| fstore_1| 将栈顶float型数值存入第二个本地变量 
|0x45| fstore_2| 将栈顶float型数值存入第三个本地变量 
|0x46| fstore_3| 将栈顶float型数值存入第四个本地变量 
|0x47| dstore_0| 将栈顶double型数值存入第一个本地变量 
|0x48| dstore_1| 将栈顶double型数值存入第二个本地变量 
|0x49| dstore_2| 将栈顶double型数值存入第三个本地变量 
|0x4a| dstore_3| 将栈顶double型数值存入第四个本地变量 
|0x4b| astore_0| 将栈顶引用型数值存入第一个本地变量 
|0x4c| astore_1| 将栈顶引用型数值存入第二个本地变量 
|0x4d| astore_2| 将栈顶引用型数值存入第三个本地变量 
|0x4e| astore_3| 将栈顶引用型数值存入第四个本地变量
|0x4f| iastore| 将栈顶int型数值存入指定数组的指定索引位置 
|0x50| lastore| 将栈顶long型数值存入指定数组的指定索引位置 
|0x51| fastore| 将栈顶float型数值存入指定数组的指定索引位置 
|0x52| dastore| 将栈顶double型数值存入指定数组的指定索引位置 
|0x53| aastore| 将栈顶引用型数值存入指定数组的指定索引位置 
|0x54| bastore| 将栈顶boolean或byte型数值存入指定数组的指定索引位置 
|0x55| castore| 将栈顶char型数值存入指定数组的指定索引位置 
|0x56| sastore| 将栈顶short型数值存入指定数组的指定索引位置 
|0x57| pop| 将栈顶数值弹出 (数值不能是long或double类型的) 
|0x58| pop2| 将栈顶的一个（long或double类型的)或两个数值弹出（其它） 
|0x59| dup| 复制栈顶数值并将复制值压入栈顶 
|0x5a| dup_x1| 复制栈顶数值并将两个复制值压入栈顶 
|0x5b| dup_x2| 复制栈顶数值并将三个（或两个）复制值压入栈顶 
|0x5c| dup2| 复制栈顶一个（long或double类型的)或两个（其它）数值并将复制值压入栈顶
|0x5d| dup2_x1| <待补充> 
|0x5e| dup2_x2| <待补充> 
|0x5f| swap| 将栈最顶端的两个数值互换(数值不能是long或double类型的) 
|0x60| iadd| 将栈顶两int型数值相加并将结果压入栈顶 
|0x61| ladd| 将栈顶两long型数值相加并将结果压入栈顶 
|0x62| fadd| 将栈顶两float型数值相加并将结果压入栈顶 
|0x63| dadd| 将栈顶两double型数值相加并将结果压入栈顶 
|0x64| isub| 将栈顶两int型数值相减并将结果压入栈顶 
|0x65| lsub| 将栈顶两long型数值相减并将结果压入栈顶 
|0x66| fsub| 将栈顶两float型数值相减并将结果压入栈顶 
|0x67| dsub| 将栈顶两double型数值相减并将结果压入栈顶 
|0x68| imul| 将栈顶两int型数值相乘并将结果压入栈顶 
|0x69| lmul| 将栈顶两long型数值相乘并将结果压入栈顶 
|0x6a| fmul| 将栈顶两float型数值相乘并将结果压入栈顶 
|0x6b| dmul| 将栈顶两double型数值相乘并将结果压入栈顶 
|0x6c| idiv| 将栈顶两int型数值相除并将结果压入栈顶 
|0x6d| ldiv| 将栈顶两long型数值相除并将结果压入栈顶 
|0x6e| fdiv| 将栈顶两float型数值相除并将结果压入栈顶 
|0x6f| ddiv| 将栈顶两double型数值相除并将结果压入栈顶 
|0x70| irem| 将栈顶两int型数值作取模运算并将结果压入栈顶 
|0x71| lrem| 将栈顶两long型数值作取模运算并将结果压入栈顶 
|0x72| frem| 将栈顶两float型数值作取模运算并将结果压入栈顶 
|0x73| drem| 将栈顶两double型数值作取模运算并将结果压入栈顶 
|0x74| ineg| 将栈顶int型数值取负并将结果压入栈顶 
|0x75| lneg| 将栈顶long型数值取负并将结果压入栈顶 
|0x76| fneg| 将栈顶float型数值取负并将结果压入栈顶
|0x77| dneg| 将栈顶double型数值取负并将结果压入栈顶 
|0x78| ishl| 将int型数值左移位指定位数并将结果压入栈顶 
|0x79| lshl| 将long型数值左移位指定位数并将结果压入栈顶 
|0x7a| ishr| 将int型数值右（符号）移位指定位数并将结果压入栈顶 
|0x7b| lshr| 将long型数值右（符号）移位指定位数并将结果压入栈顶 
|0x7c| iushr| 将int型数值右（无符号）移位指定位数并将结果压入栈顶 
|0x7d| lushr| 将long型数值右（无符号）移位指定位数并将结果压入栈顶 
|0x7e| iand| 将栈顶两int型数值作“按位与”并将结果压入栈顶 
|0x7f| land| 将栈顶两long型数值作“按位与”并将结果压入栈顶 
|0x80| ior| 将栈顶两int型数值作“按位或”并将结果压入栈顶 
|0x81| lor| 将栈顶两long型数值作“按位或”并将结果压入栈顶 
|0x82| ixor| 将栈顶两int型数值作“按位异或”并将结果压入栈顶 
|0x83| lxor| 将栈顶两long型数值作“按位异或”并将结果压入栈顶 
|0x84| iinc| 将指定位置的int型变量增加指定值（i++, i--, i+=2） 
|0x85| i2l| 将栈顶int型数值强制转换成long型数值并将结果压入栈顶 
|0x86| i2f| 将栈顶int型数值强制转换成float型数值并将结果压入栈顶 
|0x87| i2d| 将栈顶int型数值强制转换成double型数值并将结果压入栈顶 
|0x88| l2i| 将栈顶long型数值强制转换成int型数值并将结果压入栈顶 
|0x89| l2f| 将栈顶long型数值强制转换成float型数值并将结果压入栈顶 
|0x8a| l2d| 将栈顶long型数值强制转换成double型数值并将结果压入栈顶 
|0x8b| f2i| 将栈顶float型数值强制转换成int型数值并将结果压入栈顶 
|0x8c| f2l| 将栈顶float型数值强制转换成long型数值并将结果压入栈顶 
|0x8d| f2d| 将栈顶float型数值强制转换成double型数值并将结果压入栈顶 
|0x8e| d2i| 将栈顶double型数值强制转换成int型数值并将结果压入栈顶 
|0x8f| d2l| 将栈顶double型数值强制转换成long型数值并将结果压入栈顶 
|0x90| d2f| 将栈顶double型数值强制转换成float型数值并将结果压入栈顶 
|0x91| i2b| 将栈顶int型数值强制转换成byte型数值并将结果压入栈顶 
|0x92| i2c| 将栈顶int型数值强制转换成char型数值并将结果压入栈顶 
|0x93| i2s| 将栈顶int型数值强制转换成short型数值并将结果压入栈顶 
|0x94| lcmp| 比较栈顶两long型数值大小，并将结果（1，0，-1）压入栈顶 
|0x95| fcmpl| 比较栈顶两float型数值大小，并将结果（1，0，-1）压入栈顶；当其中一 个数值为NaN时，将-1压入栈顶
|0x96| fcmpg| 比较栈顶两float型数值大小，并将结果（1，0，-1）压入栈顶；当其中一 个数值为NaN时，将1压入栈顶
|0x97| dcmpl| 比较栈顶两double型数值大小，并将结果（1，0，-1）压入栈顶；当其中 一个数值为NaN时，将-1压入栈顶 
|0x98| dcmpg| 比较栈顶两double型数值大小，并将结果（1，0，-1）压入栈顶；当其中 一个数值为NaN时，将1压入栈顶
|0x99| ifeq| 当栈顶int型数值等于0时跳转 
|0x9a| ifne| 当栈顶int型数值不等于0时跳转 
|0x9b| iflt| 当栈顶int型数值小于0时跳转
|0x9c| ifge| 当栈顶int型数值大于等于0时跳转 
|0x9d| ifgt| 当栈顶int型数值大于0时跳转 
|0x9e| ifle| 当栈顶int型数值小于等于0时跳转 
|0x9f| if_icmpeq| 比较栈顶两int型数值大小，当结果等于0时跳转 
|0xa0| if_icmpne| 比较栈顶两int型数值大小，当结果不等于0时跳转 
|0xa1| if_icmplt| 比较栈顶两int型数值大小，当结果小于0时跳转 
|0xa2| if_icmpge| 比较栈顶两int型数值大小，当结果大于等于0时跳转 
|0xa3| if_icmpgt| 比较栈顶两int型数值大小，当结果大于0时跳转 
|0xa4| if_icmple| 比较栈顶两int型数值大小，当结果小于等于0时跳转 
|0xa5| if_acmpeq| 比较栈顶两引用型数值，当结果相等时跳转 
|0xa6| if_acmpne| 比较栈顶两引用型数值，当结果不相等时跳转 
|0xa7| goto| 无条件跳转 
|0xa8| jsr| 跳转至指定16位offset位置，并将jsr下一条指令地址压入栈顶 
|0xa9| ret |返回至本地变量指定的index的指令位置（一般与jsr, jsr_w联合使用） 
|0xaa| tableswitch| 用于switch条件跳转，case值连续（可变长度指令） 
|0xab| lookupswitch |用于switch条件跳转，case值不连续（可变长度指令） 
|0xac| ireturn|从当前方法返回int 0xad lreturn 从当前方法返回long 
|0xae| freturn|从当前方法返回float 0xaf dreturn 从当前方法返回double 
|0xb0| areturn|从当前方法返回对象引用 0xb1 return 从当前方法返回void 
|0xb2| getstatic| 获取指定类的静态域，并将其值压入栈顶 
|0xb3| putstatic| 为指定的类的静态域赋值 
|0xb4| getfield| 获取指定类的实例域，并将其值压入栈顶 
|0xb5| putfield| 为指定的类的实例域赋值 
|0xb6| invokevirtual| 调用实例方法 
|0xb7| invokespecial| 调用超类构造方法，实例初始化方法，私有方法 
|0xb8| invokestatic| 调用静态方法 
|0xb9| invokeinterface| 调用接口方法 
|0xba| -- 0xbb| new 创建一个对象，并将其引用值压入栈顶 
|0xbc| newarray| 创建一个指定原始类型（如int, float, char…）的数组，并将其引用值压 入栈顶 
|0xbd| anewarray|创建一个引用型（如类，接口，数组）的数组，并将其引用值压入栈顶 
|0xbe| arraylength| 获得数组的长度值并压入栈顶 
|0xbf| athrow| 将栈顶的异常抛出 
|0xc0| checkcast| 检验类型转换，检验未通过将抛出ClassCastException 
|0xc1| instanceof| 检验对象是否是指定的类的实例，如果是将1压入栈顶，否则将0压入栈顶 
|0xc2| monitorenter| 获得对象的锁，用于同步方法或同步块 
|0xc3| monitorexit| 释放对象的锁，用于同步方法或同步块
|0xc4| wide |<待补充> 
|0xc5| multianewarray| 创建指定类型和指定维度的多维数组（执行该指令时，操作栈中必 须包含各维度的长度值），并将其引用值压入栈顶 
|0xc6| ifnull| 为null时跳转 
|0xc7| ifnonnull| 不为null时跳转 
|0xc8| goto_w| 无条件跳转（宽索引） 
|0xc9| jsr_w| 跳转至指定32位offset位置，并将jsr_w下一条指令地址压入栈顶