---
title: Spring源码分析笔记
categories: skill
tags: [Spring, 源码]
abbrlink: fd4a5dd8
---

Spring源码分析笔记

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true} -->

工厂的分类：
BeanFactory接口：

1. ConfigurableBeanFactory
2. AutowireCapableBeanFactory
3. ListableBeanFactory
    DefaultListableBeanFactory

XmlBeanFactory（Spring3.1之后过期）：读取XML配置文件，创建对应的对象

```java
BeanFactory beanFactory = new XmlBeanFactory(Resource->xml)
beanFactory.getBean();
```

1. 怎么读取配置文件获得IO资源
2. 读取配置文件后，如何在Spring中以对象的形式进行封装。
3. 根据配置信息创建对象
4. 所创建对象的生命周期

Spring MVC 开发中同时存在 2 个 Spring 工厂。

Spring 定时任务和事务管理配置
<task:annotation >
<tx:annotation-manager >

属性编辑器(PropertyEditor) 老版的自定义转换器，由 jdk 提供。
Spring 提供了 Convertor 接口供我们实现自定义转换器。

各种Aware不在工厂启动的时候操作，而是在创建对象的第三步初始化中通过 BeanPostProcessor 完成。不要在工厂启动的时候注入，而是在对象创建的时候注入。

默认情况下 ConfigurationClassPostProcessor 处理顶级注解（注册）：
@Component（@Service @Repository @Controller）
@Configuration
@PropertySource
@Import
@ComponentScan

@Configuration 注解的类中，@Bean 注解的配置bean通过代理增加额外的scope功能。

代理设计模式  额外功能的增强  可以通过动态字节码技术创建 包括JDK CGLIB ASM Javasist（MyBatis也支持）
装饰器设计模式  本职功能的增强

SpringBoot中修改创建代理的方式，添加@EnableApsectJAutoProxy，覆盖SpringBoot的内置设置。

Spring动态代理应用默认 JDK 代理
SpringBoot默认的CGLIB代理。

Spring AOP创建的代理是在 创建对象-> 属性填充 -> 初始化的时候执行的
思考：代理对象一定会在初始化的时候创建？
不一定，如果涉及循环引用，创建->singletonFactries->lambda ->创建

BeanPostProcessor创建代理时考虑循环引用的问题
ProxyFactory .setTarget .setAdvisor
最底层的代码：
AopProxy
CglibAopProxy JdkDnmicalAopProxy

实现了BeanPostProcessor接口不建议使用@Autowired注入。如果需要获取Spring工厂创建的某个对象，可以使用BeanFactoryAware

循环引用的结论：
两个对象都是单实例的情况下，且通过set方式进行注入才能成功。

循环依赖的解决：
三个缓存池（Map），用来存放创建的对象：
singletonFactories
singletonObjects
earlySingleObjects

getBean("a");
getSingleton();
步骤一：
1.singletonObjects    null
2.earlySingleObjects  null
3.singletonFactories  null
步骤二：
1.singletonObjects    null
2.earlySingleObjects  null
3.singletonFactories  != null lambda getEarlyBeanRederence(beanName, mbd, bean) 创建代理 proxy，然后从 singletonFactories 中移除，proxy 放到 earlySingletonObjects 中。

> 先到 singletonObjects 中获取，如果null，在 earlySingleObjects 中获取，如果 null，在 singletonFactories 中获取。
> lambda getEarlyBeanRederence(beanName, mbd, bean) 创建代理，然后从 singletonFactories 中移除，proxy 放到 earlySingletonObjects 中。
creationBean a(半成品)
属性的填充：涉及 getBean("b")，过程和上面类似，在 b 中需要 a，上面过程中已经创建了 a，所以可以顺利拿到，进而创建 b。循环依赖就是“你中有我，我中有你” 的解决方法。
