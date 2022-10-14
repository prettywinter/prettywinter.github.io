---
title: Window环境配置
categories: windows
tags: [配置]
cover: https://raw.githubusercontents.com/prettywinter/dist/main/images/blogcover/工欲善其事.jpg
# 置顶
pin: true
---

工欲善其事必先利其器，整理一下环境配置。

<!-- more -->

## 一、Win环境变量配置

以下环境变量都是 Win 平台的。配置环境变量时，安装路径最好不要有中文或者空格，因为我们也不清楚到底会在什么时候出问题，中文和空格也许就是潜在的危险。Nacos 启动的时候如果JDK的路径含有空格就会启动失败。推荐开发使用 Linux。

由于环境变量的配置都差不多，为了缩减篇幅，这里说一下面的配置方式：1. 直接配置安装路径的bin目录到系统 Path 里；2. 新建系统变量，设置变量名，变量值为安装路径，然后在系统 Path 里添加 `%env_name%\bin`。

浪子配置的都是 `系统变量`，用户变量配置后只能当前用户使用。

### 1. JDK

变量名：`JAVA_HOME`
变量值(依据自己的jdk安装路径选择)：`D:\software\Java\jdk1.8.0_131`
Path: `%JAVA_HOME%\bin`

变量名：`JRE_HOME`
变量值示例：`D:\software\Java\jre1.8.0_131`
Path: `%JRE_HOME%\bin`

变量名：`CLASSPATH`
变量值: `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar`。注意最前面的英文小数点 `.`

检验：`java -version`

> **注：** jdk1.5以后CLASSPATH可以不用再进行设置;
> 配置系统变量即使用任何用户登录系统都可以使用相关的命令；如果添加的是用户变量，那么只有添加变量的当前用户才能使用该命令。

