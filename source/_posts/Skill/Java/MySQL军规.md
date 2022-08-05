---
title: MySQL军规
categories: skill
tags: [MySQL]
---

MySQL的优化原则，示项目具体情况而定。

<!-- more -->

## 一、核心

### 1. 尽量不在数据库做运算

让数据库做它擅长的事情：尽量不在数据库做运算，复杂运算移到程序端，尽可能简单应用MySQL。

{% note color:red 反例: md5()/Order by rand() %}

### 2. 控制单表数据量

- 纯 INT 不超过 1000w；
- 含 char 不超过 500w；
- 合理分表不超载：
    - userId
    - DATE
    - AREA
- 建议单库不超过 300~400 个表。

### 3. 保持表身段苗条

- 表字段数少而精
    √ IO高效 √全表遍历 √表修复快 √提高幵发 √alter table快
- 单表1G体积 500W行评估
    - 顺序读1G文件需N秒 单行丌赸过200Byte
    - 单表不超过50个纯INT字段
    - 单表不超过20个CHAR(10)字段
- 表字段数上限控制在 20~50 个。

### 4. 平衡范式与冗余

严格遵循三大范式？
在一些场景中，不必严格遵循三范式，没有绝对的对不错，可以适当时牺牲范式、加入冗余。比如效率优先、提升性能的场景。但会增加代码复杂度。

### 5. 拒绝3B

大SQL（{% emp Big SQL %}）
大事务（{% emp Big Transaction %})
大批量（{% emp Big Batch %}）

## 二、字段

### 1. 用好数值类型

三类数值类型：
 TINYINT(1Byte)
 SMALLINT(2B)
 MEDIUMINT(3B)
 INT(4B)、BIGINT(8B)
 FLOAT(4B)、DOUBLE(8B)
 DECIMAL(M,D)、

### 2. 将字符类型转为数字类型

数字型VS字符串型索引：
  - 更高效
  - 查询更快
  - 占用空间更小


{% note color:green 举例： 用无符号INT存储IP，而非CHAR(15)，使用数据库的函数去查看INT UNSIGNED、INET_ATON()、INET_NTOA() %}

### 3. 优先使用ENUM或SET

优先使用ENUM或SET
- 字符串
- 可能值已知且有限

