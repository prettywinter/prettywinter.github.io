---
title: Spring Data Jpa
categories:
  - Skill
  - Java
tags:
  - java
  - jpa
abbrlink: '2219457'
---

Spring Data Jpa 整理。

<!-- more -->

## Spring Data Jpa

```gradle build.gradle
// 引入依赖
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

### 1. 实体类

实体类需要添加 `@Entity` 声明，该注解有一个 name 属性，标识对应的数据库的表名，当然你也可以使用另一个注解 @Table 实现同样的效果。如果在实体类中需要忽略某个字段，使用 `@Transient` 注解，此注解表明该字段不参与数据持久化。如果该实体有一对多的关系，使用集合需要添加 `@OneToMany` 注解，同理，还有一个 `@ManyToMany` 注解代表多对多的关系。

```java
@Entity(name = "表名")
@Table("表名")
@DynamicInsert // 动态插入
@DynamicUpdate // 动态更新 只更新需要更新的列（改变）
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

    /**
     * 主键 id，必须有
     * GeneratedValue 代表使用数据库的自增模式，最好不要使用默认的 auto 策略
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
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

如果点进去看这两个接口源码的话，你会发现 JpaRepository 其实是 CrudRepository 的孙子类：JpaRepository 继承了接口 PagingAndSortingRepository 和 QueryByExampleExecutor。而PagingAndSortingRepository 又继承 CrudRepository。所以，JpaRepository 接口同时拥有了基本 CRUD 功能以及分页功能。

```java
public interface SysMenuDao extends JpaRepository<User, Long> {

}
```

### 分页

PageRequest、Pageable、Page、PageImpl<>

PageRequest 是 Pageable 的子类，Pageable 是一个分页接口。我们可以使用 PageRequest 构造分页对象，查询时传入此参数实现分页查询。

PageImpl 是 Page 的实现类，PageImpl 用于最终返回的分页列表信息。他的构造方法接收三个参数：结果集，Pageable，总记录数。

```java
// 构造分页对象
PageRequest pageRequest = PageRequest.of(pageIndex, pageSize);
Page<User> all = userRepository.findAll(pageRequest);
log.info("{}", all);
log.info("总数：{}", all.getTotalElements());
return new PageImpl<>(userRepository.findAll(), pageRequest, all.getTotalElements());
```
