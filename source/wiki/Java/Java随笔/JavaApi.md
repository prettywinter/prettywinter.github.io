---
layout: wiki
wiki: Java
title: Java API
order: 151
---

<!-- more -->

## 一些 Java Api

用法的话有空更新

### 1. 文件监听

WatchService 接口可以用来实现对系统的文件进行监听。

```java
@Test
public void watchFileTest() {
    // 监听路径
    String path = "D:\\";

    try {
        // 创建 WatchService 实例
        WatchService watchService = FileSystems.getDefault().newWatchService();
        // 这里只监听 创建 事件
        // 除此之外，还有修改、删除、特殊事件 
        // StandardWatchEventKinds.ENTRY_MODIFY
        // StandardWatchEventKinds.ENTRY_DELETE
        // StandardWatchEventKinds.OVERFLOW
        Paths.get(path).register(watchService, StandardWatchEventKinds.ENTRY_CREATE);

        while (true) {
            // 一直等待事件，如果没有就阻塞
            // 如果对事件不是那么敏感，可以使用 watchService.poll(long timeout, TimeUnit unit) 每隔多少时间获取一次
            WatchKey key = watchService.take();
            // 获取事件列表
            List<WatchEvent<?>> events = key.pollEvents();
            // 由于只监听了一种事件，这里使用流处理。
            events.stream()
                    .filter(e -> Objects.equals(e.kind(), StandardWatchEventKinds.ENTRY_CREATE))
                    .forEach(e -> {
                        String fileName = e.context().toString();
                        System.out.println("创建了 [" + fileName + "] 文件");
                        try {
                            // 休眠 1ms 是为了让系统完成相应的回调函数，
                            // 否则你会看到输出多次 ”创建了 [" + fileName + "] 文件“ 内容
                            Thread.sleep(100);
                        } catch (InterruptedException ex) {
                            throw new RuntimeException(ex);
                        }
                    });
            // 重置 watchKey 继续监听文件夹的事件
            key.reset();
        }
    } catch (IOException | InterruptedException e) {
        throw new RuntimeException(e);
    }
}
```

### 2. 获取某个接口的所有实现类

ServiceLoader 的使用非常的简单。

```java
@Test
public void testServiceLoader() {
    java.util.ServiceLoader<AssetsStrategy> strategies = java.util.ServiceLoader.load(AssetsStrategy.class);
    strategies.forEach(System.out::println);
}
```

当然，如果直接执行以上代码是没有任何打印内容的。因为它还需要一点儿额外的配置。你需要在 `resources/META-INF` 目录下新建 `services` 目录，然后在新建的目录中添加一个文件，这个文件名必须是 **接口所在** 的 `包路径+接口名称`，不带任何后缀，区分大小写。最后按照同样的格式把实现了该接口的实现类写到该文件中。

举个栗子，上面我要获取的是 `AssetsStrategy` 接口的实现类，假如该接口在 `com.example.strategy` 包中，并且它有 `Model1`、`Model2` 两个实现类，那么我应该在 `resources/META-INF/services/com.example.strategy.AssetsStrategy` 文件中添加以下内容：

```txt com.example.strategy.AssetsStrategy
com.example.model.Model1
com.example.model.Model2
```

然后运行上面的测试代码，就可以看到结果啦~

## Java 异步

https://www.callicoder.com/java-8-completablefuture-tutorial/
https://www.callicoder.com/java-concurrency-multithreading-basics/

https://www.codedemo.club/spring-security-async-principal-propagation/