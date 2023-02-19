---
title: Window 开发环境配置
cover: 'https://fastly.jsdelivr.net/gh/prettywinter/dist/images/blogcover/工欲善其事.jpg'
categories:
  - Window
tags:
  - win
  - 环境配置
pin: true
abbrlink: a35e2b34
---

Window 开发环境配置整理。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

## 环境配置

Window 的环境变量配置非常简单。它分为用户变量和系统变量，配置系统变量使用任何用户登录系统都可以使用相关的命令；如果添加的是用户变量，那么只有添加变量的当前用户才能使用该命令，本文都以配置{% emp 系统变量 %}为例。

配置系统变量一般有两种方式：

- 直接在 {% kbd Path %} 里配置（一般配置到 bin 目录）。
- 新建一个系统变量，设置 key-value，之后在 {% kbd Path %} 中引用 {% kbd %key% %} 。

### 1. JDK

key: {% mark JAVA_HOME color:dark %}
value: {% mark D:\software\java color:dark %}
Path: {% mark %JAVA_HOME%\bin color:dark %}

key: {% mark CLASSPATH color:dark %}
value: {% mark .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar color:dark %}
Path: {% mark %JAVA_HOME%\bin color:dark %}
ps 测试安装是否成功：`java -version`

> **Tip：** jdk1.5以后CLASSPATH可以不用再进行设置;

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/变量设置01.png  %}
{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/变量设置02.png  %}

### 2. Maven

key: {% mark M2_HOME color:dark %}
value: {% mark D:\software\maven-3.8.6 color:dark %}
Path: {% mark %M2_HOME%\bin color:dark %}
ps 测试安装是否成功：`mvn -version`

