---
title: Vsocde、Fleet配置备份
categories:
  - 配置备份
cover: vscode
abbrlink: 9e8f8686
---

自用的 VsCode（1.72~lasted） 配置备份，Plugin 部分需安装对应的插件才可生效。

<!-- more -->

## VsCode

```json settings.json
{
    // ########################### 编辑器配置 ############################

    // 字体及字体大小，设置的字体本机需要安装才能生效
    // 如果有空格或者中文，尽量使用 单引号 括住
    "editor.fontFamily": "'Fira Code', '文泉驿微米黑'",
    "editor.fontSize": 16,
    // 启用连字
    "editor.fontLigatures": true,
    // 设置行高
    "editor.lineHeight": 23,
    // 字母间像素
    "editor.letterSpacing": 0.5,
    // tab 缩进大小
    "editor.tabSize": 4,
    // 保存时自动格式化内容
    "editor.formatOnSave": true,
    // 彩色括号对
    "editor.bracketPairColorization.enabled": true,
    "editor.guides.bracketPairs": "active",
    "editor.suggestSelection": "first",

    // ########################### 文件配置 ###########################

    // 1s 后自动保存文件内容 
    "files.autoSave": "afterDelay",
    // 要想 autoSaveDelay 生效，上面的设置必须为 afterDelay，否则不生效
    "files.autoSaveDelay": 1000,

    // ########################### 终端设置 ###########################
    // 光标样式
    "terminal.integrated.cursorStyle": "line",
    "terminal.integrated.cursorWidth": 2,
    "terminal.integrated.cursorBlinking": true,
    // 鼠标右键：控制是否将在终端中选定的文本复制到剪贴板。
    "terminal.integrated.copyOnSelection": true,
    // 鼠标右键：控制终端如何回应右键单击操作。copyPaste: 当有选定内容时复制，否则粘贴。
    "terminal.integrated.rightClickBehavior": "copyPaste",
    // 设置默认打开的终端类型，Linux ? zsh : PowerShell
    // "terminal.integrated.defaultProfile.linux": "zsh",
    // "terminal.integrated.defaultProfile.windows": "PowerShell",
    // 浪子喜欢在 Win 下使用 Git Bash
    "terminal.integrated.profiles.windows": {
        // 命名可自定义，在 terminal.integrated.defaultProfile.windows 处填写自定义名即可
        "Git-Bash": { 
          "path": "D:\\xxxx\\Git\\bin\\bash.exe",
          "args": [],
          "icon": "terminal-bash"
        }
      },
    // 此处填写上面的自定义名称 Git-Bash
    "terminal.integrated.defaultProfile.windows": "Git-Bash",

    // ########################### JavaScript ###########################

    // 启用或禁用在 VS Code 中重命名或移动文件时自动更新导入路径的功能。always：一直自动更新
    "javascript.updateImportsOnFileMove.enabled": "always",

    // ########################### md ###########################

    // 开启 markdown 的验证 无效链接将被警告或者错误
    "markdown.validate.enabled": true,
    // 开启 markdown 的用户代码片段，在 md 中也可以使用代码片段啦
    "[markdown]": {
      "editor.renderWhitespace": "all",
      "editor.quickSuggestions": {
        "comments": "on",
        "strings": "on",
        "other": "on"
      },
      "editor.acceptSuggestionOnEnter": "on"
    },

    // ########################### 编辑器配置 end ########################


    // ##########################################################################
    // ##########################################################################
    // ############################# Plugin start ###############################
    // ##########################################################################
    // ##########################################################################
    
    // vscode 主题设置 (Shades of Purple插件)
    "workbench.colorTheme": "Shades of Purple",

    // markdown-preview-enhanced 配置 (Markdown Preview Enhanced插件)
    "markdown-preview-enhanced.codeBlockTheme": "default.css",
    "markdown-preview-enhanced.previewTheme": "vue.css",
    "markdown-preview-enhanced.printBackground": true,

    // Draw.io Integration 配置：添加自定义字体选项、设置默认主题（Draw.io Integration插件）
    "hediet.vscode-drawio.customFonts": [
        "JetBrains Mono",
        "Fira Code",
        "文泉驿微米黑"
    ],
    "hediet.vscode-drawio.theme": "Kennedy",

    // 安装 PicGo 软件，使用 PicGo 插件只需要配置一个 configPath
    // 本机安装了 PicGo 客户端第一次启动后会自动生成 json 文件
    // "picgo.configPath": "C:\\Users\\你的用户名\\AppData\\Roaming\\picgo\\data.json",
    // 上面的选项如果配置了，下面的无须配置。

    // 只使用 VsCode PicGo 插件，需要配置以下信息
    // 当前使用图床 可选：weibo, qiniu, tcyun, upyun, github, aliyun, imgur and SM.MS
    "picgo.picBed.current": "github|weibo|qiniu|tcyun|upyun|aliyun|imgur|SM.MS",
    "picgo.picBed.uploader": "github|weibo|qiniu|tcyun|upyun|aliyun|imgur|SM.MS",
    // 设定分支
    "picgo.picBed.github.branch": "main",
    // 设定仓库
    "picgo.picBed.github.repo": "",
    // token
    "picgo.picBed.github.token": "",
    // 自定义返回的链接
    "picgo.picBed.github.customUrl": "",
    // 上传路径
    "picgo.picBed.github.path": "",

    // Java 配置
    "java.configuration.runtimes": [
        {
          // 名称，随便起，多个的话必须唯一
          "name": "JavaSE-17",
          // jdk 路径，配置环境变量的路径
          "path": "/usr/lib/jvm/java-17-openjdk",
          // 源码路径
          "sources": "/usr/lib/jvm/java-17-openjdk/lib/src.zip",
          // VsCode 默认使用这个版本的 jdk
          "default": true,
        },
        {
          "name": "JavaSE-1.8",
          "path": "/usr/lib/jvm/java-8-openjdk",
          "sources": "/usr/lib/jvm/java-8-openjdk/src.zip"
        },  
    ],
    // java 文件自动组织导入的包
    "java.saveActions.organizeImports": true,
    // vscode 包结构展示效果 flat：平面；hierarchical：分层
    "java.dependency.packagePresentation": "hierarchical",
    // 不使用 gradle-wrapper.properties 的 gradle
    "java.import.gradle.wrapper.enabled": false,
    // 设置默认的 gradle 版本
    "java.import.gradle.version": "7.6",
    // 本地 gradle 路径
    "java.import.gradle.home": "/home/clf/software/gradle-7.6",
    "java.import.gradle.java.home": "/usr/lib/jvm/java-17-openjdk",
    // GRADLE_USER_HOME 路径
    "java.import.gradle.user.home": "/home/clf/software/gradle-repo/",

    // ########################### Maven 配置 ###########################
    // settings.xml 路径
    "java.configuration.maven.globalSettings": "/home/clf/software/maven-3.9.0/conf/settings.xml",
    // maven 执行文件路径
    "maven.executable.path": "/home/clf/software/maven-3.9.0/bin/",
    // maven 命令默认选项
    "maven.executable.options": "-o -DskipTests",

    // ##########################################################################
    // ##########################################################################
    // ############################# 插件配置 end ################################
    // ##########################################################################
    // ##########################################################################
}
```
## Fleet

配置参见官网：https://www.jetbrains.com/help/fleet/settings.html#user-settings

```json settings.json
{
    // 主题
    "theme": "dark_purple",
    // 字体大小
    "editor.fontSize": 16.0,
    // 保存时格式化文本
    "editor.formatOnSave": true,
    // 使用组合键键时显示所用组合键
    "showShortcuts": true,
    // 快捷键映射，这里采用了 vscode 的
    "keymap": "pc-vscode",
    
    // ###### 终端配置 ######
    "terminal.profiles": [
        {
            "program": "D:\\software\\Git\\bin\\bash.exe",
            "args": [], // optional
            "name": "git-bash" // optional
        }
    ],
    // 终端：复制选中文本
    "terminal.copyOnSelection": true,
    // 终端字体大小
    "terminal.fontSize": 15.0,

    // ############# Java ##############
    "maven.user.settings": [
        {
            "path": "D:\\xxx\\Java\\maven-3.9.0\\conf\\settings.xml"
        }
    ],
    "java.runtimes": [
        {
            "path": "D:\\xxx\\Java\\jdk1.8.0_361"
        }
    ],
}
```