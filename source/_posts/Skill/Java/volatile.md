---
title: volatile关键字
categories: skill
tags: [Java]
---

本篇文章需要掌握JMM（Java Memory Model）

<!-- more -->

## 介绍

volatile 在多线程环境下可以保证共享数据的可见性，但不保证数据操作的原子性（线程不安全）。要想保证线程安全，还是需要锁机制或者使用原子类（原子类内部已使用 volatile 保证了可见性）。

为了提高 CPU 的运行效率，JVM 会对一些代码进行重新排序。volatile 修饰变量可以实现禁止指令重排序，修正重排序带来的并发安全问题。

![20220727094203](https://raw.githubusercontents.com/prettywinter/dist/main/images/doc/20220727094203.png)

## happens-before规则

我们知道为了提高处理速度，JVM 会进行指令重排序优化，在并发编程下可能会带来一些安全隐患，比如 **重排序导致的多个线程之间的不可见性**。如果让程序员去了解这些底层的实现以及具体的规则，那么我们的负担就太重了，会影响并发编程的效率。
从 JDK5 开始，提出了 happens-before 概念：如果一个操作的执行结果需要对另一个操作可见，那么这两个操作之间必须存在 happens-before 关系。简单来说，就是前一个操作的结果可以被后续的操作获取。

happens-before 规则可以理解为对 JVM 的约束规则。它在编译优化的同时，依然保证了多线程的可见性。

### happens-before的几个规则

1. 程序顺序原则：同一个线程中前面所有的写操作都对后续操作可见（一个线程内保证语义的串行性）。
2. 锁规则：在线程1解锁之前的所有写操作都对后续加锁的线程可见（unlock必然发生在随后的加锁lock前）。
3. volatile 规则：如果线程1写入 volatile 变量v，接着线程2读取了这个值，线程1写入 v 之前的写操作都对线程2可见（volatile 变量的写先发生于读，保证了 volatile 变量的可见性）。
4. 传递性：如果 A happens-before B，B happens-before C，那么 A happens-before C。
5. start() 规则：如果在线程 A 中启动线程 B，那么在线程 B 启动之前，线程 A 中对共享变量的修改都对线程 B 可见。需要注意的是，在线程 B 启动之后，线程 A 再对变量的修改线程 B 未必可见。线程的 start() 方法先于他的每一个动作。
6. join() 规则：对于线程 A 写入的所有变量，如果任一线程调用 A.join() 或者 A.isAlive() 成功返回后，那么 A 写入的变量都对该线程可见。线程的所有操作先于线程的终结。
7. 线程的中断先于被中断线程的代码。
8. 对象的构造函数执行、结束先于 finalize() 方法。

这些原则都是保证指令重排不会破环原有的语义结构。例如第二条锁原则，unlock必然发生在随后的lock前。如果对一个锁解锁后，再加锁，那么加锁的动作绝对不能重排在解锁前。

## volatile使用场景

适合纯赋值操作，不适合类似 a++ 操作。

```java
public class VolatileTest implements Runnable {
    // 使用 public 是为了测试获取到结果值
    public volatile boolean flag = false;
    public AtomicInteger atomicInteger = new AtomicInteger(0);

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            // 修改 flag 的值
            switchFlag();
            // +1
            atomicInteger.incrementAndGet();
        }
    }

    private void switchFlag() {
        // 纯赋值操作符合预期
        flag = true;
        // 不符合预期，可能为 true
        // flag = !flag;
    }
}

class Test03 {
    public static void main(String[] args) throws InterruptedException {
        VolatileTest v = new VolatileTest();
        Thread t1 = new Thread(v);
        Thread t2 = new Thread(v);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(v.flag);
        System.out.println(v.atomicInteger.get());
    }
}
```

volatile 可以适合做多线程中的纯赋值操作，如果一个共享变量自始至终只被各个线程赋值，而没有其它的操作，那么就可以用 volatile 来代替 synchronized 或者代替原子变量，因为赋值自身是有原子性的，而 volatile 又保证了可见性，所以 足以保证线程安全。

## volatile与synchronized

- volatile 只能修饰 **实例变量** 和 **类变量**，而 synchronized 可以修饰方法以及代码块。
- volatile 保证数据的可见性，但是不保证原子性（多线程进行写操作，不保证线程安全）；而 synchronized 是一种排它（互斥）的机制。
- volatile 用于禁止指令重排序，可以解决单例双重检查对象初始化代码执行乱序问题。
- volatile 可以看作是轻量版的 synchronized，volatile 不保证原子性，但是如果对一个共享变量进行直接赋值而没有其它的操作，那么就可以用 volatile 来代替 synchronized，因为赋值本身是有原子性的，而 volatile 又保证了可见性，所以就可以保证线程安全了。

## 总结

1. volatile 修饰符适用于以下场景，某个属性被多个线程共享，其中一个修改了此属性，其它线程可以立即得到修改后的值；或者作为触发器，实现轻量级同步。
2. volatile 属性的读写操作都是无锁的，它不能代替 synchronized，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁，所以它是低成本的。
3. volatile 只能作用于属性，修饰的属性不会被 compilers 做指令重排序。
4. volatile 提供了可见性，任一个线程修改值后将立即对其它线程可见，volatile 属性不会被线程缓存，始终从主存中读取。
5. volatile 提供了 happens-before，保证其修饰的变量在写入 happens-before 后其它线程后续对该变量的读操作。
6. volatile 可以使得 long 和 double 的赋值是原子的。
7. volatile 可以在单例双重检查中实现可见性和禁止指令重排序，从而保证安全性。