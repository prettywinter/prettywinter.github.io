---
title: Ubuntu安装Hadoop+Hive
categories: Linux
tags: [Ubuntu, hadoop, hive]
---



<!-- more -->

## 一、[官网集群配置文档](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)

{% link https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions 关于安装版本官方文档 %}

## 二、版本说明

推荐使用新版本（3.x），浪子因为所在公司使用的 2.x 版本，所以采用的 2.x 的最高版本。

java-8
hadoop-2.10.x
hive-2.3.9

## 三、安装

1. 安装 jdk

    ```bash
    $ sudo apt install openjdk-8-jdk
    # 查看路径（虽然使用 apt 安装不需要配置环境变量，但是 hadoop 无法识别，依然需要手动导入）
    # /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java 就是安装的路径
    # 如果你的 jdk 是自己上传压缩包配置的可以跳过此步骤
    $ which java
        /usr/bin/java
    $ file /usr/bin/java
        /usr/bin/java: symbolic link to /etc/alternatives/java
    $ file /etc/alternatives/java
        /etc/alternatives/java: symbolic link to /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
    $ file /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
        /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0d167f96ccfa5695cabf1b40056a77dc9bb8ab1a, for GNU/Linux 3.2.0, stripped
    
    # 下载 oracle 驱动
    cd hivepath/lib
    wget https://download.oracle.com/otn-pub/otn_software/jdbc/217/ojdbc8.jar
    ```

2. 配置环境变量：

    ```bash /etc/profile
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    export HADOOP_HOME=hadooppath
    export HIVE_HOME=hivepath
    export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
    ```

3. 检验 Hadoop 是否安装成功：

    ```bash
    source /etc/profile
    hadoop version
    ```

4. 修改 Hive 配置文件
cd .../hivepath/conf
cp hive-env.sh.template hive-env.sh

   ```bash hive-env.sh
    # 配置一下 Hadoop 的安装路径
    HADOOP_HOME=hadooppath
   ```

5. 添加 hive-site.xml 文件

新建 `hive-site.xml` 文件，添加以下内容。

    ```xml hive-site.xml
    <?xml version="1.0"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:oracle:thin:@*********:1521:fangweidbt</value>
    </property>
    
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>oracle.jdbc.OracleDriver</value>
    </property>
    
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>******</value>
    </property>
    
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>******</value>
    </property>
    </configuration>
    ```

6. 初始化

    ```bash
    schematool -dbType oracle -initSchema
    ```

然后就可以使用了。

```bash
hive
show databases;
```

## 三、Hadoop集群搭建

一般在生产环境中使用 Hadoop 都是集群部署（毕竟是大数据，数据集比较大），如果采用一台服务器存储，万一需要维护或者宕机就不能对外提供服务了。[官网集群配置文档](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)

### 1. 前置准备

- jdk(注意配置环境变量)：
{% link https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions 安装版本官方文档 %}
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
