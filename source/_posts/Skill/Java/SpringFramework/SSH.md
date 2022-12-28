---
title: SSH
categories:
  - Skill
  - Java
tags:
  - java
  - ssh
abbrlink: 782a8ece
---

Java 框架之 SSH。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

## SSH

在 Hibernate3 中，引入了 Restrictions 类作为 Expression 的替代，以后的版本，不再推荐使用 Expression。

```java
// 推荐使用
Criteria cr = session.createCriteria(ChargeRecord.class).add(Restrictions.eq("chargeStatus", "01")).setProjection(Projections.sum("chargeAmount"));
// 不再推荐
TbItemParamExample example = new TbItemParamExample();
Criteria criteria = example.createCriteria();
criteria.andItemCatIdEqualTo(cid);
Criteria ct= session.createCriteria(TUser.class);
//Criteria中可以增加查询条件
ct.add(Expression.eq("name","Erica"));
ct.add(Expression.eq("sex",new Integer(1)));
//Criteria中增加的查询条件可以由表达式对象创建
```
