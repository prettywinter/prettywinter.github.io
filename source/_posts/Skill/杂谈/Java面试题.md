---
title: Java面试题总结
categories: 杂谈
tags: [面试题]
abbrlink: 3dcb9ab5
---

把他人分享的一些面试题整理一下，这份面试题适合有工作经验的人去看，并且从实际中去获取答案，直接去找参考答案的方式浪子并不推荐。对于应付面试的可以，多了解一些东西都是有好处的。

<!-- more -->

## 一、技术框架，自主思考

1、介绍一下你做的项目和其中难点
2、你觉得Spring的出现解决了什么问题？
3、谈谈你对AOP的理解？
4、Spring中的Bean是在时候加载到IOC容器里？
5、对比一下Spring和Springboot的区别？都已经有Spring这种框架了为什么还要去开发Springboot和Spring cloud？
6、了解过阿里开发的Seata框架吗？它的出现解决了什么问题？
7、docker，k8s 这种技术的出现，解决了什么问题？
8、你对日志框架了解多少？项目中用过那个日志框架，分别用到过那些级别？
9、redis的淘汰策略的底层实现，能手写一个LRU算法看看吗？
10、了解那些中间件？RabbitMQ、RocketMQ、Dubbo、ZK，他们之前的区别？
11、Dubbo是如何实现RPC？如果让你自己实现RPC，如何去做，说说思路即可
12、讲一下Redis在你项目中的应用以及相应的算法
13、简单的讲讲Tomcat中的bio/nio，为什么7之后默认nio呢？
14、反射的出现解决了什么问题？如果不使用反射会怎么样？
15、计算机为什么要设计高速缓存架构？它解决了什么问题？
16、多线程，并发编程了解多少？
17、了解那些锁？以种类来说，例如：公平锁与非公平锁
18、让你去学习一个框架，你多久能上手？
19、让你去带一个小团队，你怎么去管理这个团队？
20、你想做业务还是技术？

## 二、Java原生知识

1、说说你对JUC的理解 它的出现解决了什么问题？
2、了解JUC的核心吗？AQS是什么 它的出现解决了什么问题？
3、你对ReentranLock的理解 它解决了什么问题？
4、Synchronized为什么在1.6之后的版本做了优化 解决了什么问题？
5、了解哪些锁？以种类来说，例如：公平锁与非公平锁
6、Synchronized的锁升级过程？它是怎么存在对象的内存布局里的？
7、Synchronized和ReentrantLock那个性能好？有哪些区别？
8、说说你对volatile的理解 它是怎么保证可见性？
9、了解CMS吗？知道G1吗？两者的区别何在？分别解决了什么问题？
10、了解对象的结构内存分布吗？对象头分别储存了那些信息？在锁升级的过程中，对象头分别做了那些变化？
11、简述一下类的生命周期
12、Thread、ThreadLocal、ThreadLocalMap之间的关系；ThreadLocal的出现解决了什么问题？什么情况下会出现内存泄漏？该如何避免这种情况发生？
13、了解HashMap的结构吗？简单的说一下HashMap的扩容机制，JDK1.7之前为什么会出现环化？JDK1.8是怎么优化的？HashMap是1、有序的吗？我要怎么把乱序的KV键值对以一定的规律输出？
14、除了HashMap，还了解那些Map?他们和HashMap的区别何在？
15、JVM内存模型，类加载机制，回收算法
16、简单的描述一下JVM类加载机制？为什么要有双亲委派机制？它解决了什么问题？
17、你对JVM了解多少？有做过调优的案例吗？说说你是怎么调优的？

## MySQL

1、MySQL的事务了解多少？MVCC解决了什么问题？redo_log、undo_log分别解决了什么问题？
2、MySQL中的MVCC底层实现机制，有了解过吗？
3、MySQL的索引知道那些？都做过那些调优？

## 算法

1、给定两个整数，让他们相除，返回他们的商，不能使用除法、乘法和mod运算符
