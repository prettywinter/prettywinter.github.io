---
title: Spring Security总结
categories: skill
tags: [Java, Spring Security]
abbrlink: 95588d77
---

Spring Security 是一个功能强大且高度可定制的身份验证和访问控制框架。 它是保护基于 Spring 的应用程序的事实标准。

Spring Security 是一个专注于为 Java 应用程序提供身份验证和授权的框架。 像所有 Spring 项目一样，Spring Security 的真正强大之处在于它可以轻松扩展以满足自定义需求。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=true} -->

<!-- code_chunk_output -->

- [核心类](#核心类)
  - [认证常用三个类](#认证常用三个类)
  - [授权常用三个类](#授权常用三个类)
  - [总结](#总结)
    - [AuthenticationManager ProviderManager AuthencationProvider关系](#authenticationmanager-providermanager-authencationprovider关系)
      - [WebSecurityConfigurerAdapter](#websecurityconfigureradapter)
      - [UserDetailsService 用来修改默认认证的数据源信息](#userdetailsservice-用来修改默认认证的数据源信息)
- [配置AuthenticationManager的两种方式](#配置authenticationmanager的两种方式)
- [密码加密](#密码加密)
- [RememberMe](#rememberme)
- [会话管理(SessionManagementFilter)](#会话管理sessionmanagementfilter)
- [CORS](#cors)
- [跨域问题](#跨域问题)
- [异常处理](#异常处理)
  - [1. 认证异常](#1-认证异常)
  - [2. 授权异常](#2-授权异常)
  - [自定义异常处理配置](#自定义异常处理配置)
- [授权](#授权)
  - [两种权限管理策略](#两种权限管理策略)
    - [基于URL权限管理](#基于url权限管理)
    - [基于方法的权限管理](#基于方法的权限管理)
  - [动态权限修改](#动态权限修改)

<!-- /code_chunk_output -->

Spring Security 的版本在 2.7.0 版本之前（不包括）可以使用 WebSecurityConfigurerAdapter 来配置相关内容。2.7.0 版本之后有所改变（弃用状态，仍可使用）。[官网配置](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html)

## 核心类

[体系结构](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)

### 认证常用三个类

- AuthenticationManager（authenticate()，返回 Authentication 表示认证成功，返回 AuthenticationException 认证失败）
    > AuthenticationManager 主要实现类为 ProviderManager，在 ProviderManager 中管理了众多的 AuthenticationProvider 实例，在一次完整的认证流程中，允许存在多个 AuthenticationProvider，用来实现多种认证方式，这些 AuthenticationProvider 实例都是由 ProviderManager 统一管理的。
- Authentication（保存认证以及认证成功的信息）
- SecurityContextHolder（取出用户信息）
  > 基于 threadLocal 线程绑定，请求时把 session 中的用户信息放入 SecurityContextHolder 中，请求结束后清空信息；之后的请求同理。

### 授权常用三个类

AccessDeclsionManager（访问决策管理器）此次请求是否放行资源
AccessDecislonVoter（访问决定投票器）此次请求是否具有访问资源的权限
ConfigAttribute（保存授权时的角色信息）

ProviderManager 是一个接口
AuthenticationProvider 是它的一个实现。
DaoAuthenticationProvider 通过调用 retrieveUser() 认证。

思考：

- 为什么在引入 Spring Security 之后没有任何配置所有的请求就要认证呢？
  SpringBootWebSecurityConfiguration，自动配置类
- 在项目中明明没有登录界面，登录界面怎么来的呢？
  访问接口时，在引入 spring security 之后会先经过一系列过滤器。在请求到达 FilterSecurityInterceptor 时，发现请求并未认证，请求拦截下来，并抛出 AccessDeniedException 异常。抛出的异常会被 ExceptionTranslationFilter 捕获，这个 Filter 中会调用 LoginUrlAuthenticationEntryPoint#commence 方法给客户端返回 302，要求客户端进行重定向到 /login 页面。客户端发送 /login 请求。/login 请求会再次被拦截器中 DefaultLoginPageGeneratingFilter 拦截到，并在拦截器中返回生成登录页面。
- 为什么使用 user 和控制台打印的密码能登录，登录时验证数据源存在哪里呢？
  Spring Security中有一个基于内存（InMemoryUserDetailsManager）的默认用户实现。

### 总结

#### AuthenticationManager ProviderManager AuthencationProvider关系

AuthenticationManager 全局的父接口，只有一个 authenticate 方法。需要 ProviderManager 实现。ProviderManager 遍历下面的 AuthencationProvider 认证，只要有一个认证通过即可。

##### WebSecurityConfigurerAdapter

WebSecurityConfigurerAdapter 是 Spring Security 为我们提供的扩展类，方便我们重写默认配置，实现定制。

##### UserDetailsService 用来修改默认认证的数据源信息

UserDetailsService接口下有许多的实现。同时，此接口也方便了我们以后的自定义数据源的扩展。

## 配置AuthenticationManager的两种方式

1. 继承 WebSecurityConfigurerAdapter，springboot 对 security 默认配置中 在工厂默认创建 AuthenticationManager。

    ```java{.line-numbres}
    // 默认配置会自动发现创建的 UserDetailService 的 Bean
    @Autowired
    public void initialize(AuthenticationManagerBuilder builder) {
        System.out.println("spring boot 默认配置");
    }
    ```

2. 自定义全局认证数据源

    ```java{.line-numbres}
    // 自定义配置
    @Override
    public void configure(AuthenticationManagerBuilder builder) {
        
    }
    ```

**总结：**

1. 默认自动配置全局 AuthenticationManager 默认找当前项目中是否存在自定义 UserDetailService 实例，自动将当前项目的 UserDetailService 实例设置为数据源。
2. 默认自动配置全局 AuthenticationManager 在工厂中使用时直接在代码中注入即可。
3. 一旦通过自定义的方式配置后，会自动覆盖默认的实现；并且需要在实现中指定自定义的数据源。

## 密码加密

常见加密方案：Hash算法+盐
单向自适应函数（占用大量系统资源，每个认证请求都会大大降低应用程序的性能），开发者可以通过bcrypt、PBKDF2、sCrypt以及argon2来体验这种自适应单向函数加密。

密码的认证是由 PasswordEncoder 进行的，他可以根据不同的加密实现不同的认证方式，所以非常的灵活。

自定义密码加密有两种方式，一种是直接指定密码加密的方式（比如Bcrypt），另一种是使用自定义升级的方式。

## RememberMe

RememberMe是一种服务端的行为，并非是把用户名密码保存在Cookie中的信息。传统的登录方式基于Session会话，一旦用户的会话过期，就要再次登录，这样就太过于繁琐。如果有一种机制，会话过期后，还能继续保持认证状态，就会方便很多,RememberMe就是为了解决这一需求而生的。

具体的实现思路就是通过Cookie来记录当前用户身份，用户登录成功之后，会通过一定算法，将用户信息时间戳等进行贾母，加密完成后，通过响应头带回前端存储再Cookie中，当浏览器会话过期之后，如果再次访问网站，会自动将Cookie中的信息发送给服务器，服务器对Cookie中的信息进行校验分析，进而确定出用户的身份，Cookie中所保存的用户信息也是有失效的，例如三天、一周等。

认证成功后写一段信息在Cookie中

## 会话管理(SessionManagementFilter)

会话并发管理：简单来说，就是多个客户端使用同一账户登录。默认情况下，同一账户可以再多少设备上登录并没有限制，我们可以在 Spring Security 中进行配置。

开启会话管理

```java{.line-numbers}
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

## CORS

## 跨域问题

1. 入门：@CrossOrigin
该注解由 Spring 框架提供，含有属性：
allowCredentials:浏览器是否应当发送凭证信息，如 Cookie。
allowHeader:允许访问头
origins:允许指定访问的所属域访问，`*` 表示所有字段。
exposeHeaders:哪些响应头可以作为响应的一部分暴露出来。使用通配符无效。
maxAge:预检查请求的有效期，有效期内不必再次发送预检请求，默认是100s。
methods:允许的请求方法，`*` 表示所有方法。

2. 进阶：addCrosMapping
Spring MVC 提供：
```java{.line-numbers}
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

1. CrosFilter
Spring Web提供的跨域处理解决方案：
```java{.line-numbers}
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
当项目中使用了Spring Security后，上面的方案有的会失效，有的可以继续使用，这是为什么？

通过 `@CrossOrigin` 注解或者重写 `addCorsMappings` 方法配置的跨域解决方案，统统失效了（发送的预检请求会被 Spring Security 拦截）；通过 CorsFilter 配置的跨域，有没有失效则要看过滤器的优先级。如果过滤优先级高于 Spring Security 过滤器，即先于 Spring Security 过滤器执行，则 CorsFilter 所配置的跨域处理依然有效；如果优先级低于 Spring Security 过滤器，则 CorsFilter 所配置的跨域处理就会失效。

```java{.line-numbers}
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

## 异常处理

### 1. 认证异常

认证异常涉及的异常类型比较多，AuthenticationException 是所有认证异常的父类。

### 2. 授权异常

相较于认证异常，权限异常类就要少了很多。在实际项目开发中，如果默认提供的异常无法满足需求，就需要根据实际需求自定义异常类。

### 自定义异常处理配置

```java{.line-numbers}
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

## 授权

权限很好理解，有权访问和无权访问，RBAC模式

### 两种权限管理策略

- 基于过滤器的权限管理（FilterSecurityInterceptor）
主要拦截Http请求，拦截下来后，根据Http请求地址进行权限校验
- 基于方法的权限管理（MethodSecurityInterceptor）
主要是用来处理方法级别的权限问题。当需要调用某一个方法时，基于 AOP 将操作拦截下来，然后判断用户是否具备相关的权限。

#### 基于URL权限管理

mvcMatchers()
antMatchers()

#### 基于方法的权限管理

这种方式就是在 controller 层的方法上加上注解，指明需要哪些权限或者具有的角色才能访问对应的资源。

首先，配置类中开启方法权限校验：@EnableGlobalMethodSecurity(prePostEnabled = true, SecuredEnable = true, jsr250Enabled = true)，~~（这里开启了三种方式，一般只用第一种）~~

```java
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

### 动态权限修改

以上方式都是通过代码实现的，比较难以修改，可以使用动态的权限修改，比如从数据库层面实现，修改字段即可达到修改相应的权限的目的。

实现 FilterInvocationSecurityMetaSource 接口，重写 getAttributes() 方法。

之后再 Security 的配置类中，使用 http.apply() 方法，覆写里面的配置，setSecurityMetaSource() 改为实现 FilterInvocationSecurityMetaSource 接口的实现类即可。setRejectPublicInvocations() 设为 true/false，为 true 时拒绝 **公共资源**（没有权限管理的公共接口） 的访问。
