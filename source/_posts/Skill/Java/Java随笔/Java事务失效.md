

https://blog.csdn.net/hanjiaqian/article/details/120501741

## 一、Spring 声明式事务失效的几种情况

Spring 的声明式事务非常的好用，但是使用不当的话事务会失效，对于生产环境中是非常严重的问题。

如果对 AOP 熟悉的 coder 可能这个问题非常简单，如果不了解也可以凭经验解决。下面总结一些情况。

权限修饰符和 final 的情况不用说了，如果使用 IDE 的话基本上都会提示错误。

### 1. 同一个Service、同一个方法内

#### ① 自行 try-catch

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void saveBatch(List<User> list) {
  try{
    saveBatch(list);
  } catch (Exception e) {
    log.error("保存失败：{}", e.getMessage());
  }
}
```

如果是这种情况，虽然添加了事务，那么也不会生效。解决的办法就是在 **catch** 中抛出异常。如下代码：

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void saveBatch(List<User> list) {
  try{
    saveBatch(list);
  } catch (Exception e) {
    log.error("保存失败：{}", e.getMessage());
    throw new RuntimeException("保存失败");
  }
}
```
#### ② 非事务方法调用事务方法

### 2. 同一个Service、不同的方法

一般使用 **声明式事务** 遇到的最多失效的原因就是这种情况了。

以下面的方法为例：

```java
// 事务方法或者非事务方法
@Override
public void importExcel(List<User> list) {
    // do something parser
    saveBatch(list);
}
// 事务方法
@Override
@Transactional(rollbackFor = Exception.class)
public void saveBatch(List<User> list) {
    userMapper.saveBatch(list);
}
```

#### ① 事务方法调用事务方法

虽然不一定失效，但是概率比较低（如果涉及多个表操作，会发生有的表回滚，有的表产生脏数据），测试时需要严格测试。

#### ② 非事务方法调用事务方法

这个肯定是会失效的。


解决办法有下面几种：
1. 使用 Spring 提供的 ` AopContext.currentProxy()` （简单方便）；
2. 将其中的一个方法移至到另一个 `xxxService` 中，调用时使用 `@Autowired` 注入调用这是没问题的（还记得开头说的 AOP？）。因为在同一个 `Service` 中调用方法就不是 Spring 代理的对象了，所以事务也就不会生效了（`@Transactional` 事务是 Spring 管理的）；
3. 不使用 Spring 的事务管理，自己编写工具管理事务。

这里说说第一种，浪子也是第一次使用，其它两种方法基本上使用过 Spring Web 的 coder 都会。

1. 暴露 Spring 代理对象，需要开启
2. 业务调用

```java
@SpringBootApplication
// 暴露出 Spring 代理对象给业务使用，默认为 false
@EnableAspectJAutoProxy(exposeProxy = true)
public class Application {...}

// 业务调用
@Override
public void importExcel(List<User> list) {
    // do something parser
    // 拿到代理对象 调用相关方法
    xxxService currentProxy = (xxxService) AopContext.currentProxy();
    currentProxy.saveBatch(list);
}

@Override
@Transactional(rollbackFor = Exception.class)
public void saveBatch(List<User> list) {
    userMapper.saveBatch(list);
}
```