{% image https://raw.sevencdn.com/prettywinter/dist/main/images/doc/变量设置01.png  %}
{% image https://raw.sevencdn.com/prettywinter/dist/main/images/doc/变量设置02.png  %}

### 2. Maven

变量名：`M2_HOME`
变量值示例:`D:\software\apache-maven-3.3.9`
Path: `%M2_HOME%\bin`
检验：`mvn –version`

**[maven配置文件使用阿里源：](https://developer.aliyun.com/mvn/guide)** 编辑maven的安装路径下的 `conf/setting.xml` 文件，找到 `<mirror>` 标签添加：

```xml .../maven/conf/setting.xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### 3. Gradle

变量名：`GRADLE_HOME`
变量值示例：`D:\software\Java\gradle-7.0`
Path: `%GRADLE_HOME%\bin`
检测： `gradle –v`

#### 3.1 配置全局下载源

在安装目录中（D:\software\Java\gradle-7.0）新建 `init.gradle` 文件，添加以下内容，保存退出。

```xml .../gradle-7.0/init.gradle
allprojects {
	repositories {
        <!-- 首选本地 maven 仓库，没有再下载 -->
        mavenLocal()
        <!-- 阿里云镜像 -->
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        <!-- maven 中央仓库 -->
        mavenCentral()
    }
}
```

#### 3.2 说明

`init.gradle` 相当于 maven 中的 `settings.xml`，因此它是有加载顺序的：

```bash
# 顺序查找，* 代表任意名称
~/.gradle/init.gradle
~/.gradle/init.d/*.gradle
GRADLE_HOME/init.d/*.gradle
GRADLE_USER_HOME/init.gradle
GRADLE_USER_HOME/init.d/*.gradle
```

Gradle查找本地仓库位置的过程：

```bash
# M2_HOME 是 Maven 的环境变量名称
~/.m2/settings.xml
M2_HOME/conf/settings.xml
~/.m2/repository
```

### 4. MySQL

官网下载安装版或者解压版，安装版一般不会有什么错误，环境变量如果安装的时候勾选加入 Path 中也不用手动配置了，比较简单，这里不说了。浪子比较喜欢解压版的，因为解压版的体积小，只包含了启动的服务，安装也是很方便的。

两种方式
- 直接在 Path 里加入目录：`D:\software\MySQL8.0\bin`，
- 或者变量名： `MYSQL_HOME`，变量值示例：`D:\software\MySQL8.0`，Path 添加 `%MYSQL_HOME%\bin`。

最好以管理员的身份运行 `cmd/powershell/cmder/gitbash` 等命令行工具（否则会出现安装 MySQL 服务失败或者命令无效）：

```bash
# 安装 MySQL 服务，service-name 可选，不写默认 mysql，安装多个版本时可以使用不同服务名称。
# 安装一个版本推荐不写 service-name
mysqld --install service-name
# 使用不安全的初始化（不生成初始密码），建立 data 文件夹。
# 如果需要生成初始化密码，使用 mysqld --initialize --console
# --console 在控制台打印
mysqld --initialize-insecure --user=mysql --console

# 启动 MySQL 服务
net start mysql
# 进入 MySQL 的 bash 环境
mysql -u root -p

# 修改密码为 123456
alter user 'root'@'localhost' identified by '123456';


# 如果你想重新配置，手动删除 MySQL 安装目录的 data 文件
# 然后移除 MySQL 服务
mysqld --remove
```

如果连接 sqlyog 报错，进入到 MySQL 的 bash 环境(mysql -u root -p)，然后执行命令：`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

### 5. Android studio

直接在path里设置sdk目录下的tools、platform-tools的绝对路径即可，示例`D:\Android\Sdk\platform-tools;`

### 6. Tomcat

Tomcat有安装版和压缩包两种，如果是安装版，需要先配置好jdk；不管如何，推荐先配置好 JDK。

变量名：`CATALINA_HOME`
变量值示例：`D:\software\tomcat8.5`
Path：`%CATALINA_HOME%\bin`。

> Tomcat启动的日志输出的默认字符集编码是 utf-8，但是window的控制台是gbk编码的，所以如果想要在window下正常输出，就要修改 Tomcat 的日志打印编码。
> 编辑tomcat安装路径下的 `conf/logging.properties` 文件,找到以下内容并修改为：`java.util.logging.ConsoleHandler.encoding = GBK`，保存重启即可。

{% link https://blog.csdn.net/u012964753/article/details/81045716 参考 %}

### 7. Node.js

安装 nodejs 后，打开命令行，修改成国内的淘宝镜像源，这样下载速度快些：~~`npm config set registry https://registry.npm.taobao.org` 过期配置不推荐~~

[最新配置命令](https://npmmirror.com/)：`npm config set registry https://registry.npmmirror.com`

nodejs 默认的模块安装是在 C 盘下的，我们可以进行配置修改，安装到其它位置。安装的目录需要自己提前建立好。
以下配置中的 `node_cache` 和 `node_global` 文件夹就是提前建好的，默认安装完 nodejs 后是没有这两个文件夹的。

- **修改全局安装路径：**
{% copy width:max npm config set prefix \"D:\\software\\nodejs\\node_global\" %}
- **修改全局缓存路径：**
{% copy width:max npm config set cache \"D:\\software\\nodejs\\node_cache\" %}

**检查是否修改成功：** `npm config ls -l`

修改成功后直接在环境变量 path 里加入 `D:\software\nodejs\node_global;`，加入这个环境变量后，使用 npm 命令全局安装后的命令就可以直接在 cmd 的任何路径使用了，例如`yarn` 、`hexo`、`vue-cli`等，否则会显示"XXX既不是内部命令，也不是外部命令"

### 8. Jetty

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

### 9. Go

变量名 `GOROOT`，
变量值示例：`D:\software\go`
变量名 `GOPATH`，
变量值示例：`D:\Projects\GoProjects`

Path：`%GOROOT%\bin`
检验：`go env` 查看配置信息。

使用 Vscode 安装 Go 插件的话，速度感人，可以参考：https://goproxy.io/zh/

### 10. [Rust](https://www.rust-lang.org/)

先安装 [MSVC - VS 2019 C++ x64/x86 build tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)。安装时可自行修改缓存路径与安装路径，避免占用过多 C 盘空间。安装完成后，Rust 所需的 msvc 命令行程序需要手动添加到环境变量中，否则安装 Rust 时 rustup-init 会提示未安装 Microsoft C++ Build Tools，其位于：`%Visual Studio 安装位置%\VC\Tools\MSVC\%version%\bin\Hostx64\x64`（请自行替换其中的 %Visual Studio 安装位置%、%version% 字段）。

`rust-init.exe` 安装工具会自动尝试加入环境变量。

Linux 可以使用如下脚本安装。

```bash
# Linux 安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

cargo 下载依赖换为国内源：

```toml xxx/.cargo/config.toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
```

### 11. MariaDB

这里使用[压缩包](https://mariadb.org/download/?t=mariadb&p=mariadb&r=10.9.3&os=windows&cpu=x86_64&pkg=zip&m=blendbyte)安装。

变量名 `MARIA_HOME`，
变量值示例：`D:\software\mariadb-10.9.3`
Path：`%MARIA_HOME%\bin`

以管理员身份运行 cmd、shell 等命令行工具：

```bash
# 名称自己起
mysqld.exe --install mariadb
# 初始化数据库，生成 data 文件夹以及配置信息
mysql_install_db
# 启动
net start mariadb(如果上面安装的时起的名称)
```

```ini mariadb-10.9.3
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

## 二、插件

### 1. 油猴插件

1. [网盘链接检查：(自动识别并标记百度云、蓝奏云、腾讯微云和天翼云盘的链接状态)](https://greasyfork.org/zh-CN/scripts/394216-%E7%BD%91%E7%9B%98%E9%93%BE%E6%8E%A5%E6%A3%80%E6%9F%A5)
2. [网盘直链下载助手](https://greasyfork.org/zh-CN/scripts/436446-%E7%BD%91%E7%9B%98%E7%9B%B4%E9%93%BE%E4%B8%8B%E8%BD%BD%E5%8A%A9%E6%89%8B)
3. [链接助手](https://greasyfork.org/zh-CN/scripts/422773-%E9%93%BE%E6%8E%A5%E5%8A%A9%E6%89%8B)
4. [Github 增强 - 高速下载](https://greasyfork.org/zh-CN/scripts/412245-github-%E5%A2%9E%E5%BC%BA-%E9%AB%98%E9%80%9F%E4%B8%8B%E8%BD%BD)
5. [CSDN/知乎/哔哩哔哩/简书免登录去除弹窗广告](https://greasyfork.org/zh-CN/scripts/428960-csdn-%E7%9F%A5%E4%B9%8E-%E5%93%94%E5%93%A9%E5%93%94%E5%93%A9-%E7%AE%80%E4%B9%A6%E5%85%8D%E7%99%BB%E5%BD%95%E5%8E%BB%E9%99%A4%E5%BC%B9%E7%AA%97%E5%B9%BF%E5%91%8A)

### 2. Firefox

1. [AdGuard 广告拦截器](https://addons.mozilla.org/zh-CN/firefox/addon/adguard-adblocker/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
2. [Tampermonkey 油猴](https://addons.mozilla.org/zh-CN/firefox/addon/tampermonkey/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
3. [Violentmonkey 暴力猴](https://addons.mozilla.org/zh-CN/firefox/addon/violentmonkey/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
4. [网页翻译](https://addons.mozilla.org/zh-CN/firefox/addon/traduzir-paginas-web/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
5. [可可翻译](https://addons.mozilla.org/zh-CN/firefox/addon/sctranslator/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)

> 目前火狐扩展商店有关去广告的扩展中国地区的用户无法使用
> **PS：** 如果使用油猴扩展安装插件有问题，可以使用暴力猴扩展，安装插件的方式还是一样。 ~~（不知道是 win10 的原因还是 Firefox 升级的原因，不过换个扩展感觉也挺好的，不断尝试新事物哇）~~

### 3. Edge（自带的扩展商店下载）

Edge浏览器取代了传统的IE，也使用了 Chromium 内核，带来了很大的改进。其实最重要的是它的易用性，自带的插件商店，也支持 `.crx` 格式的扩展，使用 Window 的用户浪子非常推荐使用这款作为默认的浏览器。

1. [AdGuard 广告拦截器](https://microsoftedge.microsoft.com/addons/detail/adguard-%E5%B9%BF%E5%91%8A%E6%8B%A6%E6%88%AA%E5%99%A8/pdffkfellgipmhklpdmokmckkkfcopbh)
2. [可可翻译](https://microsoftedge.microsoft.com/addons/detail/%E5%8F%AF%E5%8F%AF%E7%BF%BB%E8%AF%91/ebkimaahhkeiplegpghijhgmlcdkeppf)
3. [JSON Formatter for Edge](https://microsoftedge.microsoft.com/addons/detail/json-formatter-for-edge/njpoigijhgbionbfdbaopheedbpdoddi)
4. [Greenhub绿墙—网络出海工具](https://microsoftedge.microsoft.com/addons/detail/greenhub%E7%BB%BF%E5%A2%99%E2%80%94%E7%BD%91%E7%BB%9C%E5%87%BA%E6%B5%B7%E5%B7%A5%E5%85%B7/hholdpohidinjmkoanabdchniingdfac)
5. [文件蜈蚣（需要安装对应的软件）](https://microsoftedge.microsoft.com/addons/detail/jeekmibdibcjeihbjnjgbegjalfelhon)
6. [图片下载](https://microsoftedge.microsoft.com/addons/detail/fatkun%E5%9B%BE%E7%89%87%E6%89%B9%E9%87%8F%E4%B8%8B%E8%BD%BD-pro/dammmokdamnimedflemdaoamhldmldff)
7. [Replace Google CDN](https://microsoftedge.microsoft.com/addons/detail/replace-google-cdn/cojepngjobmaiajphkijbdcdjnnjhpjc)

> 细滚动条设置（默认长方形）：打开 Edge，URL地址输入 “edge://flags”，进入页面搜索 “Windows style overlay scrollbars”，点击选择框选择 Enabled，重启浏览器（仅限 win 平台）。

### 4. Google

谷歌浏览器插件略过，网上推荐有很多，浪子也只是在开发中测试用。这里有个注意点说一下： Google 浏览器使用 Chrono 下载管理器下载 virtualbox 的扩展包时有问题，下载的不是 `.vbox-extpack` 后缀的扩展，使用此扩展下载后是一个 `.gz` 的包，下载时需要暂时关闭该扩展。下面的三个网址可以下载 Google 的插件。

1. [极简插件](https://chrome.zzzmh.cn/)
2. [扩展迷](https://www.extfans.com/)
3. [类似谷歌商店，国内可用](https://www.gugeapps.net/)

> 这三个网址大部分的 crx 插件都可以搜索并下载。
> 后两个需要关注公众号获取下载

### 5. VSCode

1. Markdown All in One
2. Markdown Preview Enhanced
3. Live Server
4. Auto Complete Tag(标签自闭合、修改)
5. Draw.io.Integration(流程图)
6. Chinese(中文环境)
7. Shades of Purple (浪子喜欢的主题)
8. Vloar (Vue插件)
9. PicGo
10. Paste JSON as Code（JSON格式转换为对应语言的“实体类”，emm，Java写多了，姑且这么叫，支持转换多种语言）
11. rust-analyzer(Rust插件)
12. Even Better TOML(Rust插件)
13. JavaScript (ES6) code snippets(js 快捷键)

### 6. IDEA

1. [作者博客地址](https://zhile.io)https://plugins.zhile.io reset(2021.2.2及以下版本很好用；2021.3以下（不含）堪堪能用，需要配合一些手法；2021.3版本开始正式失效，你可以卸载这个插件了！)
2. log support lite 只支持 slf4j
3. Translation
4. GsonFormatPlus(Json -> Java对象生成)
5. Pojo To Json(Java对象 -> Json转换)
6. [Generate All Getter And Setter](https://plugins.jetbrains.com/plugin/18969-generate-all-getter-and-setter)
7. [Free Mybatis Tool](https://plugins.jetbrains.com/plugin/18617-free-mybatis-tool)

> IDEA 2021 之后的版本如果不需要多人协同可以关闭 Code With ME 和 Space 插件，提高运行速度。
> IDEA 2021.3 ~ 最新版本食用方法：https://gitee.com/ja-netfilter/ja-netfilter
