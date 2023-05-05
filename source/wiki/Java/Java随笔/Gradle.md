---
layout: wiki
wiki: Java
title: Gradle
order: 159
---

## 使用

Gradle 的语法简介，使用起来也挺方便，我很喜欢用，接下来讲讲它的配置使用。本文以 gradle 8.1 版本为例，对于 7.x 应该同样生效叭叭叭吧。

### 1. settings.gradle


```groovy settings.gradle
// 项目的根文件名称
rootProject.name = '项目文件夹名称'
// 包含的子模块名称
include 'module1', 'module2', 'module3', 'module4'
```

### 2. build.gradle

```groovy build.gradle

// 插件
plugins {
    // Java 插件 它为 Java 项目提供标准的 Java 工具链（如编译、测试、打包、生成JavaDoc等）和默认的目录布局
    id 'java'
    // SpringBoot 插件 帮助配置和构建 Spring Boot 应用程序
    id 'org.springframework.boot' version '3.0.6'
    // Spring Dependency Management 插件可以帮助管理项目中的依赖关系，
    // 它可以让您在不需要手动指定版本号的情况下，使用特定版本的 Spring 库，并确保这些库与您项目中其他库的版本相兼容。
    // 使用该插件，不需要在 build.gradle 中声明 dependencyManagement，它会辅助自动生成
    id 'io.spring.dependency-management' version '1.1.0'
}

group 'com.leaf'
// 项目版本
version '1.0-SNAPSHOT'
// java version
sourceCompatibility = '17'

// 版本号管理
ext {
    postgresqlVersion = '42.6.0'
}

// 配置所有项目公共内容
allprojects {
    // 该插件启用了 Java 编译任务和其他与 Java 编译相关的任务，例如运行 JUnit 测试和生成 Javadoc 文档。
    apply plugin: 'java'
    // 该插件启用了与 IntelliJ IDEA 集成相关的任务，并为 Gradle 启用了与 IntelliJ IDEA 的互操作性，例如生成 IDEA 工程文件。
    apply plugin: 'idea'
    // 该插件向应用中添加 Spring Boot 的支持，并自动配置应用以使用 Spring Boot 约定。
    apply plugin: 'org.springframework.boot'
    // 该插件提供了一种便捷的方式来管理应用中的依赖项，允许您声明应用所需的依赖项，而不必手动指定每个依赖项的版本信息。
    apply plugin: 'io.spring.dependency-management'

    // 创建一个 Java 编译任务，然后将 options.encoding 属性设置为 UTF-8，以便编译器能够正确地处理文件中的 Unicode 字符。
    tasks.withType(JavaCompile) {
        options.encoding = 'UTF-8'
    }

    // 依赖下载源设置
    repositories {
        // 本地 maven 仓库
        mavenLocal()
        // 阿里云仓库
        maven { url 'https://maven.aliyun.com/repository/public/' }
        maven { url 'https://maven.aliyun.com/repository/spring/' }
        // 如果项目中需要从 http 下载依赖，打开下面的设置
        // allowInsecureProtocol = true
        // maven { url 'http://maven.nexus.com/xxxx'}
        // maven 仓库
        mavenCentral()
    }

    // 全局依赖，对所有项目模块生效，子模块如果单独配置相同的依赖仍然不能覆盖
    dependencies {
//        testImplementation 'org.springframework.boot:spring-boot-starter-test'
//        /* spring security */
//        implementation('org.springframework.boot:spring-boot-starter-security')
//        /* spring web */
//        implementation('org.springframework.boot:spring-boot-starter-web')
//        /* spring aop */
//        implementation('org.springframework.boot:spring-boot-starter-aop')
//        /* spring data jpa */
//        implementation('org.springframework.boot:spring-boot-starter-data-jpa')
//        /* validation */
//        implementation('org.springframework.boot:spring-boot-starter-validation')
//        /* postgresql */
//        implementation('org.postgresql:postgresql:${postgresqlVersion}')
//        /* java jwt */
//        // https://mvnrepository.com/artifact/com.auth0/java-jwt
//        implementation 'com.auth0:java-jwt:4.2.1'
//        /* 缩略图 */
//        implementation 'net.coobird:thumbnailator:0.4.19'

    }
}

// 为子模块单独配置，两种配置方式，二选一
subprojects {
    // 方式一
    dependencies {
        if (name == "module1") {
            implementation 'com.example:library1:1.0'
        } else if (name == "module2") {
            implementation 'com.example:library2:2.0'
        } else {
            implementation 'com.example:global-library:1.0'
        }
    }

    // 方式二
    project(":module1") {
        dependencies {
            testImplementation 'org.springframework.boot:spring-boot-starter-test'
            implementation('org.springframework.boot:spring-boot-starter-web')
        }
    }
    project(":module2") {
        dependencies {
            implementation 'com.example:global-library:1.0'
        }
    }
}
```

上面的是多模块的写法，子模块中的 `build.gradle` 可以写也可以不写，就空着也行。对于单体应用使用 gradle 更加简单，就不写了，和上面是类似的，需要使用的 gradle api 更加少了（~~当然，看项目大小~~）。
