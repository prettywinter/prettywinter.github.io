---
title: vsocde配置备份
categories:
  - 配置备份
cover: vscode
abbrlink: 9e8f8686
---

自用的 VsCode（1.72~lasted） 配置备份，Plugin 部分需安装对应的插件才可生效。

<!-- more -->

```json setting.json
{   
    // #################################################################
    // ########################### 编辑器配置 ############################
    // #################################################################

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
    "editor.tabSize": 2,
    // 保存时自动格式化
    "editor.formatOnSave": true,
    // 彩色括号对 1.6x 版本之后 IDE 自带，不需要安装插件了
    "editor.bracketPairColorization.enabled": true,
    "editor.guides.bracketPairs": "active",
    "editor.suggestSelection": "first",
    // 启用或禁用在 VS Code 中重命名或移动文件时自动更新导入路径的功能。
    "javascript.updateImportsOnFileMove.enabled": "always",
    // 开启 markdown 的验证 无效链接将被警告或者错误
    "markdown.validate.enabled": true,

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

    // ########################### 开启 md 文件的代码片段 ##########################
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

    // #################################################################
    // ########################### 编辑器配置 end ########################
    // #################################################################


    // ##########################################################################
    // ##########################################################################
    // ############################# Plugin start ###############################
    // ##########################################################################
    // ##########################################################################
    
    // vscode 主题设置(Shades of Purple插件)
    "workbench.colorTheme": "Shades of Purple",

    // markdown-preview-enhanced 配置(Markdown Preview Enhanced插件)
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

    // Java 相关配置(java 相关插件)
    "java.configuration.runtimes": [
        {
            "name": "JavaSE-17",
            "path": "/usr/local/jdk-17",
          },
    ],
    // maven settings 路径(java 相关插件)
    "java.configuration.maven.globalSettings": "/usr/local/maven-3.8.6/conf/settings.xml",
    "java.dependency.packagePresentation": "hierarchical",


    // ##########################################################################
    // ##########################################################################
    // ############################# 插件配置 end ################################
    // ##########################################################################
    // ##########################################################################
}
```
