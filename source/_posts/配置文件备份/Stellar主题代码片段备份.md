---
title: Stellar主题代码片段备份
categories:
  - 配置备份
cover: vscode
abbrlink: 92bbe64e
---

Hexo 博客使用的 Stellar 主题，该主题简洁大方，深得我意。该主题提供了许多标签，写出来的效果很不错，写文章时经常用到，为此，写了一些常用的代码片段，备份一下。

<!-- more -->

```json
{
	// Place your global snippets here. Each snippet is defined under a snippet name and has a scope, prefix, body and 
	// description. Add comma separated ids of the languages where the snippet is applicable in the scope field. If scope 
	// is left empty or omitted, the snippet gets applied to all languages. The prefix is what is 
	// used to trigger the snippet and the body will be expanded and inserted. Possible variables are: 
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. 
	// Placeholders with the same ids are connected.
	// Example:
	// "Print to console": {
	// 	"scope": "javascript,typescript",
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }

	// ###################### Hexo 主题标签代码片段 ##########################
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
}
```