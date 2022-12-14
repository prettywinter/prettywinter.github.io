---
title: Spring Security总结
categories:
  - Skill
  - Java
tags:
  - java
  - spring
  - spring security
abbrlink: 95588d77
---

[Spring Security](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html) 是一个功能强大且高度可定制的身份验证和访问控制框架，专注于为 Java 应用程序提供身份验证和授权。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true} -->

<!-- code_chunk_output -->

- [一、官网学习](#一官网学习)
  - [1. 思考](#1-思考)
- [二、登录认证](#二登录认证)
  - [1. 一些说明](#1-一些说明)
      - [AuthenticationManager ProviderManager AuthencationProvider关系](#authenticationmanager-providermanager-authencationprovider关系)
      - [WebSecurityConfigurerAdapter（最新版本标为弃用）](#websecurityconfigureradapter最新版本标为弃用)
      - [UserDetails](#userdetails)
      - [UserDetailsService用来修改默认认证的数据源信息](#userdetailsservice用来修改默认认证的数据源信息)
      - [Authentication](#authentication)
  - [2. 配置AuthenticationManager的两种方式](#2-配置authenticationmanager的两种方式)
  - [3. 密码加密](#3-密码加密)
  - [4. RememberMe](#4-rememberme)
  - [5. 会话管理(SessionManagementFilter)](#5-会话管理sessionmanagementfilter)
  - [6. 跨域问题](#6-跨域问题)
  - [7. 异常处理](#7-异常处理)
    - [7.1 认证异常](#71-认证异常)
    - [7.2 授权异常](#72-授权异常)
    - [7.3 自定义异常处理配置](#73-自定义异常处理配置)
  - [8. CSRF（Cross-site request forgery）](#8-csrfcross-site-request-forgery)
- [二、授权](#二授权)
  - [2.1 两种权限管理策略](#21-两种权限管理策略)
    - [1. 基于URL权限管理](#1-基于url权限管理)
    - [2. 基于方法的权限管理](#2-基于方法的权限管理)
  - [2.2 动态权限修改](#22-动态权限修改)
  - [2.3 RBAC(Role-Based Access Control)](#23-rbacrole-based-access-control)

<!-- /code_chunk_output -->

## 一、官网学习

{% link https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html 官方文档 %}

Spring Security是基于一系列的 Filter 来完成的。核心分为认证、授权。

新版本的 Security 配置不需要继承 {% kbd WebSecurityConfigurerAdapter %} 类，其配置写法主要是以注册 Bean 的方式为主，具体的使用可以阅读 [官方文档](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter#ldap-authentication)

### 1. 思考

- 为什么在引入 Spring Security 之后没有任何配置所有的请求就要认证呢？
  SpringBootWebSecurityConfiguration，自动配置类
- 在项目中明明没有登录界面，登录界面怎么来的呢？
  访问接口时，在引入 spring security 之后会先经过一系列过滤器。在请求到达 FilterSecurityInterceptor 时，发现请求并未认证，请求拦截下来，并抛出 AccessDeniedException 异常。抛出的异常会被 ExceptionTranslationFilter 捕获，这个 Filter 中会调用 LoginUrlAuthenticationEntryPoint#commence 方法给客户端返回 302，要求客户端进行重定向到 /login 页面。客户端发送 /login 请求。/login 请求会再次被拦截器中 DefaultLoginPageGeneratingFilter 拦截到，并在拦截器中返回生成登录页面。
- 为什么使用 user 和控制台打印的密码能登录，登录时验证数据源存在哪里呢？
  Spring Security中有一个基于内存（InMemoryUserDetailsManager）的默认用户实现。

## 二、登录认证

登录

1. 调用 ProviderManager 的方法进行认证，如果认证成功生成 jwt，把用户信息存入 redis。
2. 自定义 UserDetailsService，查询数据库

校验

定义 jwt 认证过滤器，获取 token，解析 token 中的 userid，查询 redis 中是否存在相应的 userid 并获取用户信息，最后存入 SecurityContextHolder。

### 1. 一些说明

##### AuthenticationManager ProviderManager AuthencationProvider关系

AuthenticationManager 有全局的和局部的，无论哪种，都是通过 ProviderManager 进行实现。每一个 ProviderManager 中都代理一个 AuthenticationProvider列表，列表中每一个实现代表一种身份认证方式。认证时底层数据源调用 UserDetailsService 来实现。

##### WebSecurityConfigurerAdapter（最新版本标为弃用）

WebSecurityConfigurerAdapter 是 Spring Security 为我们提供的扩展类，方便我们重写默认配置，实现定制。

Spring Security(5.7) 中的WebSecurityConfigurerAdapter 标记为弃用状态。新版的改变与使用参考 [5.7新版配置](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter#ldap-authentication)，不同之处在于改变了写法，新版主要以配置 Bean 的方式使用。

##### UserDetails

提供核心用户信息，UserDetailsService#loadByUsername() 返回的就是一个 UserDetails 对象。

##### UserDetailsService用来修改默认认证的数据源信息

UserDetailsService 接口下有许多的实现。同时，此接口也方便了我们以后的自定义数据源的扩展。里面定义了一个根据用户名查询用户信息的方法。

##### Authentication

封装的用户信息，包括用户名和密码。

### 2. 配置AuthenticationManager的两种方式

1. 继承 WebSecurityConfigurerAdapter，springboot 对 security 默认配置中 在工厂默认创建 AuthenticationManager。

    ```java
    // 默认配置会自动发现创建的 UserDetailService 的 Bean
    @Autowired
    public void initialize(AuthenticationManagerBuilder builder, DataSource datasource) {
        System.out.println("spring boot 自动配置的全局 AuthenticationManager");
    }
    ```

2. 自定义全局认证数据源

    ```java
    // 自定义配置 AuthenticationManager
    @Override
    public void configure(AuthenticationManagerBuilder builder) {
      builder.userDetailsService(customerUserDetailsService())
    }
    // 暴露本地的 AuthenticationManager 自定义实例
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
      return super.authenticationManagerBean();
    }
    ```

**小结：**

1. 默认自动配置全局 AuthenticationManager 默认找当前项目中是否存在自定义 UserDetailService 实例，如果存在会自动将当前项目的 UserDetailService 实例设置为数据源。
2. 默认自动配置全局 AuthenticationManager 在工厂中使用时直接在代码中注入即可。
3. 一旦通过自定义的方式配置后，会 **自动覆盖** 默认的实现；并且需要在实现中 **指定** 自定义的数据源对象 UserDetailsService 实例，并且这个 AuthenticationManager 需要暴露出来（自定义的默认是本地的，不允许其它组件注入）。

### 3. 密码加密

常见加密方案：Hash算法+盐
单向自适应函数（占用大量系统资源，每个认证请求都会大大降低应用程序的性能），开发者可以通过bcrypt、PBKDF2、sCrypt以及argon2来体验这种自适应单向函数加密。

密码的认证是由 PasswordEncoder 进行的，他可以根据不同的加密实现不同的认证方式，所以非常的灵活。

自定义密码加密有两种方式，一种是直接指定密码加密的方式（比如Bcrypt），另一种是使用自定义升级的方式。

### 4. RememberMe

RememberMe是一种服务端的行为，并非是把用户名密码保存在Cookie中的信息。传统的登录方式基于Session会话，一旦用户的会话过期，就要再次登录，这样就太过于繁琐。如果有一种机制，会话过期后，还能继续保持认证状态，就会方便很多,RememberMe就是为了解决这一需求而生的。

具体的实现思路就是通过Cookie来记录当前用户身份，用户登录成功之后，会通过一定算法，将用户信息时间戳等进行贾母，加密完成后，通过响应头带回前端存储再Cookie中，当浏览器会话过期之后，如果再次访问网站，会自动将Cookie中的信息发送给服务器，服务器对Cookie中的信息进行校验分析，进而确定出用户的身份，Cookie中所保存的用户信息也是有失效的，例如三天、一周等。

认证成功后写一段信息在Cookie中

### 5. 会话管理(SessionManagementFilter)

会话并发管理：简单来说，就是多个客户端使用同一账户登录。默认情况下，同一账户可以再多少设备上登录并没有限制，我们可以在 Spring Security 中进行配置。

开启会话管理

```java
// 自定义配置
@Override
public void configure(HttpSecurity http) throw Exception {
    http...
        .sessionManagement() // 开启会话管理
        // 最大并发会话为 1
        .maximumSessions(1);
}
```

Spring Security 开启会话管理默认的是挤掉另一个客户端登录；我们可以设置为禁止其它客户端登录（当前用户登录成功，其它客户端无法使用当前的账户登录，除非当前用户注销退出）。集群下的会话管理可以使用 Redis 的Session 共享。

### 6. 跨域问题

1. 入门：@CrossOrigin
该注解由 Spring 框架提供，含有属性：
allowCredentials:浏览器是否应当发送凭证信息，如 Cookie。
allowHeader:允许访问头
origins:允许指定访问的所属域访问，`*` 表示所有字段。
exposeHeaders:哪些响应头可以作为响应的一部分暴露出来。使用通配符无效。
maxAge:预检查请求的有效期，有效期内不必再次发送预检请求，默认是100s。
methods:允许的请求方法，`*` 表示所有方法。

2. 进阶：addCrosMapping（Spring MVC 提供）

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowCredentials(false)
                .allowedHeaders("*")
                .allowedMethods("*")
                .allowedOrigins("*")
                .maxAge(3600);
    }
}
```

3. CrosFilter（Spring Web提供的跨域处理解决方案）

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

import java.util.Arrays;

@Configuration
public class WebMvcConfig {
    FilterRegistrationBean<CorsFilter> corsFilter() {
        FilterRegistrationBean<CorsFilter> registrationBean = new FilterRegistrationBean<>();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.setAllowedHeaders(Arrays.asList("*"));
        corsConfiguration.setAllowedMethods(Arrays.asList("*"));
        corsConfiguration.setAllowedOrigins(Arrays.asList("*"));
        corsConfiguration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfiguration);
        registrationBean.setFilter(new CorsFilter(source));
        registrationBean.setOrder(-1);
        return registrationBean;
    }
}
```

4. Spring Security中的跨域解决方案
**Q：** 当项目中使用了Spring Security后，上面的方案有的会失效，有的可以继续使用，这是为什么？
**A：** 通过 `@CrossOrigin` 注解或者重写 `addCorsMappings` 方法配置的跨域解决方案，统统失效了（发送的预检请求会被 Spring Security 拦截）；通过 CorsFilter 配置的跨域，有没有失效则要看过滤器的优先级。如果过滤优先级高于 Spring Security 过滤器，即先于 Spring Security 过滤器执行，则 CorsFilter 所配置的跨域处理依然有效；如果优先级低于 Spring Security 过滤器，则 CorsFilter 所配置的跨域处理就会失效。

```java
@Configuration
public class WebMvcConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest()
                .authenticated().and()
                .formLogin().and()
                // 跨域处理方案
                .cors().configurationSource(corsConfigurationSource()).and()
                .csrf().disable();
    }
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.setAllowedHeaders(Arrays.asList("*"));
        corsConfiguration.setAllowedMethods(Arrays.asList("*"));
        corsConfiguration.setAllowedOrigins(Arrays.asList("*"));
        corsConfiguration.setMaxAge(3600L);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfiguration);
        return source;
    }
}
```

### 7. 异常处理

#### 7.1 认证异常

认证异常涉及的异常类型比较多，AuthenticationException 是所有认证异常的父类。

#### 7.2 授权异常

相较于认证异常，权限异常类就要少了很多。在实际项目开发中，如果默认提供的异常无法满足需求，就需要根据实际需求自定义异常类。

#### 7.3 自定义异常处理配置

```java
@Configuration
public class WebMvcConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().mvcMatchers("/hello").hasRole("ADMIN")
                .anyRequest()
                .authenticated().and()
                .formLogin().and()
                .csrf().disable()
                // 异常处理
                .exceptionHandling()
                // 认证异常
                .authenticationEntryPoint((request, response, authException) -> {
                    response.setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
                    response.setStatus(HttpStatus.UNAUTHORIZED.value());
                    response.getWriter().write("尚未认证，请进行认证操作");
                })
                // 权限异常
                .accessDeniedHandler((request, response, accessDeniedException) -> {
                    response.setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
                    response.setStatus(HttpStatus.FORBIDDEN.value());
                    response.getWriter().write("无权访问");
                });
    }
}
```
### 8. CSRF（Cross-site request forgery）

CSRF 即跨站请求伪造，是 web 常见的攻击方式之一。

Spring Security 防止 CSRF 攻击的方式就是通过 csrf_token，前端发起请求的时候需要携带这个 csrf_token，后端会有过滤器进行校验，如果没有携带或者是伪造的就不允许访问。

CSRF 攻击依靠的是 Cookie 中所携带的认证信息。但是在前后端分离的项目我们的认证信息是基于 Token，而 token 并不存储在 Cookie 中，我们的 token 是设置到请求头中的，所以 CSRF 攻击在前后端分离的项目中就失效了。

## 二、授权

### 2.1 两种权限管理策略

- 基于过滤器的权限管理（FilterSecurityInterceptor）
主要拦截Http请求，拦截下来后，根据Http请求地址进行权限校验
- 基于方法的权限管理（MethodSecurityInterceptor）
主要是用来处理方法级别的权限问题。当需要调用某一个方法时，基于 AOP 将操作拦截下来，然后判断用户是否具备相关的权限。

#### 1. 基于URL权限管理

- antMatchers()：最早出现的，用于任何 HttpMethod 请求列表。

- mvcMatchers()：4.x 版本新增的，使用 Spring MVC 的匹配规则。例如，路径 `/path`，它会匹配 `/path`，`/path/`，`/path.html` 等。如果 Spring MVC 不会处理当前请求，会使用 antMatchers()。

  两种方式本质上没有太大区别。mvcMatchers() 指定的话优先使用 MVC 匹配，如果匹配不到，会使用 antMatchers() 匹配。

- regexMatchers()；支持正则表达式。

#### 2. 基于方法的权限管理

这种方式就是在 controller 层的方法上加上注解，指明需要哪些权限或者具有的角色才能访问对应的资源。

首先，配置类中开启方法权限校验：@EnableGlobalMethodSecurity(prePostEnabled = true, SecuredEnable = true, jsr250Enabled = true)，~~（这里开启了三种方式，一般只用第一种）~~

```java
// 开启配置 启用方法级别的权限认证
@Configuration
// 开启之后就可以使用 @preAuthorize 也是最常用的注解
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig{
  ...
}

// 前置校验
@preAuthorize("hasAuthority("权限名称")")
// 过滤 filterTarget 必须是数组 集合，不能过滤某个对象
@preFilter(filterTarget = "users")
// 后置校验
@PostAuthorize()
@PostFilter()

@Secured("role1", "role2") // 只能判断角色，或者 关系，满足一个即通过请求
// 允许所有用户访问
@PermitAll()
// 拒绝所有访问
@DenyAll()
// jsr250 提供的注解，和 Secured() 类似
@RolesAllowed("role1", "role2")
```

项目中，一般前四个用的较多，后面几个功能较为单一。

### 2.2 动态权限修改

以上方式都是通过代码实现的，比较难以修改，可以使用动态的权限修改，比如从数据库层面实现，修改字段即可达到修改相应的权限的目的。

实现 FilterInvocationSecurityMetaSource 接口，重写 getAttributes() 方法。

之后再 SecurityConfig 配置类中，使用 http.apply() 方法，覆写里面的配置，setSecurityMetaSource() 改为实现 FilterInvocationSecurityMetaSource 接口的实现类即可。setRejectPublicInvocations() 设为 true/false，为 true 时拒绝 **公共资源**（没有权限管理的公共接口） 的访问。

### 2.3 RBAC(Role-Based Access Control)

RBAC 即基于角色的权限控制。为什么说基于角色呢？

想像一下，每次添加用户时都单独添加权限的话，那肯定是比较繁琐的。如果我们加入一个“角色”，把常用的一些权限分配给预定义的角色，然后在添加用户的时候直接把“角色”（比如 admin）设置给用户，这个用户就自动拥有对应的权限。这样就比较简单方便。
