---
title: Hadoop
categories: skill
tags: [Hadoop, 大数据]
---

Hadoop 简单了解。[官网集群配置文档](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)

<!-- more -->

## 一、认识

### 1. HDFS

Hadoop Distributed File System：Hadoop 分布式文件系统，用来存储数据的。


### 2. Mapreduce

思想：先分再合，分而治之（拆分 -> 求解 -> 合并）
不可拆分的计算任务或相互之间有依赖关系的数据无法进行并行计算。

MapReduce 借鉴了函数式语言的思想，用 Map 和 Reduce 两个函数提供了高层的并行编程抽象模型。
Map：对一组数据元素进行某种重复式处理；
Reduce：对 Map 的中间结果进行某种进一步的结果整理。

得益于 MapReduce 分布式计算框架的优秀设计，我们在使用的时候不需要关注底层实现细节，只需要关注业务实现，可以轻松编写分布式应用程序，这些应用程序以可靠、容错的方式并行处理大数据集。

一个完整的 MapReduce 程序在分布式运行时有三类：
MRAppMaster（一个）：负责整个 MR 程序过程调度及状态协调
MapTask：负责 map 阶段的整个数据处理流程
ReduceTask：负责 reduce 阶段的整个数据流程处理

一个 MapReduce 编程模型中只能包含一个 Map 阶段和一个 Reduce 阶段，或者只有 Map 阶段。不能有多个 map 阶段、多个 reduce 阶段的情景出现。
如果业务逻辑非常复杂，那就只能多个 MapReduce 程序串行运行。

整个 MapReduce 程序中，数据都是以 `键值对` 的形式传输的。在实际编程中，需要考虑每个阶段的 kv 输出是什么。

> MapReduce 的实时计算性能差，不能进行流式处理。
> 因此对于要求实时处理、响应快的数据一般使用 Spark、Flink 等其它数据计算工具。

### 3. YARN

{% image https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20221004175027.png YARN架构图 %}

全称 Yet Another Resource Negotiator，另一种资源协调者，是一种新的 Hadoop 资源管理器。
YARN 是一个通用资源管理系统和调度平台，可为上层应用提供统一的资源管理和调度。它的引入为集群在利用率、资源统一管理和数据共享等方面带来了巨大好处。

ResourceManager（RM）
YARN 集群中的主角色，决定系统中所有应用程序之间资源分配的最终权限，即最终仲裁者。接收用户的作业提交，并通过 NM 分配，管理各个机器上的计算资源。

NodeManager（NM）
YARN 中的从角色，一台机器上一个，负责管理本机器上的计算资源。根据 RM 命令，启动 Container 容器、监视容器的资源使用情况。并且向 RM 汇报资源使用情况。

ApplicationMaster（AM）
用户提交的每个应用程序均包含一个 AM。应用程序内的 “老大”，负责程序内部各阶段的资源申请，监督程序的执行情况。

当用户向 YARN 中提交一个应用程序后，YARN 将分两个阶段运行该应用程序。
第一个阶段是客户端申请资源启动运行本次程序的 `ApplicationMaster`；
第二个阶段是由 `ApplicationMaster` 根据本次程序内部具体情况，为它申请资源，并监控它的整个运行过程，知道运行完成。

MR 向 YARN 交互流程：
1. 客户端向 YARN 中 ResourceManager 提交应用程序；
2. RM 为该应用程序分配第一个 Container，并与对应的 NodeManager 通信，要求它在这个 Container 中启动这个应用程序的 ApplicationMaster。
3. AM 启动成功之后，首先向 RM 注册并保持通信，这样用户可以直接通过 RM 查看应用的运行状态。
4. AM 为本次程序内部的各个 Task 任务向 RM 申请资源，并监控它的运行状态。
5. AM 申请到资源后，便与对应的 NM 通信，要求它启动任务。
6. NM 为任务设置好运行环境后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务。
7. 各个任务通过某个 RPC 协议向 ApplicationMaster 汇报自己的状态和进度，以让 ApplicationMaster 随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。在应用程序运行过程中，用户可以随时通过 RPC 向 AM 查询应用程序的当前运行状态。
8. 应用程序执行完成后，AM 向 RM 注销并关闭自己。

