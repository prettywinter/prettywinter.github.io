---
title: VsCode代码片段备份
categories:
  - 配置备份
cover: vscode
abbrlink: 7fc7cf56
---

VsCode 代码片段备份。

<!-- more -->

## Java

```json code-snippets
// ###################### Hexo Stellar 主题标签代码片段 ##########################
	"hexo-stellar-label": {
		"scope": "markdown",
		"prefix": "label",
		"body": [
			"{% ${1:label-name} %} $2",
		],
		"description": "generator stellar theme label template"
	},
	"hexo-stellar-image": {
		"scope": "markdown",
		"prefix": "md-img",
		"body": [
			"{% image ${1:src} ${2:desc} width:${3:宽度（可选）} bg:${4:背景（可选）} %}",
		],
		"description": "generator stellar theme image template"
	},
	"hexo-stellar-link": {
		"scope": "markdown",
		"prefix": "md-link",
		"body": [
			"{% link ${1:href} ${2:title} %}",
		],
		"description": "generator stellar theme link template"
	},
	"hexo-stellar-copy": {
		"scope": "markdown",
		"prefix": "md-cp",
		"body": [
			"{% copy ${1:content} %}",
		],
		"description": "generator stellar theme link template"
	},
	"hexo-stellar-mark": {
		"scope": "markdown",
		"prefix": "md-mark",
		"body": [
			"{% mark ${1:content} color:${2:green} %}",
		],
		"description": "generator stellar theme mark template, support red|orange|yellow|green|cyan青|blue蓝|purple|light|dark|warning|error"
	},
	"hexo-stellar-poetry": {
		"scope": "markdown",
		"prefix": "md-poetry",
		"body": [
			"{% poetry ${1:title} author:${2:author} footer:${3:页脚} %}",
			"$4",
			"{% endpoetry %}"
		],
		"description": "generator stellar theme poetry template"
	}
// ########################################################
// ######################### Java #########################
// ########################################################
{
    "log info": {
		"scope": "java",
		"prefix": "logi",
		"body": [
			"log.info(\"$1\", $2)"
		],
		"description": "generator java log.info()"
	},
	"log error": {
		"scope": "java",
		"prefix": "loge",
		"body": [
			"log.error(\"$1\", $2)"
		],
		"description": "generator java log.error()"
	},
	"log debug": {
		"scope": "java",
		"prefix": "logd",
		"body": [
			"log.debug(\"$1\", $2)"
		],
		"description": "generator java log.debug()"
	},
	"java doc": {
		"scope": "java",
		"prefix": "jdoc",
		"body": [
			"/**",
			" *",
			" * @author ${1:user}",
			" * @since 1.0.0",
			" */"
		],
		"description": "generator java class basic info"
	},
}
```

> 生成时间方式：`@date ${CURRENT_YEAR}-${CURRENT_MONTH}-${CURRENT_DATE} ${CURRENT_HOUR}:${CURRENT_MINUTE}:${CURRENT_SECOND}`
