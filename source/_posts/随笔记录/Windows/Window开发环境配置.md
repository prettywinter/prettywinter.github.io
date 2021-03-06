---
title: Window开发环境配置
categories: win
tags: [Win, 配置]
abbrlink: 6a7c9e96
---

Window开发环境的配置，主要是Windows平台，Linux非常简单，末尾给一个示例。如果你的 Linux 是 CentOS 或者 Ubantu，使用 rpm、deb 安装一般不用配置环境变量。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [环境配置](#环境配置)
  - [1. JDK](#1-jdk)
  - [2. Maven](#2-maven)
  - [3. Gradle](#3-gradle)
  - [4. MySQL数据库环境变量](#4-mysql数据库环境变量)
  - [5. Android studio](#5-android-studio)
  - [6. Tomcat](#6-tomcat)
  - [7. Node.js](#7-nodejs)
  - [8. Jetty](#8-jetty)
  - [9. Go](#9-go)
- [一些问题解决](#一些问题解决)
  - [1. MySQL](#1-mysql)
    - [1. Mysql5.7.20以及以上版本无法启动问题](#1-mysql5720以及以上版本无法启动问题)
    - [2. MySQL8.0安装](#2-mysql80安装)
- [附](#附)
  - [1.docker 镜像加速](#1docker-镜像加速)
  - [2.Vmware 双网卡配置，主要方便本机测试](#2vmware-双网卡配置主要方便本机测试)

<!-- /code_chunk_output -->

## 环境配置

> 配置环境变量时，安装路径最好不要有中文或者空格，因为我们也不清楚到底会在什么时候出问题，中文和空格也许就是潜在的危险。Nacos 启动的时候如果JDK的路径含有空格就会启动失败。
> 推荐开发使用 Linux。

### 1. JDK

新建系统变量名：`JAVA_HOME`
变量值示例(依据自己的jdk安装路径选择)：`D:\software\Java\jdk1.8.0_131`

JRE_HOME，和上面一样，示例
新建系统变量名：`JRE_HOME`
变量值：`D:\software\Java\jre1.8.0_131`
新建系统变量名：`CLASSPATH`
变量值: `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar`。注意最前面的英文小数点 `.`

> 注：jdk1.5以后CLASSPATH可以不用再进行设置;
> 配置系统变量即使用任何用户登录系统都可以使用相关的命令；如果添加的是用户变量，那么只有添加变量的当前用户才能使用该命令。

配置完成后，path里使用引用方式(%env_name%)配置JAVA_HOME和JRE_HOME到bin目录，例如`%JAVA_HOME%\bin`。最后打开cmd，输入 `java -version` 显示版本号代表安装成功。

![变量设置](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/变量设置01.png)
![变量设置](https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/变量设置02.png)

### 2. Maven

设置系统变量名：`M2_HOME`
变量值示例:`D:\software\apache-maven-3.3.9`
然后在path里配置到bin目录；打开cmd命令行 输入 `mvn –version` 显示版本号则maven配置成功

**[maven配置文件使用阿里源：](https://developer.aliyun.com/mvn/guide)**
编辑maven的安装路径下的 `conf/setting.xml` 文件，找到 `<mirror>` 标签添加：

```xml{.line-numbers}
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

> Maven 3.8.5 版本与 Idea2020.3.x 系列不兼容，在 Idea 中加载 Maven 项目时会抛出 Idea 内部没有相关方法的异常信息。
> Idea 2020.3.4 和 2021.2.2 版本最高可以使用 Maven3.8.4；

### 3. Gradle

设置系统变量名：`GRADLE_HOME`
变量值示例：`D:\software\Java\gradle-7.0`
检测配置是否成功： `gradle –v`

配置仓库下载源

```xml{.line-numbers}
<!-- 指定所使用的仓库， mavenCentral()表示使用中央仓库，此刻项目中所使用的jar包都会默认依次从仓库中下载到本地指定的目录 -->
allprojects {
    repositories {
        // 从本地仓库寻找依赖，如果过没有再执行下面的操作
        mavenLocal()
        // 如果只配置中央仓库，表示只从中央仓库下载jar包。但是如果指定下载位置已经有了，就不会再下载了。
        mavenCentral()
        maven {
            url 'https://maven.aliyun.com/repository/public/'
        }
    }
}
```

### 4. MySQL数据库环境变量

直接在path里配置绝对路径到bin目录即可。

当然也可以像上面一样配置，系统变量名： `MYSQL_HOME`，变量值：`D:\software\MySQL8.0`

### 5. Android studio

直接在path里设置sdk目录下的tools、platform-tools的绝对路径即可，示例`D:\Android\Sdk\platform-tools;`

### 6. Tomcat

Tomcat有安装版和压缩包两种，如果是安装版，需要先配置好jdk;不管如何，推荐先配置好 JDK。

新建系统变量名：`CATALINA_HOME`
变量值示例：`D:\software\tomcat8.5`
配置完成后在系统path里配置 `%CATALINA_HOME%\bin`。

> Tomcat启动的日志输出的默认字符集编码是 utf-8，但是window的控制台是gbk编码的，所以如果想要在window下正常输出，就要修改 Tomcat 的日志打印编码。
> 编辑tomcat安装路径下的 `conf/logging.properties` 文件,找到以下内容并修改为：`java.util.logging.ConsoleHandler.encoding = GBK`，保存重启即可。

{% link 参考::https://blog.csdn.net/u012964753/article/details/81045716 %}

### 7. Node.js

安装 nodejs 后，打开命令行，修改成国内的淘宝镜像源，这样下载速度快些：~~`npm config set registry https://registry.npm.taobao.org` 往期配置，过期不推荐~~

[官网最新配置方法](https://npmmirror.com/)：`npm config set registry https://registry.npmmirror.com`

nodejs 默认的模块安装是在 C 盘下的，我们可以进行配置修改，安装到其它位置。安装的目录需要自己提前建立好。
以下配置中的 `node_cache` 和 `node_global` 文件夹就是提前建好的，默认安装完 nodejs 后是没有这两个文件夹的。

- **修改全局安装路径：**
`npm config set prefix "D:\software\nodejs\node_global"`
- **修改全局缓存路径：**
- `npm config set cache "D:\software\nodejs\node_cache"`
**然后检查是否修改成功：**
`npm config ls -l`

修改成功后直接在环境变量 path 里加入 `D:\software\nodejs\node_global;`，加入这个环境变量后，使用 npm 命令全局安装后的命令就可以直接在 cmd 的任何路径使用了，例如`yarn` 、`hexo`、`vue-cli`等，否则会显示"XXX既不是内部命令，也不是外部命令"

### 8. Jetty

{% link 官网下载::https://www.eclipse.org/jetty/documentation.php %}

下载后解压，之后进入 jetty 的安装目录。
运行以下命令(`$JETTY_HOME` 是安装目录)：

```shell{.line-numbers}
# 为当前基目录添加标准文件及文件夹
java -jar $JETTY_HOME/start.jar --add-to-startd=http,deploy
# 创建demo演示，不然启动后是404页面
java -jar $JETTY_HOME/start.jar --add-module=demo
# 启动jetty服务
java -jar $JETTY_HOME/start.jar

# 如果不想用8080，可以使用以下命令在启动时动态修改
# 也可以修改配置文件，使其永久生效
java -jar $JETTY_HOME/start.jar jetty.http.port=9999
```

之后就可以在浏览器使用 `localhost:9999` 进行访问了。
{% link 官方文档::https://www.eclipse.org/jetty/documentation/jetty-11/operations-guide/index.html %}

### 9. Go

新建系统环境变量名 `GOROOT`，变量值为 go 的安装路径；
新建系统变量名 `GOPATH`，变量值为手动创建的 Go 工程目录。
在 Path 中加入 `%GOROOT%\bin`，保存。打开 cmd，输入 `go env` 查看配置信息。

使用 Vscode 安装 Go 插件的话，速度感人，可以参考：{% link VScode安装Go插件::https://goproxy.io/zh/ %}

## 一些问题解决

### 1. MySQL

官网下载安装版或者解压版，安装版一般不会有什么错误，环境变量如果安装的时候勾选加入 Path 中也不用手动配置了，比较简单，这里不说了。由于浪子比较喜欢解压版的，因为解压版的体积小，只包含了启动的服务，也方便安装。

以下问题假设均发生在安装配置好环境变量之后。

#### 1. Mysql5.7.20以及以上版本无法启动问题

网络上一大堆教程提到要设置my.ini，其实5.7.20不需要my.ini，设置my.ini反而导致MySQL启动失败且不报任何错误。可以直接打开cmd（管理员身份最好），进入MySQL的 bin 目录，执行以下命令：

```bash{.line-numbers}
# 安装 MySQL 服务
mysqld -install
# 初始化，建立date文件
mysqld --initialize-insecure --user=mysql
# 启动 MySQL 服务
net start mysql
# 进入 MySQL 的 bash 环境
mysql -u root -p

# 如果你有其它的设置，后面不想要了
# 那么可以移除 MySQL 服务
mysqld -remove
```

> **注意：** 如果安装 MySQL 服务失败或者命令无效，请确认是否是以管理员的身份运行的 cmd/shell/cmder 等软件。

#### 2. MySQL8.0安装

```bash{.line-numbers}
# 安装服务并启动
mysqld --install
mysqld --initialize-insecure --user=mysql
net start mysql
# 进入 MySQL 环境，初始无密码，直接回车就 ok
mysql -u root -p
# 修改密码为 123456
alter user 'root'@'localhost' identified by '123456';
```

如果连接 sqlyog 报错，进入到 MySQL 的 bash 环境(mysql -u root -p)，然后执行命令：`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

## 附

### 1.docker 镜像加速

Docker配置下载镜像加速(`vim /etc/docker/daemon.json`)，设置完毕重启。
1、Docker官方的中国镜像加速地址(不用注册)：https://registry.docker-cn.com  
2、中科大的镜像加速器(不用注册（推荐）)：https://docker.mirrors.ustc.edu.cn/
3、阿里云的镜像加速器(需要注册)：登录阿里云的容器hub服务，镜像加速器那一栏里会为你独立分配一个加速器地址
4、DaoCloud的镜像加速器(需要注册)：登录DaoCloud的加速器获取脚本，该脚本可以将加速器添加到守护进程的配置文件中

### 2.Vmware 双网卡配置，主要方便本机测试

VMWare 创建虚拟机时可以配置双网卡。打开网络适配器，可以看到 VMWare 有 VM1 和 VM8，NAT模式使用的是 VM8，而“仅主机”模式使用的 VM1，配置了双网卡，那么就可以把 VM1 的 IP 修改成自己想要的 IP，这样就可以与宿主机通信了，NAT 模式的网络再如何变也无妨了，因为我们使用的是另一个网络通道。
