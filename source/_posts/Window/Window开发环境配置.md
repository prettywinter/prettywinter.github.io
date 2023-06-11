---
title: Window 开发环境配置
cover: 'https://fastly.jsdelivr.net/gh/prettywinter/dist/images/blogcover/工欲善其事.jpg'
categories:
  - Windows
tags:
  - win
  - 配置
pin: true
abbrlink: a35e2b34
---

Windows 开发环境配置 + 软件配置。

<!-- more -->

## 一、环境配置

Windows 的环境变量配置比较简单，它是以键值对（key-value）的方式配置的。

在 Windows 系统中，有两种变量：用户变量和系统变量。

- 用户变量：只有当前登录用户能可以使用的变量，类似于 Linux 系统中用户目录下的 `.bashrc` 文件；

- 系统变量：对所有用户生效、所有用户可以使用的变量，类似于 Linux 系统中的 `/etc/profile` 文件；

本文都以配置 **系统变量** 为例，但是配置方式不限，配置的 `value` 路径要替换自己本机的安装路径。Windows 配置变量一般有两种方式：

1. 直接在 {% kbd Path %} 里编辑添加软件的执行文件所在的目录（一般是软件的 bin 目录）。
2. 新建一个变量，设置变量名称（一般随便起，部分有约定，比如 `JAVA_HOME`，很多需要 java 环境的软件默认查询 `JAVA_HOME` 变量名）和路径（安装软件的根目录）的 key-value，然后在 {% kbd Path %} 中引用 {% kbd %key% %} 。

### 1. JDK

第一个配置附带了截图，后面的就不带了 :wink:，只是帮助理解一下上面的第二种配置说明。第一种是比较简单的，直接把软件执行程序路径添加在 {% kbd Path %} 中。

| key       | value                                              |
| --------- | -------------------------------------------------- |
| JAVA_HOME | D:\software\java                                   |
| CLASSPATH | .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar |

Path 配置 `%JAVA_HOME%\bin`

测试安装是否成功：`java -version`

> **Tip：** jdk1.5 以后 CLASSPATH 可以不用再进行设置;

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/变量设置01.png  %}
{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/变量设置02.png  %}

### 2. Maven

Maven 官网下载特别慢，甚至打不开，可以在 https://archive.apache.org/dist/maven/maven-3/ 下载。

| key     | value                   |
| ------- | ----------------------- |
| M2_HOME | D:\software\maven-3.9.2 |


Path: `%M2_HOME%\bin`

测试安装是否成功：`mvn -version`