## 二、命令

Hadoop 支持多种文件系统，包括本地文件(file:///)、分布式文件系统(hdfs://hostname:port) 等。
具体操作的是什么文件系统取决于命令中文件路径的前缀协议。如果没有指定前缀，则会读取环境变量中的 fs.defaultFS 属性。

```bash
hadoop fs -ls file:///
hadoop fs -ls hdfs://hostname:port/
hadoop fs -ls gfs://
# 直接使用 / 读取 fs.defaultFS 属性
hadoop fs -ls /
```

> hadopp dfs 只能操作 HDFS 文件系统，已经过期。
> hdfs dfs 只能操作 HDFS 文件系统相关，常用。
> hadoop fs 可操作任意文件系统，使用范围更广。官方推荐。
> 官方命令文档：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/CommandsManual.html

## 三、集群搭建

一般在生产环境中使用 Hadoop 都是集群部署（毕竟是大数据，数据集比较大），如果采用一台服务器存储，万一需要维护或者宕机就不能对外提供服务了。

### 1. 前置准备

- jdk(注意配置环境变量)：
{% image https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions 安装版本官方文档 %}
- ssh

### 2. 开始

1. 创建服务器地址映射

    ```bash /etc/hosts
    ...

    # ip hostname 域名
    192.168.10.1 node1 test.com
    ```

2. SSH 配置

    ```bash
    # 密钥生成省略，使用 ssh-key
    # 复制公钥到目标主机
    ssh-copy-id hostname
    ```

3. 集群时间同步

    ```bash
    sudo apt install ntpdate
    ntpdate cn.pool.ntp.org
    ```
    > ntp 常用服务器：https://dns.icoa.cn/ntp/

4. 安装 Hadoop

   ```bash
   # 官网下载安装包并解压
   # 复制到多台机器
   scp -r /sourcepath/ root@hostname:/targetpath/
   ```
5. 修改配置文件
配置文件都在 Hadoop 安装目录的 etc/hadoop/ 下。

   ```bash hadoop-3.3.4/etc/hadoop/hadoop-env.sh
   export JAVA_HOME=/path
   export HDFS_NAMENODE_USER=
   export HDFS_DATANODE_USER=
   export HDFS_SECONDARYNAMENODE_USER=
   export HDFS_RESOURCEMANAGER_USER=
   export HDFS_NODEMANAGER_USER=
   ```

    ```xml core-site.xml
    <!-- 默认使用的文件系统，支持 file、HDFS、GFS、Amazon等 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ip/name:port</value>
    </property>
    <!-- Hadoop 本地保存数据路径 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/path</value>
    </property>

    <!-- 设置 HDFS Web UI 用户身份 -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>user</value>
    </property>

    <!-- hive 用户代理 -->
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>

    <!-- 文件系统垃圾桶保存时间 -->
    <property>
        <name>fs.trash.interval</name>
        <value>1440</value>
    </property>
    ```

   ```xml hdfs-site.xml
   <!-- SNN 进程运行机器位置信息 -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>ip/hostname:port</value>
    </property>
   ```

    ```xml mapred-site.xml
    <!-- MR 程序默认运行模式，yarn：集群模式 local：本地模式 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <!-- MR 程序历史服务器地址 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>ip/hostname:port</value>
    </property>

    <!-- MR 程序历史服务器 web 地址 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>ip/hostname:port</value>
    </property>

    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>

    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>

    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    ```

    ```xml yarn-site.xml
    <!-- yarn 集群主角色运行机器位置 -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>ip/hostname</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 是否对容器实施物理内存限制 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>

    <!-- 是否对容器实施虚拟内存限制 -->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>

    <!-- 开启日志 -->
    <property>
        <name>yarn.log.aggregation-enabled</name>
        <value>true</value>
    </property>

    <!-- yarn 历史服务器地址 -->
    <property>
        <name>yarn.log.server.url</name>
        <value>http://ip|name/jobhistory/logs</value>
    </property>

    <!-- 历史日志保存时间 7 天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
    ```

   ```bash workers
   域名
   ```

6. 集群启动

首次运行初始化：`hdfs namenode -format`