[配置阿里源](https://developer.aliyun.com/mvn/guide)

```xml .../maven-3.8.6/maven/conf/setting.xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### 3. Gradle

key: {% mark GRADLE_HOME color:dark %}
value: {% mark D:\software\gradle-7.5 color:dark %}
Path: {% mark %GRADLE_HOME%\bin color:dark %}
ps 测试安装是否成功：`gradle -version`

配置下载源；

```gradle .../gradle-7.5/init.d/init.gradle
allprojects {
	repositories {
        // 本地仓库
        mavenLocal()
        // 镜像仓库
        maven { url 'https://maven.aliyun.com/repository/public/' }
        maven { url 'https://maven.aliyun.com/repository/spring/' }
        // maven 中央仓库
        mavenCentral()
    }
}
```

#### gradle加载顺序

```bash
# 排列顺序即加载顺序 * 代表可以为任意名称
~/.gradle/init.gradle
~/.gradle/init.d/*.gradle
GRADLE_HOME/init.d/*.gradle
GRADLE_USER_HOME/init.gradle
GRADLE_USER_HOME/init.d/*.gradle

# Gradle 查找依赖的顺序
# M2_HOME 是 Maven 的配置
~/.m2/settings.xml
M2_HOME/conf/settings.xml
~/.m2/repository
```

### 4. MySQL

使用安装文件（exe 或者 msi）安装一般没有什么问题，安装过程中会提示将安装路径加入环境变量。

这里只介绍使用 {% mark 压缩包 color:green %} 的方式。

key: {% mark MYSQL_HOME color:dark %}
value: {% mark D:\software\MySQL8.0 color:dark %}
Path: {% mark %MYSQL_HOME%\bin color:dark %}

管理员身份运行 cmd、ps、cmder 等工具：

```bash
# service-name 自己命名，如果不写默认为 mysql，只安装一个版本的话推荐不指定
# 在安装多个 MySQL 版本的时候很有用
mysqld --install service-name
# 移除 MySQL 服务
mysqld --remove service-name

# 使用不安全的初始化（即不生成初始密码）
mysqld --initialize-insecure --user=mysql --console
# 如果需要生成初始化密码，则使用以下命令。--console 选项把生成的密码打印到控制台
mysqld --initialize --user=mysql --console

# 启动 MySQL 服务
net start service-name
# 进入 MySQL bash
mysql -u root -p
# 修改密码
alter user 'root'@'localhost' identified by '';
```

### 5. MariaDB

依然使用 [压缩包](https://mariadb.org/download/?t=mariadb&p=mariadb&r=10.9.3&os=windows&cpu=x86_64&pkg=zip&m=blendbyte) 安装。

key: {% mark MARIA_HOME color:dark %}
value: {% mark D:\software\mariadb-10.9.3 color:dark %}
Path: {% mark %MARIA_HOME%\bin color:dark %}

管理员身份运行 cmd、shell 等命令行工具：

```bash D:\software\mariadb-10.9.3\bin
# 名称自己起
mysqld.exe --install mariadb<service-name>
# 初始化数据库，生成 data 文件夹以及配置信息
mysql_install_db
# 启动
net start mariadb(上面安装时起的名称)
```
配置文件（如果本机安装之前还有 MySQL 数据库，推荐解压后先修改配置文件，更改默认端口）：

```ini mariadb-10.9.3/data/my.ini
[mysqld]
datadir=D:/software/mariadb-10.9.3/data
port = 3333
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
skip-character-set-client-handshake

[client]
plugin-dir=D:\software\mariadb-10.9.3/lib/plugin
; 客户端连接端口
port=3333

[mariadb]
feedback=off
; 指定端口
port = 3333
; 默认创建数据库的类型
default-storage-engine = InnoDB
; 最大连接数
max_connections = 100
; 慢查询
slow_query_log_file = slow.log
slow_query_log  = 0
log_queries_not_using_indexes  = 1
long_query_time = 0.5
min_examined_row_limit = 100
```

> 如果本机已经安装了 MySQL 数据库，那么使用这种方式安装在初始化数据库后，应该先修改 my.ini 文件，修改端口号，然后启动服务。
> FEEDBACK 启动成功但是连接失败官方说明：https://mariadb.com/kb/en/feedback-plugin/
> 配置文档：https://mariadb.com/kb/en/configuring-mariadb-with-option-files/
> Unicode 排序规则：https://www.monolune.com/articles/mysql-utf8-charsets-and-collations-explained/

### 6. PostgreSQL

[压缩包下载](https://www.enterprisedb.com/download-postgresql-binaries)，这里选择了 15 版本。

Path：`D:\software\pgsql\bin`

管理员身份运行 cmd、shell 等命令行工具：

```bash D:\software\pgsql\bin
# 初始化 initdb --help 查看帮助信息
# -D data         指定初始化的数据库目录(此处为当前目录的data文件夹)
# -E utf8         数据库编码格式
# -U postgres     超级管理员名称
# -A              使用密码授权
initdb.exe -D "D:\software\pgsql\data" -E UTF8 -U postgres -W
# 注册服务 -N 指定服务名称
pg_ctl register -N "postgresql" -D "D:\software\pgsql\data"

# 启动
pg_ctl -D "D:\software\pgsql\data" -l logfile start
```

> 如果使用最新版本推荐使用官方提供的客户端 pgAdmin4，可在设置中调整为中文界面(缺点就是响应较慢)，Navicat 15.x 连接时报错 ”datlastsysoid“ 字段不存在，估计需要 Navicat 16 吧。
> pgsql 的配置文件在 data 文件夹中

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20221023164038.png pgAdmin4界面示例 %}

### 7. Tomcat

Tomcat 有安装版和压缩包两种，如果是安装版，需要先配置好jdk。

变量名：`CATALINA_HOME`
变量值示例：`D:\software\tomcat8.5`
Path：`%CATALINA_HOME%\bin`。

> Tomcat启动的日志输出的默认字符集编码是 utf-8，但是 window 的 shell 是 gbk 编码，所以如果想要在 window 下正常输出，就要修改 Tomcat 的日志打印编码。
> 编辑tomcat安装路径下的 `conf/logging.properties` 文件,找到以下内容并修改为：`java.util.logging.ConsoleHandler.encoding = GBK`，保存重启即可。

### 8. NodeJs

安装 NodeJs 后，Window 需要修改安装目录的权限，不然只有管理员才能对目录下的文件等进行操作，对于当前用户是非常不方便的。而且当你使用 npm 的时候会报错，提示没有权限。

在安装目录即 `nodejs` 文件夹，点击右键，选择属性，点击 “安全” 标签，选择 “User” 用户，点击编辑，把权限都勾上，保存退出。

```bash
# 配置阿里镜像源
npm config set registry https://registry.npmmirror.com
# 设置全局安装目录（需要先手动创建）
npm config set prefix "D:\software\nodejs\node_global"
# 设置全局缓存目录（需要先手动创建）
npm config set cache "D:\software\nodejs\node_cache"
# 查看修改是否成功
npm config ls -l
```

> 修改成功之后，在 path 中加入 `D:\software\nodejs\node_global` 路径。
> 这样，使用 **全局安装** 的工具（例如 yarn、pnpm 等）就可以在任何路径下使用了。

### 9. Jetty

{% link https://www.eclipse.org/jetty/documentation/jetty-11/operations-guide/index.html 官方文档 %}

下载后解压，之后进入 jetty 的安装目录。
运行以下命令(`$JETTY_HOME` 是安装目录)：

```shell
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

### 10. Go

key: `GOROOT`，
value：`D:\software\go`
key:`GOPATH`，
value:`D:\Projects\GoProjects`

Path：`%GOROOT%\bin`
检验：`go env`

使用 Vscode 安装 Go 插件的话，速度感人，可以参考：https://goproxy.io/zh/

### 11. [Rust](https://www.rust-lang.org/)

Rust 安装参考：https://course.rs/first-try/installation.html

替换下载依赖源：

```toml .../.cargo/config.toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
```

## 插件

### 1. 油猴插件

1. [网盘链接检查：(自动识别并标记百度云、蓝奏云、腾讯微云和天翼云盘的链接状态)](https://greasyfork.org/zh-CN/scripts/394216-%E7%BD%91%E7%9B%98%E9%93%BE%E6%8E%A5%E6%A3%80%E6%9F%A5)
2. [网盘直链下载助手](https://greasyfork.org/zh-CN/scripts/436446-%E7%BD%91%E7%9B%98%E7%9B%B4%E9%93%BE%E4%B8%8B%E8%BD%BD%E5%8A%A9%E6%89%8B)
3. [链接助手](https://greasyfork.org/zh-CN/scripts/422773-%E9%93%BE%E6%8E%A5%E5%8A%A9%E6%89%8B)
4. [Github 增强 - 高速下载](https://greasyfork.org/zh-CN/scripts/412245-github-%E5%A2%9E%E5%BC%BA-%E9%AB%98%E9%80%9F%E4%B8%8B%E8%BD%BD)
5. [CSDN/知乎/哔哩哔哩/简书免登录去除弹窗广告](https://greasyfork.org/zh-CN/scripts/428960-csdn-%E7%9F%A5%E4%B9%8E-%E5%93%94%E5%93%A9%E5%93%94%E5%93%A9-%E7%AE%80%E4%B9%A6%E5%85%8D%E7%99%BB%E5%BD%95%E5%8E%BB%E9%99%A4%E5%BC%B9%E7%AA%97%E5%B9%BF%E5%91%8A)

### 2. Edge

1. [可可翻译](https://microsoftedge.microsoft.com/addons/detail/%E5%8F%AF%E5%8F%AF%E7%BF%BB%E8%AF%91/ebkimaahhkeiplegpghijhgmlcdkeppf)
2. [JSON Formatter for Edge](https://microsoftedge.microsoft.com/addons/detail/json-formatter-for-edge/njpoigijhgbionbfdbaopheedbpdoddi)
3. [Greenhub绿墙—网络出海工具](https://microsoftedge.microsoft.com/addons/detail/greenhub%E7%BB%BF%E5%A2%99%E2%80%94%E7%BD%91%E7%BB%9C%E5%87%BA%E6%B5%B7%E5%B7%A5%E5%85%B7/hholdpohidinjmkoanabdchniingdfac)
4. [文件蜈蚣（需要安装对应的软件）](https://microsoftedge.microsoft.com/addons/detail/jeekmibdibcjeihbjnjgbegjalfelhon)
5. [图片下载](https://microsoftedge.microsoft.com/addons/detail/fatkun%E5%9B%BE%E7%89%87%E6%89%B9%E9%87%8F%E4%B8%8B%E8%BD%BD-pro/dammmokdamnimedflemdaoamhldmldff)
6. [Replace Google CDN](https://microsoftedge.microsoft.com/addons/detail/replace-google-cdn/cojepngjobmaiajphkijbdcdjnnjhpjc)

> 细滚动条设置（默认长方形）：打开 Edge，URL地址输入 “edge://flags”，进入页面搜索 “Windows 11 fluent scrollbars.”，点击选择框选择 Enabled，重启浏览器（仅限 win 平台）。

### 3. 火狐

1. [TWP - Translate Web Pages](https://addons.mozilla.org/zh-CN/firefox/addon/traduzir-paginas-web/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
2. [可可翻译](https://addons.mozilla.org/zh-CN/firefox/addon/sctranslator/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
3. [Browse.live](https://addons.mozilla.org/zh-CN/firefox/addon/browse-live/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)

### 4. IDEA

1. [Generate All Getter And Setter](https://plugins.jetbrains.com/plugin/18969-generate-all-getter-and-setter)
2. [Log Support 2](https://plugins.jetbrains.com/plugin/9417-log-support-2)
3. [MyBatisPlus](https://plugins.jetbrains.com/plugin/12670-mybatisplus)
4. Translation

> 推荐使用 2021.2.2 及以下版本。