- [配置阿里下载源](https://developer.aliyun.com/mvn/guide)
 
也可以使用其它的加速源。打开 maven 的安装目录中的 `setting.xml` 文件（比如 `.../maven-3.9.2/conf/setting.xml`），找到 `<mirrors>` 标签，内部添加以下内容，保存退出。

```xml .../maven-3.9.2/conf/setting.xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### 3. [Gradle](https://gradle.org/)

| key              | value                   |
| ---------------- | ----------------------- |
| GRADLE_HOME      | D:\software\gradle-7.6  |
| GRADLE_USER_HOME | D:\software\gradle-repo |

Path: `%GRADLE_HOME%\bin`

- 配置下载源；

```gradle .../gradle-7.6/init.d/init.gradle
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

> Gradle 在 Windows 中非常推荐配置这两个，尤其是使用 IDEA2023 以下版本的；
> GRADLE_HOME 是本地 Gradle 的环境变量，和上面的类似；
> GRADLE_USER_HOME 内设置的路径不仅存放 gradle 下载的依赖路径（和 maven 本地仓库一样），以及编辑器读取并自动下载在 `gradle-wrapper.property` 文件设置的 gradle 版本（使用 idea 的应该都知道本机没有安装的会自动下载）。

- gradle加载顺序

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

> 由于 IDEA 中的 Gradle 经常自动下载，建议设置环境变量 `GRADLE_USER_HOME`，这样自动下载的 gradle 就会自动安装到这个目录。

### 4. MySQL

使用安装文件（exe 或者 msi）安装一般没有什么问题，安装过程中会提示将安装路径加入环境变量。

这里只介绍使用 {% mark 压缩包 color:green %} 的方式。

| key        | value                |
| ---------- | -------------------- |
| MYSQL_HOME | D:\software\MySQL8.0 |

Path: {% mark %MYSQL_HOME%\bin color:dark %}

管理员身份运行 cmd、ps、cmder 等工具：

```bash
# service-name 自己命名，如果不写默认为 mysql，只安装一个版本使用的话可以不指定
# 在安装多个不同版本 MySQL 的时候很有用
mysqld --install [service-name]
# 移除 MySQL 服务
mysqld --remove [service-name]

# 下面的命令二选一：分别是 无密码版 和 带密码版
# 使用不安全的初始化（即不生成初始密码）
mysqld --initialize-insecure --user=mysql --console
# 如果需要生成初始化密码，则使用以下命令。--console 选项把生成的密码打印到控制台（需要查看密码保存）
mysqld --initialize --user=mysql --console

# 启动 MySQL 服务
net start [service-name]
# 进入 MySQL bash
mysql -u root -p
# 修改密码
alter user 'root'@'localhost' identified by '';
```

> **TIP**：安装多个 MySQL 版本的时候，先 **新建/编辑** `my.ini` 文件先修改端口号再初始化（即生成 data 目录）。

### 5. [MariaDB](https://mariadb.org/download/?t=mariadb&p=mariadb&r=10.9.3&os=windows&cpu=x86_64&pkg=zip&m=blendbyte)

| key        | value                      |
| ---------- | -------------------------- |
| MARIA_HOME | D:\software\mariadb-11.0.2 |

Path: `%MARIA_HOME%\bin`

管理员身份运行 cmd、shell 等命令行工具：

```bash D:\software\mariadb-11.0.2\bin
# 名称自己起
mysqld.exe --install [service-name]
# 初始化数据库，生成 data 文件夹以及配置信息
mysql_install_db
# 启动
net start [service-name](上面安装时起的名称)
```
配置文件（如果本机安装之前还有 MySQL 数据库，推荐解压后先修改配置文件，更改默认端口）：

```ini .../mariadb-11.0.2/data/my.ini
[mysqld]
; 文件目录
datadir=D:/software/mariadb-11.0.2/data
port = 3333
; 字符集设置
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
skip-character-set-client-handshake

[client]
plugin-dir=D:/software/mariadb-11.0.2/lib/plugin
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

> 如果本机已经安装了 MySQL 数据库，那么使用这种方式安装需要在初始化数据库后，先修改 `my.ini` 文件，修改端口号避免端口占用导致启动失败，然后启动服务。
> FEEDBACK 启动成功但是连接失败官方说明：https://mariadb.com/kb/en/feedback-plugin/
> 官方配置文档：https://mariadb.com/kb/en/configuring-mariadb-with-option-files/
> Unicode 排序规则：https://www.monolune.com/articles/mysql-utf8-charsets-and-collations-explained/

### 6. [PostgreSQL](https://www.enterprisedb.com/download-postgresql-binaries)

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

> Navicat 15.x 连接时报错 ”datlastsysoid“ 字段不存在，估计需要 Navicat 16 吧；
> 如果使用最新版本推荐使用官方提供的客户端 pgAdmin4，可在设置中调整为中文界面(缺点就是响应较慢)，推荐使用开源的 Dbeaver；
> pgsql 的配置文件在 data 文件夹中。

{% image https://fastly.jsdelivr.net/gh/prettywinter/dist/images/doc/20221023164038.png pgAdmin4界面示例 %}

### 7. Tomcat

Tomcat 有安装版和压缩包两种，如果是安装版，需要先配置好jdk。

| key           | value                 |
| ------------- | --------------------- |
| CATALINA_HOME | D:\software\tomcat8.5 |

Path：`%CATALINA_HOME%\bin`。

> Tomcat启动的日志输出的默认字符集编码是 utf-8，但是 window 的 shell 是 gbk 编码，所以如果想要在 window 下正常输出，就要修改 Tomcat 的日志打印编码。
> 编辑tomcat安装路径下的 `conf/logging.properties` 文件,找到以下内容并修改；
> `java.util.logging.ConsoleHandler.encoding = GBK`，保存重启即可。

### 8. [NodeJs](https://nodejs.org/en)

Windows 中安装的 `nodejs.msi` 中自带了 `npm`，而 Linux 需要单独安装（使用包管理器的时候）。另外，Windos 中安装后需要修改安装目录的权限，不然只有管理员才能操作该目录，对于当前用户是非常不方便的。

在安装目录即 `nodejs` 文件夹，单击右键，选择属性，点击 “安全” 标签，选择 “User” 用户，点击编辑，把权限都勾上，保存退出。

打开自己的 bash 命令行，使用以下命令配置以下 `npm` 的安装、缓存路径以及使用的镜像源（这里使用的阿里的）。

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

Path：`D:\software\nodejs\node_global`

> 配置 Path 是为了使用 **全局安装** 的工具（例如 yarn、pnpm 等）。

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

### 10. [Rust](https://www.rust-lang.org/)

Rust 安装参考语言圣经：https://course.rs/first-try/installation.html

替换下载依赖源：

```toml .../.cargo/config.toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
```

### 11. [Python](https://www.python.org/downloads/)

```bash
# 设置镜像源
python -m pip install --upgrade pip
pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple

# 代码格式化 black
python -m pip install -U black
```

> black Github 地址：https://github.com/psf/black

## 二、软件

### 1. VsCode

VsCode 使用的频率只次于写代码，平常写文章、前端还有其它的场景使用较多，浪子喜欢使用 压缩包（俗称绿色、便携版）的方式安装。安装简便，安装后可以指定插件的位置：在安装目录新建 **data** 文件夹，这样安装插件时，VsCode 会自动把插件下载到此目录。升级也是非常的简单，下载最新版本的解压，把旧版的 **data** 文件夹移动到新版本根目录中，删除旧版本的即可。

VsCode 1.79 下载地址：
{% copy https://vscode.cdn.azure.cn/stable/b380da4ef1ee00e224a15c1d4d9793e27c2b6302/VSCode-win32-x64-1.79.0.zip %}

注意这里使用的的 cdn 加速，VsCode 下载是比较慢的，原下载链接前缀由 `https://az764295.vo.msecnd.net` 改为 `https://vscode.cdn.azure.cn` 即可加速。

## 三、插件

插件在精不在多。

### 1. 油猴插件

1. [网盘链接检查：(自动识别并标记百度云、蓝奏云、腾讯微云和天翼云盘的链接状态)](https://greasyfork.org/zh-CN/scripts/394216-%E7%BD%91%E7%9B%98%E9%93%BE%E6%8E%A5%E6%A3%80%E6%9F%A5)
2. [网盘直链下载助手](https://greasyfork.org/zh-CN/scripts/436446-%E7%BD%91%E7%9B%98%E7%9B%B4%E9%93%BE%E4%B8%8B%E8%BD%BD%E5%8A%A9%E6%89%8B)
3. [链接助手](https://greasyfork.org/zh-CN/scripts/422773-%E9%93%BE%E6%8E%A5%E5%8A%A9%E6%89%8B)
4. [Github 增强 - 高速下载](https://greasyfork.org/zh-CN/scripts/412245-github-%E5%A2%9E%E5%BC%BA-%E9%AB%98%E9%80%9F%E4%B8%8B%E8%BD%BD)
5. [CSDN/知乎/哔哩哔哩/简书免登录去除弹窗广告](https://greasyfork.org/zh-CN/scripts/428960-csdn-%E7%9F%A5%E4%B9%8E-%E5%93%94%E5%93%A9%E5%93%94%E5%93%A9-%E7%AE%80%E4%B9%A6%E5%85%8D%E7%99%BB%E5%BD%95%E5%8E%BB%E9%99%A4%E5%BC%B9%E7%AA%97%E5%B9%BF%E5%91%8A)

### 2. Edge

1. [可可翻译](https://microsoftedge.microsoft.com/addons/detail/%E5%8F%AF%E5%8F%AF%E7%BF%BB%E8%AF%91/ebkimaahhkeiplegpghijhgmlcdkeppf)
2. [沉浸式翻译](https://microsoftedge.microsoft.com/addons/detail/%E6%B2%89%E6%B5%B8%E5%BC%8F%E7%BF%BB%E8%AF%91/amkbmndfnliijdhojkpoglbnaaahippg)
3. [JSON Beautifier & Editor](https://microsoftedge.microsoft.com/addons/detail/json-beautifier-editor/kpfjmbnldfjpgbglcdcmlkekajbcjlem)
4. [AdGuard 广告拦截器](https://microsoftedge.microsoft.com/addons/detail/adguard-%E5%B9%BF%E5%91%8A%E6%8B%A6%E6%88%AA%E5%99%A8/pdffkfellgipmhklpdmokmckkkfcopbh)
5. [Greenhub绿墙—网络出海工具](https://microsoftedge.microsoft.com/addons/detail/greenhub%E7%BB%BF%E5%A2%99%E2%80%94%E7%BD%91%E7%BB%9C%E5%87%BA%E6%B5%B7%E5%B7%A5%E5%85%B7/hholdpohidinjmkoanabdchniingdfac)
6. [文件蜈蚣（需要安装对应的软件）](https://microsoftedge.microsoft.com/addons/detail/jeekmibdibcjeihbjnjgbegjalfelhon)
7. [图片下载](https://microsoftedge.microsoft.com/addons/detail/fatkun%E5%9B%BE%E7%89%87%E6%89%B9%E9%87%8F%E4%B8%8B%E8%BD%BD-pro/dammmokdamnimedflemdaoamhldmldff)
8. [Replace Google CDN](https://microsoftedge.microsoft.com/addons/detail/replace-google-cdn/cojepngjobmaiajphkijbdcdjnnjhpjc)

> 细滚动条设置（默认长方形）：打开 Edge，URL地址输入 “edge://flags”，进入页面搜索 “Windows 11 fluent scrollbars.”，点击选择框选择 Enabled，重启浏览器（仅限 win 平台）。

### 3. 火狐

1. [沉浸式翻译](https://addons.mozilla.org/zh-CN/firefox/addon/immersive-translate/?utm_content=addons-manager-reviews-link&utm_medium=firefox-browser&utm_source=firefox-browser)
2. [可可翻译](https://addons.mozilla.org/zh-CN/firefox/addon/sctranslator/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
3. [Browse.live](https://addons.mozilla.org/zh-CN/firefox/addon/browse-live/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
4. [Circle 阅读助手基础版](https://addons.mozilla.org/zh-CN/firefox/addon/circle-reader-mini/?utm_content=addons-manager-reviews-link&utm_medium=firefox-browser&utm_source=firefox-browser)

> 火狐新版本默认在当前标签页中选择收藏夹的标签单击打开会覆盖当前页；
> 浪子还是喜欢以前的方式：点击打开新标签页
> 地址栏输入 `about:config` 回车，搜索 `browser.search.openintab`、`browser.urlbar.openintab`、`browser.tabs.loadBookmarksInTabs` 三个选项的值都设置为 true 即可。

### 4. IDEA

1. [Generate All Getter And Setter](https://plugins.jetbrains.com/plugin/18969-generate-all-getter-and-setter)
2. [Log Support 2](https://plugins.jetbrains.com/plugin/9417-log-support-2)
3. Translation

> 推荐使用 2021.2.2 及以下版本。

## 四、其它

### 1. Git-Bash 使用 [tree](https://gnuwin32.sourceforge.net/packages/tree.htm) 命令

下载 [tree](https://gnuwin32.sourceforge.net/packages/tree.htm) 的压缩包版的二进制（Binaries）文件并解压。

进入解压的目录中，把 `bin` 目录下的 `tree.exe` 复制到 Git 的安装路径中的 `xxx\Git\usr\bin\` 下。

如果想要在 git-bash 中使用 `ll` 命令，可以在 `C:\Users\yourname` 下创建新文件 `.bashrc`，有的话就不用创建了，在该文件中加入以下内容保存退出：

```bash .bashrc
alias ll='ls -al'
```

### 2. 命令操作

| 目标     | 操作                             |
| -------- | -------------------------------- |
| 控制面板 | win + r 输入 `appwiz.cpl` 回车   |
| 系统变量 | win + r 输入 `sysdm.cpl` 回车    |
| 磁盘管理 | win + r 输入 `diskmgmt.msc` 回车 |
| 服务     | win + r 输入 `services.msc` 回车 |
| 电脑信息 | win + r 输入 `winver` 回车       |

### 3、删除开机多余启动项

{% kbd Win %} + {% kbd R %}，输入 `msconfig`，点击确定，选择 “引导” 标签，选择 **非当前OS** 记录，点击删除并应用，按照提示重启即可。

管理员方式进入 ps：

```bash
# 查看启动项
efibootmgr
# 删除对应的项
efibootmgr -b 0001(序号) -B
```

### 4. PowerShell 运行脚本报错

管理员身份运行 ps，

```shell
# 输入以下命令，选择 Y
set-executionpolicy RemoteSigned
```

ps 有四种策略：

| 策略         | 说明                                                                                   |
| ------------ | -------------------------------------------------------------------------------------- |
| Restricted   | 禁止运行任何脚本配置文件（默认）                                                       |
| AllSigned    | 可以运行发布签名的脚本和配置文件，包括本地计算机上编写的脚本                           |
| RemoteSigned | 要求从网络上下载的脚本和配置文件由可信者发布签名，不要求本地计算机上编写的脚本进行签名 |
| Unrestricted | 可以运行未签名的脚本                                                                   |
