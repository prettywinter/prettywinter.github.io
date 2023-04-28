---
layout: wiki
wiki: Java
title: Java事务失效
order: 152
---

Java 事务失效的几种情况。

<!-- more -->

## 一、Spring 声明式事务失效的几种情况

Spring 的声明式事务非常的好用，但是使用不当的话事务会失效，对于生产环境中是非常严重的问题。

如果对 AOP 熟悉的 coder 可能这个问题非常简单，如果不了解也可以凭经验解决。下面总结一些情况。

权限修饰符和 final 的情况不用说了，如果使用 IDE 的话基本上都会提示错误。

### 1. 自行 try-catch，吞掉异常

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

如果是这种情况，虽然添加了事务，那么也不会生效。解决的办法就是在 **catch** 中抛出异常，Spring 事务回滚默认是捕获 RuntimeException 和 Error，当然 Spring 也提供了 `rollbackFor` 属性捕获其它异常，开发中最好指定异常。如下代码：

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

### 2. 同一服务类的方法相互调用

不论是事务方法调用事务方法，还是非事务方法调用事务方法，一般使用 **声明式事务** 失效的情况以此类居多。

```java
@Override
public void importExcel(List<User> list) {
    // do something parser
    saveBatch(list);
}

@Override
@Transactional(propagation = Propagation.NESTED)
public void saveBatch(List<User> list) {
    // do something
}
```

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

### 3. 嵌套调用

事务嵌套调用时，有时候我们希望这种场景：里面的事务方法如果发生异常回滚，但我不希望外面的回滚：

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void importExcel(List<User> list) {
    // do something parser
    updateSome(user);
    saveBatch(list);
}

@Override
@Transactional(propagation = Propagation.NESTED)
public void saveBatch(List<User> list) {
    // do something
}
```

上面的例子中，当 `saveBatch` 发生异常时，我希望它回滚，但是我不希望之前的 `updateSome` 回滚。但是事实确实都回滚了。原因就是如果下面的 `saveBatch` 发生异常时会向上抛，以至于外层也发生了异常，所以解决办法也很简单，直接 try-catch。

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void importExcel(List<User> list) {
    // do something parser
    updateSome(user);
    try {
      saveBatch(list);
    } catch (Exception e) {
      // TODO: handle exception
    }
}

@Override
@Transactional(propagation = Propagation.NESTED)
public void saveBatch(List<User> list) {
    // do something
}
```

## 二、事务优化

使用事务时最好指定回滚异常，设置事务传播属性；
尽量避免大事务连接，可以考虑使用上面的解决办法把事务方法抽离出来，进行内部调用。

> 本文参考：https://blog.csdn.net/hanjiaqian/article/details/120501741。
