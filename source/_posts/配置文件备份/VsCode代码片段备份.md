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