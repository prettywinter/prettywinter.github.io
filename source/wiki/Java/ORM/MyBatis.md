---
layout: wiki
wiki: Java
title: MyBatis、MyBatis Plus（MP）
order: 101
---

## 一、MyBatis

## 二、MyBatis Plus

### 1. 使用 Wrappers.xxxquery() 查询某个字段

有时候，我们只需要查询一张表中的一个或几个字段（同一张表），又不想写 SQL 的情况下，可以使用 `queryWapper.select()` 方式，举个栗子：

```java
public List<Integer> listManageLevel() {
    LambdaQueryWrapper<User> query = Wrappers.lambdaQuery();
    // 使用 select 方式只查询 age 字段
    query.select(User::getAge).groupBy(User::getAge);
    // 如果使用 selectList() 返回的是对象集合
    List<User> users = userMapper.selectList(query);
    // 也可以使用下面的方式，返回一个 Object 集合，注意修改返回值
    // List<Object> ages = sasacMapper.selectObjs(query);

    List<Integer> ages = users.stream()
                            .map(User::getAge)
                            .collect(Collectors.toList());
    return ages;
}
```

> 注意，上面的查询返回的是一个对象集合，还需要使用 map 映射取出字段。当然，你也可以使用注释掉的代码，返回一个 Object 集合，对于单字段来说是比较好的选择。
