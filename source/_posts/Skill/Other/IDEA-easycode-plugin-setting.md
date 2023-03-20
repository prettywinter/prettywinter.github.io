---
title: IDEA 插件 EasyCode 配置
categories:
  - IDEA
tags:
  - idea plugin
  - easycode
abbrlink: 50f4489
---

<!-- more -->

## [EasyCode](https://plugins.jetbrains.com/plugin/10954-easycode)

EasyCode 是 IDEA 中的一款自动代码生成插件，非常的好用，本文放置浪子的配置文件 `spring-data-jpa` 作为备份。`json` 格式文件，可以导入配置。

```json EasyCodeConfig.json
{
  "author" : "jhlz",
  "version" : "1.2.7",
  "userSecure" : "",
  "currTypeMapperGroupName" : "Default",
  "currTemplateGroupName" : "spring-data-jpa",
  "currColumnConfigGroupName" : "Default",
  "currGlobalConfigGroupName" : "Default",
  "typeMapper" : {
    "Default" : {
      "name" : "Default",
      "elementList" : [ {
        "matchType" : "REGEX",
        "columnType" : "varchar(\\(\\d+\\))?",
        "javaType" : "java.lang.String"
      }, {
        "matchType" : "REGEX",
        "columnType" : "char(\\(\\d+\\))?",
        "javaType" : "java.lang.String"
      }, {
        "matchType" : "REGEX",
        "columnType" : "(tiny|medium|long)*text",
        "javaType" : "java.lang.String"
      }, {
        "matchType" : "REGEX",
        "columnType" : "decimal(\\(\\d+,\\d+\\))?",
        "javaType" : "java.lang.Double"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "integer",
        "javaType" : "java.lang.Integer"
      }, {
        "matchType" : "REGEX",
        "columnType" : "(tiny|small|medium)*int(\\(\\d+\\))?",
        "javaType" : "java.lang.Integer"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "int4",
        "javaType" : "java.lang.Integer"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "int8",
        "javaType" : "java.lang.Long"
      }, {
        "matchType" : "REGEX",
        "columnType" : "bigint(\\(\\d+\\))?",
        "javaType" : "java.lang.Long"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "date",
        "javaType" : "java.time.LocalDate"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "datetime",
        "javaType" : "java.time.LocalDateTime"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "timestamp",
        "javaType" : "java.time.LocalDateTime"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "time",
        "javaType" : "java.time.LocalTime"
      }, {
        "matchType" : "ORDINARY",
        "columnType" : "boolean",
        "javaType" : "java.lang.Boolean"
      } ]
    }
  },
  "template" : {
    "spring-data-jpa" : {
      "name" : "spring-data-jpa",
      "elementList" : [ {
        "name" : "controller.java.vm",
        "code" : "##导入宏定义、设置包名、类名、文件名##导入宏定义\n$!{define.vm}\n#setPackageSuffix(\"controller\")\n#setTableSuffix(\"Controller\")\n#save(\"/controller\", \"Controller.java\")\n\n##拿到主键\n#if(!$tableInfo.pkColumn.isEmpty())\n    #set($pk = $tableInfo.pkColumn.get(0))\n#end\n##定义服务名\n#set($serviceSortType = $!tool.append($!tableInfo.name, \"Service\"))\n#set($serviceType = $!tool.append($!tableInfo.savePackageName, \".service.\", $serviceSortType))\n#set($serviceVarName = $!tool.firstLowerCase($serviceSortType))\n\n#set($entityShortType = $!tableInfo.name)\n## set($entityType = $!tableInfo.psiClassObj.getQualifiedName())\n#set($entityType = $!tool.append($!tableInfo.savePackageName, \".entity.\", $!tableInfo.name))\n#set($entityVarName = $!tool.firstLowerCase($!tableInfo.name))\n#set($pkType = $!pk.type)\n\nimport $pkType;\nimport $entityType;\nimport $serviceType;\nimport org.springframework.data.domain.Page;\nimport org.springframework.data.domain.Pageable;\nimport org.springframework.data.web.PageableDefault;\nimport org.springframework.web.bind.annotation.*;\n\nimport java.util.List;\n\n/**\n * $!{tableInfo.comment}控制层\n *\n * @author $!author\n * @since $!time.currTime()\n */\n@RestController\n@RequestMapping(\"/$!tool.firstLowerCase($!tableInfo.name)\")\npublic class $!{tableName} {\n\n\t/**\n\t * 获取$!{tableInfo.comment}列表(分页)\n\t */\n\t@GetMapping\n\tpublic Page<$entityShortType> listPage(@RequestBody $entityShortType $entityVarName, \n\t                               @PageableDefault(page = 0, size = 20) Pageable page) {\n\t\treturn null;\n\t}\n\n\t/**\n\t * 获取$!{tableInfo.comment}\n\t */\n\t@GetMapping(\"/{id}\")\n\tpublic $entityShortType queryById(@PathVariable $!pk.shortType id) {\n\t\treturn ${serviceVarName}.findById(id);\n\t}\n\n\t/**\n\t * 添加$!{tableInfo.comment}\n\t */\n\t@PostMapping\n\tpublic void add(@RequestBody $entityShortType $entityVarName) {\n\t\t${serviceVarName}.save($entityVarName);\n\t}\n\n\n\t/**\n\t * 修改$!{tableInfo.comment}\n\t */\n\t@PutMapping\n\tpublic void edit(@RequestBody $entityShortType $entityVarName) {\n\t\t${serviceVarName}.update($entityVarName);\n\t}\n\n\t/**\n\t * 删除$!{tableInfo.comment}\n\t */\n\t@DeleteMapping(\"/{ids}\")\n\tpublic void delete(@PathVariable List<$!pk.shortType> ids) {\n\t\t${serviceVarName}.remove(ids);\n\t}\n\n\tprivate final $serviceSortType $serviceVarName;\n\n    public $!{tableName} ($serviceSortType $serviceVarName) {\n        this.$serviceVarName = $serviceVarName;\n    }\n}\n"
      }, {
        "name" : "entity.java.vm",
        "code" : "##引入宏定义\n$!{define.vm}\n\n##使用宏定义设置回调（保存位置与文件后缀）\n#save(\"/entity\", \".java\")\n\n##使用宏定义设置包后缀\n#setPackageSuffix(\"entity\")\n\nimport jakarta.persistence.Entity;\nimport jakarta.persistence.GeneratedValue;\nimport jakarta.persistence.GenerationType;\nimport jakarta.persistence.Id;\nimport org.hibernate.annotations.DynamicInsert;\nimport org.hibernate.annotations.DynamicUpdate;\n\n##使用全局变量实现默认包导入\n$!{autoImport.vm}\nimport java.io.Serial;\nimport java.io.Serializable;\n\n#set($entityShortType = $!tableInfo.name)\n#set($entityType = $!tool.append($!tableInfo.savePackageName, \".entity.\", $!tableInfo.name))\n#set($entityVarName = $!tool.firstLowerCase($!tableInfo.name))\n#set($pkType = $!pk.type)\n\n##使用宏定义实现类注释信息\n#tableComment(\"实体类\")\n@Entity(name = \"$!tool.hump2Underline($!{tableInfo.name})\")\n@DynamicInsert\n@DynamicUpdate\npublic class $!{tableInfo.name} implements Serializable {\n    @Serial\n    private static final long serialVersionUID = $!tool.serial();\n#foreach($column in $tableInfo.fullColumn)\n    #if(${column.comment})/**\n     * ${column.comment}\n     */#end\n     \n#if(${column.name} == \"id\")\n    @Id\n    @GeneratedValue(strategy = GenerationType.IDENTITY)\n#end\n    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};\n#end\n\n#foreach($column in $tableInfo.fullColumn)\n##使用宏定义实现get,set方法\n#getSetMethod($column)\n#end\n\n    @Override\n    public String toString() {\n        return \"$!{tableInfo.name}{\" +\n#foreach($column in $tableInfo.fullColumn)\n    #if(!$foreach.hasNext) \n        $tool.append('\"', \"$column.name=\", '\" + ', $column.name, ' + ')\n                $tool.append('\"', '}', '\";')\n    #else \n        $tool.append('\"', \"$column.name=\", '\" + ', $column.name, ' + \"' , ',', '\"', ' +')\n    #end\n#end\n    }\n\n    @Override\n    public boolean equals(Object o) {\n        if (this == o) return true;\n        if (o == null || getClass() != o.getClass()) return false;\n        $entityShortType $entityVarName = ($entityShortType) o;\n        return\n#foreach($column in $tableInfo.fullColumn)\n    #if(!$foreach.hasNext) \n        Objects.equals($column.name, $entityVarName.$column.name);\n    #else\n        Objects.equals($column.name, $entityVarName.$column.name) &&\n    #end\n#end   \n    }\n\n    @Override\n    public int hashCode() {\n        return Objects.hash(#foreach($column in $tableInfo.fullColumn)\n#if(!$foreach.hasNext)$tool.append($column.name)#else$tool.append($column.name,\",\") #end\n#end\n);\n    }\n\n}\n"
      }, {
        "name" : "repository.java.vm",
        "code" : "##导入宏定义、设置包名、类名、文件名##导入宏定义\n$!{define.vm}\n#setPackageSuffix(\"repository\")\n#setTableSuffix(\"Repository\")\n#save(\"/repository\", \"Repository.java\")\n\n##拿到主键\n#if(!$tableInfo.pkColumn.isEmpty())\n    #set($pk = $tableInfo.pkColumn.get(0))\n#end\n##实体类名、主键类名\n#set($entityShortType = $!tableInfo.name)\n#set($entityType = $!tool.append($!tableInfo.savePackageName, \".entity.\", $!tableInfo.name))\n#set($pkShortType = $!pk.shortType)\n#set($pkType = $!pk.type)\n\nimport $pkType;\nimport $entityType;\nimport org.springframework.data.jpa.repository.JpaRepository;\nimport org.springframework.stereotype.Repository;\n\n/**\n * $!{tableInfo.comment}持久层\n *\n * @author $!author\n * @since $!time.currTime()\n */\n@Repository\npublic interface $!{tableName} extends JpaRepository<$entityShortType, $pkShortType> {\n\n}\n"
      }, {
        "name" : "service.java.vm",
        "code" : "##导入宏定义、设置包名、类名、文件名##导入宏定义\n$!{define.vm}\n#setPackageSuffix(\"service\")\n#setTableSuffix(\"Service\")\n#save(\"/service\", \"Service.java\")\n\n##拿到主键\n#if(!$tableInfo.pkColumn.isEmpty())\n    #set($pk = $tableInfo.pkColumn.get(0))\n#end\n##实体类名、主键类名\n#set($entityShortType = $!tableInfo.name)\n#set($entityType = $!tool.append($!tableInfo.savePackageName, \".entity.\", $!tableInfo.name))\n#set($entityVarName = $!tool.firstLowerCase($!tableInfo.name))\n#set($pkShortType = $!pk.shortType)\n#set($pkType = $!pk.type)\n\nimport $pkType;\nimport $entityType;\nimport org.springframework.data.domain.Page;\nimport org.springframework.data.domain.Pageable;\nimport java.util.Collection;\nimport java.util.List;\n\n\n/**\n * $!{tableInfo.comment}业务层\n *\n * @author $!author\n * @since $!time.currTime()\n */\npublic interface $!{tableName} {\n\n    /**\n     * 分页列表\n     *\n     * @param user 条件参数\n     * @param page 分页参数\n     * @return 符合条件的分页数据\n     */\n    Page<$entityShortType> listPage($entityShortType $entityVarName, Pageable page);\n    \n    /**\n     * 根据 ID 获取详情\n     *\n     * @param id id\n     * @return 详情对象\n     */\n    $entityShortType findById($pkShortType id);\n    \n    /**\n     * 添加数据\n     *\n     * @param $entityVarName 添加数据内容\n     */\n    void save($entityShortType $entityVarName);\n    \n    /**\n     * 更新数据\n     *\n     * @param $entityVarName 更新的数据内容\n     */\n    void update($entityShortType $entityVarName);\n\n    /**\n     * 删除数据\n     *\n     * @param ids 删除数据的 id 集合\n     */\n    void remove(List<$pkShortType> ids);\n}\n"
      }, {
        "name" : "serviceImpl.java.vm",
        "code" : "##导入宏定义、设置包名、类名、文件名\n$!{define.vm}\n#setPackageSuffix(\"service.impl\")\n#setTableSuffix(\"ServiceImpl\")\n#save(\"/service/impl\", \"ServiceImpl.java\")\n\n##拿到主键\n#if(!$tableInfo.pkColumn.isEmpty())\n    #set($pk = $tableInfo.pkColumn.get(0))\n#end\n##业务层类名、持久层类名、实体名\n#set($serviceSortType = $!tool.append($!tableInfo.name, \"Service\"))\n#set($serviceType = $!tool.append($!tableInfo.savePackageName, \".service.\", $serviceSortType))\n#set($repositorySortType = $!tool.append($!tableInfo.name, \"Repository\"))\n#set($repositoryType = $!tool.append($!tableInfo.savePackageName, \".repository.\", $repositorySortType))\n#set($repositoryVarName = $!tool.firstLowerCase($!repositorySortType))\n#set($entityShortType = $!tableInfo.name)\n#set($entityType = $!tool.append($!tableInfo.savePackageName, \".entity.\", $!tableInfo.name))\n#set($entityVarName = $!tool.firstLowerCase($!tableInfo.name))\n#set($pkShortType = $!pk.shortType)\n#set($pkType = $!pk.type)\n\nimport $pkType;\nimport $entityType;\nimport $serviceType;\nimport $repositoryType;\nimport org.springframework.stereotype.Service;\nimport javax.annotation.Resource;\nimport org.springframework.data.domain.Page;\nimport org.springframework.data.domain.Pageable;\nimport org.springframework.transaction.annotation.Transactional;\n\nimport java.util.Collection;\nimport java.util.List;\nimport java.util.stream.Collectors;\nimport java.util.stream.StreamSupport;\n\n\n/**\n * $!{tableInfo.comment}业务层\n *\n * @author $!author\n * @since $!time.currTime()\n */\n@Service\npublic class $!{tableName} implements $!serviceSortType {\n    @Override\n    public Page<$entityShortType> listPage($entityShortType $entityVarName, Pageable page) {\n        return $!{repositoryVarName}.findAll(page);\n    }\n    \n    @Override\n    public $entityShortType findById($pkShortType id) {\n        return $!{repositoryVarName}.findById(id).orElseThrow();\n    }\n\n    @Override\n    @Transactional(rollbackFor = Exception.class)\n    public $entityShortType save($entityShortType $entityVarName) {\n        $!{repositoryVarName}.save($entityVarName);\n    }\n    \n    @Override\n    @Transactional(rollbackFor = Exception.class)\n    public void update($entityShortType $entityVarName) {\n        $!{repositoryVarName}.save($entityVarName);\n    }\n\n    @Override\n    @Transactional(rollbackFor = Exception.class)\n    public void remove(List<$pkShortType> ids) {\n        $!{repositoryVarName}.deleteAllById(ids);\n    }\n\n    private final $repositorySortType $repositoryVarName;\n\n    public $!{tableName} ($repositorySortType $repositoryVarName) {\n        this.$repositoryVarName = $repositoryVarName;\n    }\n}\n"
      }, {
        "name" : "record.java.vm",
        "code" : "##引入宏定义\n$!{define.vm}\n\n##使用宏定义设置回调（保存位置与文件后缀）\n#save(\"/entity\", \".java\")\n\n##使用宏定义设置包后缀\n#setPackageSuffix(\"entity\")\n\nimport jakarta.persistence.Entity;\nimport jakarta.persistence.GeneratedValue;\nimport jakarta.persistence.GenerationType;\nimport jakarta.persistence.Id;\n\n##使用全局变量实现默认包导入\n$!{autoImport.vm}\nimport java.io.Serial;\nimport java.io.Serializable;\n\n#set($entityShortType = $!tableInfo.name)\n#set($entityType = $!tool.append($!tableInfo.savePackageName, \".entity.\", $!tableInfo.name))\n#set($entityVarName = $!tool.firstLowerCase($!tableInfo.name))\n#set($pkType = $!pk.type)\n\n##使用宏定义实现类注释信息\n#tableComment(\"实体类\")\n@Entity(name = \"$!tool.hump2Underline($!{tableInfo.name})\")\npublic record $!{tableInfo.name}(\n#foreach($column in $tableInfo.fullColumn)\n    #if(${column.comment})/**\n     * ${column.comment}\n     */#end\n     \n#if(${column.name} == \"id\")\n    @Id\n    @GeneratedValue(strategy = GenerationType.IDENTITY)\n#end\n#if(!$foreach.hasNext) \n    $!{tool.getClsNameByFullName($column.type)} $!{column.name}\n#else\n    $!{tool.getClsNameByFullName($column.type)} $!{column.name},\n#end\n    \n#end\n) implements Serializable {\n    \n}\n"
      } ]
    }
  },
  "columnConfig" : { },
  "globalConfig" : { }
}
```
