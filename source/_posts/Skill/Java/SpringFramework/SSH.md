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
## spring-boot-starter-data-jpa

### 1. 实体类

实体类需要添加 `@Entity` 声明。如果在实体类中需要忽略某个字段，使用 `@Transient` 注解，此注解表明该字段不参与数据持久化。如果该实体有一对多的关系，使用集合需要添加 `@OneToMany` 注解。

```java
@Entity
public class SysMenu extends BaseEntity {

    /**
     * 菜单名称
     */
    private String menuName;

    /**
     * 父菜单ID
     */
    private Long parentId;

    @OneToMany
    private List<SysMenu> children;

    //...
```

抽取使用公共属性类时，需要在该类上加入 @MappedSuperclass 注解。

```java
@Data
@MappedSuperclass
public abstract class BaseEntity implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    private Long id;
    /**
     * 创建者
     */
    private String createBy;

    /**
     * 创建时间
     */
    private LocalDateTime createTime;

    /**
     * 更新者
     */
    private String updateBy;

    /**
     * 更新时间
     */
    private LocalDateTime updateTime;

    /**
     * 备注
     */
    private String remark;
}
```

### 2. 持久层

持久层类可以实现两个接口：JpaRepository<T, O>, CrudRepository<T, O>，开发中任意选择。

如果点进去看这两个接口源码的话，你会发现 JpaRepository 其实是 CrudRepository 的孙子类：JpaRepository 继承了接口 PagingAndSortingRepository 和 QueryByExampleExecutor。而PagingAndSortingRepository 又继承 CrudRepository。所以，JpaRepository 接口同时拥有了基本CRUD功能以及分页功能。

```java
public interface SysMenuDao extends JpaRepository<User, Long> {

}
```

### 