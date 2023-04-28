---
layout: wiki
wiki: Java
title: Java单元测试
order: 153
---

对于开发来说，单元测试是非常重要的。本篇文章就来记录 Java 中一个较好的单元测试开发实践。

<!-- more -->

一个好的单元测试尽量实现脱离数据库，减少其它不需要启动的组件和依赖。一般而言我们都会在测试的时候连接数据库，这样的话启动时会启动整个项目（测试类使用 @SpringBootTest）。如果这样的话对于项目的启动和测试时所花费的时间那是比较长的，尤其是微服务，那测试启动差不多何以喝一杯 coffee 了。

## Mock数据的优点

1. 可以很简单的虚拟出一个复杂对象（比如虚拟出一个接口的实现类）；
2. 可以配置 mock 对象的行为；
3. 可以使测试用例只注重测试流程与结果；
4. 减少外部类、系统和依赖给单元测试带来的耦合。

## Mockito

采用 Mock 框架，我们可以对已存在的接口测试，只注重代码的 **流程与结果**，从而实现测试目的。

Mockito 是 SpringBoot 自带的一个单元测试工具，可以对数据进行模拟。**注意：** 本文的代码示例均为为 SpringBoot 2.5.x+ 版本。

```java MockitoTest.java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;
import org.mockito.quality.Strictness;
import org.springframework.util.Assert;

import java.util.Objects;

public class MockitoTest {

    /**
     * 注意：如果使用 @MockBean 必须与 RunWith 一同使用，否则此 mock 对象无法获取
     */
    @Mock
    UserService userService;

    AutoCloseable autoCloseable;

    /**
     * 每次执行测试类之前都执行此方法
     * @BeforeAll 在运行测试类时只执行一次
     */
    @BeforeEach
    public void openMock() {
        // 开启 mock 功能（Spring Boot2.5 的写法）
        autoCloseable = MockitoAnnotations.openMocks(this);
        // 设置 Mockito 推荐的严格检查配置
        Mockito.mockitoSession().startMocking().setStrictness(Strictness.STRICT_STUBS);
    }

    /**
     * 每次执行测试类之后都执行此方法
     * @AfterAll 在运行完测试类后只执行一次
     */
    @AfterEach
    public void closeMock() {
        try {
            autoCloseable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * test: Mockito example
     */
    @Test
    public void testMockitoWithReturn() {
        User user = new User();
        user.setId("10");
        // 当你调用 when() 括号里的表达式的时候，会返回后面的 thenReturn() 中的值
        // 除此之外，还可以模拟抛出异常，指定返回的对象值
        Mockito.when(userService.insert(user)).thenReturn(10);
        
        // 调用获取 mock 中设置的 thenReturn 值
        int byId = userService.insert(user);
        // 进行断言测试
        Assert.isTrue(Objects.equals(10, byId), "id must is 10");
    }

    /**
     * 返回异常
     */
    @Test
    public void testMockitoWithException() {
        User user = new User();
        user.setId("10");
        // 当你调用 when() 括号里的表达式的时候，会返回后面的 thenThrow() 抛出的异常
        Mockito.when(userService.insert(user)).thenThrow(new ArrayIndexOutOfBoundsException());
        
        // 调用获取 mock 中设置的 thenThrow 异常（所以就不断言了）
        int byId = userService.insert(user);

        // 调用这个就会正常执行
        int newUser = userService.insert(new User());
        Assert.notNull(newUser, "newUser must is not null");
    }

    /**
     * 返回自定义数据
     */
    @Test
    public void testMockitoWithCustomer() {
        ExPass exPass = new ExPass();
        exPass.setId("10");
        // thenAnswer() 与 then() 是一样的
        Mockito.when(exPassMapper.insert(exPass)).thenAnswer((invocation) -> {
            return new IllegalArgumentException();
        });

        int byId = exPassMapper.insert(exPass);
        Assert.isTrue(Objects.equals(1, byId), "save not is false");
    }
}
```

> 在 SpringBoot2.5.x 中的 mockito 的版本是 3.9.0。除了上面测试的方法外还有一个 `thenCallRealMethod()` 方法，它的使用也非常简单，有兴趣的可以自己尝试一下~
