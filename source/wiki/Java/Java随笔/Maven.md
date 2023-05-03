---
layout: wiki
wiki: Java
title: Maven
order: 157
---

## Maven打包

打包之前推荐先清理（clean）再打包（package）。jar 包比较简单，SpringBoot 默认打的就是 jar 包。这里说一下 war 包的打包方式方式。

1. 修改打包方式，去除内嵌的 tomcat 依赖

    ```xml pom.xml
    <packaging>war</packaging>

    ...

    <!-- 去除内嵌的 tomcat 依赖 -->
    <scope>provided</scope>
    ```

2. 如果整合的是 jsp，那么使用打包的插件时，打包插件必须是 `1.4.2` 的版本，另外，在pom文件中加入以下内容，指定 jsp 文件的打包位置：

    ```xml pom.xml
        <resources>
            <!-- 打包时将 jsp 文件拷贝到 META-INF 目录下 -->
            <resource>
                <!-- 指定 resources 插件处理那个目录下的资源文件 -->
                <directory>src/main/webapp</directory>
                <!-- 指定必须要放在此目录下才能被访问到 -->
                <targetPath>META-INF/resources</targetPath>
                <includes>
                    <include>**/**</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/**</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
            <directory>src/main/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </resource>
        </resources>
    ```

3. 在插件中指定入口类：

    ```xml
    <configuration>
        <fork>true</fork>
        <jvmArgument>-Dfile.encoding=UTF-8</jvmArgument>
        <mainClass>com.XXX.Application.class</mainClass>
    </configuration>
    ```

4. 在启动类中，继承 `SpringBootServletInitiallizer` 类，重写 `configure()` 方法进行配置:

    ```java
    public class xxxApplication extends SpringBootServletInitializer {
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources(xxxApplication.class);
        }
    }
    ```

> **注意：** 打成的 war 包使用外部 Tomcat 部署时，项目配置文件中配置的端口和项目名称会失效。
> 后台方式启动 jar 包：`nohup java -jar jar包名称 &`