存储
- ENUM占用1字节，转为数值运算
- SET视节点定，最多占用8字节
- 比较时需要加 {% kbd ' %} 单引号(即使是数值)

{% note color:green 举例： `sex` enum('F','M') COMMENT '性别' \n `c1` enum('0','1','2','3') COMMENT '职介审核' %}
 

### 4. 避免使用NULL字段

- 很难进行查询优化
- NULL列加索引，需要额外空间
- 含NULL复合索引无效

### 5. 少用并拆分TEXT/BLOB

- TEXT类型处理性能远低亍VARCHAR
- 强制生成硬盘临时表
- 浪费更多空间
- VARCHAR(65535) ==> 64K (注意UTF-8)
- 尽量不用TEXT/BLOB数据类型
- 若必须使用则拆分到单独的表

### 6. 不在数据库里存图片

## 三、索引

### 1. 谨慎合理添加索引，能不加的索引尽量不加

- 改善查询
- 减慢更新
- 索引不是赹多赹好
- 综合评估数据密度和数据分布
- 最好不超过字段数20%，结合核心SQL优先考虑覆盖索引

{% note color:green 举例： 不要给“性别”列创建索引 %}

### 2. 字符字段必须建前缀索引

使用场景：前缀的区分度比较高的情况下。

区分度
- 单字母区分度：$26$
- 4字母区分度：$26 * 26 * 26 * 26 = 456,976$
- 5字母区分度：$26 * 26 * 26 * 26 * 26=11,881,376$
- 6字母区分度： $26 * 26 * 26 * 26 * 26 * 26=308,915,776$

建立前缀索引的方式：

```bash
ALTER TABLE table_name ADD KEY(column_name(prefix_length)); 
```

这里面有个 {% kbd prefix_length %} 参数很难确定，这个参数就是前缀长度的意思。通常可以使用以下方法进行确定，先计算全列的区分度，然后在计算前缀长度为多少时和全列的区分度最相似。不断地调整 {% kbd prefix_length %} 的值，直到和全列计算出区分度相近。

```sql
-- 全列区分度
SELECT COUNT(DISTINCT column_name) / COUNT(*) FROM table_name; 
-- 计算前缀长度为多少时和全列的区分度最相似
SELECT COUNT(DISTINCT LEFT(column_name, prefix_length)) / COUNT(*) FROM table_name;
```
### 3. 不在索引列做运算

不在索引列进行数学运算或函数运算，否则会导致无法使用索引以及全表扫描。

### 4. 自增列或全局ID做INNODB主键

- 对主键建立聚簇索引
- 二级索引存储主键值
- 主键不应更新修改
- 按自增顺序插入值
- 忌用字符串做主键
- 聚簇索引分裂
- 推荐用独立于业务的AUTO_INCREMENT列或全局ID生成器做代理主键
- 若不指定主键，InnoDB会用唯一且非空值索引代替

### 5. 尽量不用外键

尽量不使用外键，由程序来保证约束，实际中确实很少使用。

- 外键可节省开发量
- 有额外开销
- 逐行操作
- 可‘到达’其它表，意味着锁
- 高并发时容易死锁

## 四、SQL

### 1. SQL语句尽可能简单

### 2. 保持事务连接小

### 3. 尽可能避免使用SP/TRIG/FUNC

尽可能少用存储过程、触发器
减用使用MySQL函数对结果进行处理，由客户端程序负责。

### 4. 尽量不用SELECT *，只取需要的数据

### 5. 改写OR为IN()

同一字段，将OR改写为IN()
- OR效率：O(n)
- IN 效率：O(Log n)

当n很大时，OR会慢很多。注意控制IN的个数，建议n小于200。

{% note color:green 举例： select * from opp WHERE phone=‘12347856' or 
phone=‘42242233' 
select * from opp WHERE phone in ('12347856' , '42242233') %}

### 6. 改写OR为UNION

不同字段，将or改为union
- 减少对不同字段进行 "or" 查询
- Merge index往往很弱智，如果有足够信心：
{% kbd set globaloptimizer_switch='index_merge=off'; %}

{% note color:green 举例： select * from opp WHERE phone='010-88886666' or 
cellPhone='13800138000'; ==>
Select * from opp WHERE phone='010-88886666' 
union 
Select * from opp WHERE cellPhone='13800138000'; %}

### 7. 避免负向查询和使用%前缀模糊查询

负向查询：NOT、!=、<>、!<、!>、NOT EXISTS、NOT IN、NOT LIKE 等。

避免 % 前缀模糊查询: B+ Tree，不能使用索引，导致全表扫描，效率低。

{% note color:green 举例： select * from post WHERE title like ‘北京%' ;
298 rows in set (0.01 sec) %}

### 8. 少用count(*)

COUNT(*)的资源开销大，尽量少用。

计数统计：
- 实时统计：用memcache，双向更新，凌晨
跑基准；
- 非实时统计：尽量用单独统计表，定期重算。

### 9. LIMIT高效分页

LIMIT 的偏移量越大则查询越慢。

```sql
-- 分页方式一
Select * from table WHERE id>=23423 limit 11;
-- 分页方式二
Select * from table WHERE id >= ( select id from table limit 10000,1 ) limit 10;
-- 分页方式三
SELECT * FROM table INNER JOIN (SELECT id FROM table LIMIT 10000,10) USING (id) ;
```

按场景分析并重组索引。

### 10. 用UNION ALL而非UNION

若无需对结果去重，则用 UNION ALL，UNION 有去重开销。

### 11. 分解连接保证高并发

高并发DB不建议进行两个表以上的JOIN
- 适当分解联接保证高幵发
- 可缓存大量早期数据
- 使用了多个MyISAM表
- 对大表的小ID IN()
- 联接引用同一个表多次

举例：
```sql
MySQL> Select * from tag JOIN tag_post on tag_post.tag_id=tag.id JOIN post on tag_post.post_id=post.id WHERE tag.tag=‘二手玩具’;

-- 拆分
MySQL> Select * from tag WHERE tag=‘二手玩具’;
MySQL> Select * from tag_post WHERE tag_id=1321;
MySQL> Select * from post WHERE post.id in (123,456,314,141)
```

### 12. GROUP BY去除排序

GROUP BY 实现：分组、自劢排序

- 无需排序：Order by NULL
- 特定排序：Group by DESC/ASC

### 13. 同数据类型的列值比较

原则：数字对数字，字符对字符

数值列与字符类型比较
  - 同时转换为双精度进行比对

字符列与数值类型比较
  - 字符列整列转数值
  - 不会使用索引查询

### 14. 不同数据类型的列值比较

字符列与数值类型比较

```sql
-- 字段：
`remark` varchar(50) NOT NULL COMMENT '备注,默认为空',

MySQL>SELECT `id`, `gift_code` FROM gift WHERE `deal_id` = 640 AND remark=115127; 

MySQL>SELECT `id`, `gift_code` FROM pool_gift WHERE `deal_id` = 640 AND remark='115127';
```

### 15. Load Data导数据

批量数据导入：
- 成批装载比单行装载更快，不需要每次刷新缓存
- 无索引时装载比索引装载更快
- Insert values ,values，values 减少索引刷新
- Load data 比 insert 快约20倍
- 尽量不用 INSERT ... SELECT，可能会出现延迟
、同步出错

### 16. 打散大批量更新

凌晨不限制
白天上限默认为100条/秒（特殊再议）

```sql
update post set tag=1 WHERE id in (1,2,3);
sleep 0.01;
update post set tag=1 WHERE id in (4,5,6);
sleep 0.01;
```

### 17. Know Every SQL

```sql
EXPLAIN
SHOW PROFILE
Show Slow Log
Show Processlist
SHOW QUERY_RESPONSE_TIME(Percona)
MySQLdumpslow
MySQLsla
```

## 五、约定

### 1. 隔离线上线下环境

构建数据库的生态环境原则：线上连线上，线下连线下。
- 开发无线上库操作权限
- 实时数据用real库，模拟环境用sim库，测试用qa库，开发用dev库

### 2. 禁止未经DBA确认的子查询

大部分情况子查询的优化较差，特别是 WHERE 中使用 IN 的子查询，一般可用JOIN改写。

### 3. 永远不在程序端显示加锁

永远不在程序端对数据库显式加锁：
- 外部锁对数据库不可控
- 高并发时是灾难
- 极难调试和排查

并发扣款等一致性问题
- 采用事务
- 相对值修改
- Commit前二次较验冲突

### 4. 统一字符集为UTF-8

### 5. 统一命名规范

库表等名称统一用小写，注意避免用保留字命名。

- 索引命名默认为“idx_字段名”
- 库名用缩写，尽量在2~7个字母

> 本篇文章参考 MySQL 军规改写。