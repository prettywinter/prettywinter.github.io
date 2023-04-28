---
layout: wiki
wiki: Database
title: 业务常用SQL
order: 1
---

一些常用的业务 SQL，另外数据库推荐使用日期格式：`%Y-%m-%d %H:%i:%s`。

<!-- more -->

## MySQL

```sql
-- 重置表 ID 从 1 开始自增; id_key 为自增 id
alter table table_name drop id_key;
alter table table_name add id_key int not null primary key auto_increment first;

-- 增加递增列
SELECT @rank:=@rank + 1 AS num FROM (SELECT @rank:=0) a
-- 构建近 12 个月的月份虚拟表
-- 注意：table_name 中的数据记录条数必须 >= 12 
SELECT DATE_FORMAT(@cdate := date_add( @cdate, INTERVAL - 1 MONTH ),'%Y-%m') AS cdate
FROM (SELECT @cdate := date_add(NOW(), INTERVAL 1 MONTH ) FROM table_name LIMIT 12) a ORDER BY cdate
-- 构建近 30 天的天数虚拟表
-- 注意：table_name 中的数据记录条数必须 >= 30
SELECT DATE_FORMAT(@cdate := date_add( @cdate, INTERVAL - 1 DAY ),'%Y-%m-%d') AS cdate
FROM (SELECT @cdate := date_add(now(), INTERVAL 1 DAY ) FROM table_name LIMIT 30) a ORDER BY cdate

-- 构建近 30 天的日期虚拟表，此种写法 indexs 不能去掉，必须查询，适合有这种需求的使用
-- 同样，table_name 中的数据记录条数必须 >= 30
SELECT @cdate := @cdate + 1 AS indexs,
DATE_FORMAT( DATE( DATE_SUB( CURRENT_DATE, INTERVAL @s DAY ) ), '%Y-%m-%d' ) AS cdate 
FROM table_name, ( SELECT @cdate := -1 ) temp
WHERE @cdate < 30
ORDER BY cdate

-- 查询今天的数据
SELECT * FROM table_name WHERE TO_DAYS(create_time) = TO_DAYS(NOW());
-- 查询当前一周的数据 1 表示一周的第一天从星期一开始
SELECT * FROM table_name WHERE YEARWEEK(DATE_FORMAT(create_time,'%Y-%m-%d'), 1) = YEARWEEK(NOW(), 1);
-- 查询本月的数据
SELECT * FROM table_name WHERE DATE_FORMAT(create_time, '%Y-%m') = DATE_FORMAT(CURDATE(), '%Y-%m');

-- 查询最近七天的数据(当前日期往前推七天)
SELECT a.cdate AS mday
FROM (
    SELECT CURDATE() AS cdate
    UNION ALL
    SELECT DATE_SUB(CURDATE(), INTERVAL 1 DAY) AS cdate
    UNION ALL
    SELECT DATE_SUB(CURDATE(), INTERVAL 2 DAY) AS cdate
    UNION ALL
    SELECT DATE_SUB(CURDATE(), INTERVAL 3 DAY) AS cdate
    UNION ALL
    SELECT DATE_SUB(CURDATE(), INTERVAL 4 DAY) AS cdate
    UNION ALL
    SELECT DATE_SUB(CURDATE(), INTERVAL 5 DAY) AS cdate
    UNION ALL
    SELECT DATE_SUB(CURDATE(), INTERVAL 6 DAY) AS cdate
) a LEFT JOIN (
  SELECT DATE(create_time) AS c_time, other
  FROM table_name
  GROUP BY DATE(create_time)
) b ON a.cdate = b.c_time ORDER BY mday;
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

## PG

```sql
-- 生成指定的一段时间的连续日期，学了这个我感觉 MySQL5.7 是个什么垃圾！！！
select to_char(cdate,'YYYY-MM-DD') as date from
generate_series(to_timestamp('2023-03-27','YYYY-MM-DD'), to_timestamp('2023-04-01','YYYY-MM-DD'),'1 days') as cdate
order by date
```

> PG 的 generate_series 函数可以学习一下，非常 nice：https://www.cnblogs.com/mchina/archive/2013/04/03/2997722.html
