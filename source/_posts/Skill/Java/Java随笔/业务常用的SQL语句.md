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
-- 构建近12个月的月份虚拟表
SELECT DATE_FORMAT(@cdate := date_add( @cdate, INTERVAL - 1 MONTH ),'%Y-%m') as date
FROM (SELECT @cdate := date_add(now(), INTERVAL 1 MONTH ) FROM table_name LIMIT 12) d ORDER BY date
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
