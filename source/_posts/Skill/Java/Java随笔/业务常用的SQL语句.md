---
title: 一些业务常用SQL
categories:
  - Skill
tags:
  - sql
abbrlink: 44d4a5e6
---

一些常用的业务 SQL。

<!-- more -->

## MySQL

```sql
-- 增加递增列
select @rank:=@rank + 1 AS num from (select @rank:=0) a
-- 构建近 12 个月的月份虚拟表
-- 注意：table_name 中的数据记录条数必须 >= 12 
SELECT DATE_FORMAT(@cdate := date_add( @cdate, INTERVAL - 1 MONTH ),'%Y-%m') as date
FROM (SELECT @cdate := date_add(now(), INTERVAL 1 MONTH ) FROM table_name LIMIT 12) d ORDER BY date
-- 构建近 30 天的天数虚拟表
-- 注意：table_name 中的数据记录条数必须 >= 30 
SELECT DATE_FORMAT(@cdate := date_add( @cdate, INTERVAL - 1 DAY ),'%Y-%m-%d') as date
FROM (SELECT @cdate := date_add(now(), INTERVAL 1 DAY ) FROM event e  LIMIT 30) d ORDER BY date

-- 构建近 30 天的日期虚拟表，此种写法 indexs 不能去掉，必须查询，适合有这种需求的使用
-- 同样，table_name 中的数据记录条数必须 >= 30
SELECT @s := @s + 1 AS indexs,
DATE_FORMAT( DATE( DATE_SUB( CURRENT_DATE, INTERVAL @s DAY ) ), '%Y-%m-%d' ) AS dates 
FROM table_name, ( SELECT @s := -1 ) temp
WHERE @s < 30 
ORDER BY dates
```

## Oracle

```sql
-- 补全一年 12 个月
select lpad(level,2,0) from dual connect by level<13

select to_char(sysdate, 'yyyy-') || lpad(level, 2, 0) datevalue from dual connect by level < 13;

--oracle 未来12个月
SELECT TO_CHAR(ADD_MONTHS(ADD_MONTHS(SYSDATE, 0), ROWNUM - 1), 'YYYY-MM') AS YEARMONTH
FROM ALL_OBJECTS
WHERE ROWNUM <= 12;

--oracle 前12个月
SELECT TO_CHAR(ADD_MONTHS(SYSDATE, 1 - ROWNUM), 'YYYY-MM') AS YEARMONTH
FROM ALL_OBJECTS
WHERE ROWNUM <= 12;
```

[hive、oracle、mysql内建函数对照表](https://help.aliyun.com/document_detail/96342.html)
