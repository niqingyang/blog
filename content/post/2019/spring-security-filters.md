---
title: "Spring Security - 过滤器"
date: 2019-11-20T21:25:40+08:00
draft: false
categories:
    - develop
tags:
    - spring security
    - spring
    - java
themeColor: "#137300"
themeColor: "#137300f7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410225041-spring-security.png
---

## Spring Security 的过滤器

Spring Boot 项目中通过引入依赖 `spring-boot-starter-security`，就自动开启 spring 对 spring security 的支持，默认会对访问的所有资源要求提供身份认证（提供默认的登录页面并会在控制台输出随机生成的密码）。

Spring Security 中通过继承适配器类 `WebSecurityConfigurerAdapter` 来覆盖默认配置，为其实现类加上 `@EnableWebSecurity` 注解后，自定义的配置才会生效。在 `WebSecurityConfigurerAdapter` 中可以通过 getHttp() 获取到 HttpSecurity 对象，可以通过覆盖 `userDetailsService` 方法实现自定义获取用户，通过覆盖 `authenticationManager` 方法实现自定义认证管理。

Spring Security 的核心是 `FilterChain` （即 过滤器链），一个 `FilterChain` 中可以包含多个 `Filter` （即 过滤器），Spring Security 底层通过 Filter 来工作，每个 Filter 都有起各自的功能，而且各个 Filter 之前还有关联关系，所以他们的顺序非常重要，特别是 Spring Security 已经定义了的一些 Filter，他们的执行顺序是固定的，从 `FilterComparator` 类中可以看到默认的 Filter 及其顺序如下：

### ChannelProcessingFilter

用于决定的是 web 请求的协议，即 http 或 https

```java
http.requiresChannel().anyRequest().requiresInsecure()
```

### ConcurrentSessionFilter

用于对并发会话控制的支持，允许或限制用户可以拥有的活动会话数量

```java
http
	.sessionManagement() // 获取 sessoin 管理器
	.invalidSessionUrl("/session/invalid") // session 失效后跳转地址
	.maximumSessions(1) // 控制同一个用户最大的会话数量
	.expiredUrl("/session/expired") // 用户会员数量达到最大数量后重定向的地址
	.maxSessionsPreventsLogin(true); // 设置为 true 后，将拒绝会话数量达到最大值的用户登录
```

### WebAsyncManagerIntegrationFilter

提供了对 `SecurityContext` 和 Spring Web 的 `WebAsyncManager` 的集成，其通过 `SecurityContextCallableProcessingInterceptor.beforeConcurrentHandling(org.springframework.web.context.request.NativeWebRequest, Callable)` 会把 `SecurityContext` 设置到异步线程中，使其也能获取到用户上下文认证信息

### SecurityContextPersistenceFilter

该类是承接容器的 Session 与 Spring Security 的重要 Filter，会在所有的 Filter 之前执行，主要工作是从 `SecurityContextRepository` 中获取 `SecurityContext`，然后放到上下文中，保证后面的身份认证处理机制（例如：CAS 等）在运行的时候能够从 `SecurityContextHolder` 中获取到一个有效的 `SecurityContext`，以提高效率，避免每一次请求都要查询用户认证信息

在请求完成后，该 Filter 会通过 `SecurityContextHolder.clearContext()` 清除上下文，并通过 `SecurityContextRepository` 将 `SecurityContext` 存储回存储库中。

### HeaderWriterFilter

Spring Securty 使用该 Filter 在一个请求的处理过程中为响应对象增加一些头部信息。头部信息由外部提供，比如用于增加一些浏览器保护的头部，比如 X-Frame-Options, X-XSS-Protection 和 X-Content-Type-Options 等。

使用方式：

```java
http.headers() ....
```

<info>

> 参考文档：  
> [官方文档](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#default-security-headers "https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#default-security-headers")
> [https://blog.csdn.net/andy_zhang2007/article/details/84818677](https://blog.csdn.net/andy_zhang2007/article/details/84818677 "https://blog.csdn.net/andy_zhang2007/article/details/84818677")

</info>

### CorsFilter

该类是 Spring 框架提供的对于 CORS 的支持，必须在 Spring Security 之前处理 CORS，以避免跨域请求中的预检 pre-flight 请求因为没有携带任何 Cookie 而被 Spring Security 的身份验证机制拒绝掉。

**关于 CORS 跨域请求中的预检 pre-flight**
借助 Access-Control-Allow-Origin 响应头字段可以允许跨域 AJAX， 对于**非简单请求**，CORS 机制跨域会首先进行 preflight（一个 OPTIONS 请求）， 该请求成功后才会发送真正的请求。 这一设计旨在确保服务器对 CORS 标准知情，以保护不支持 CORS 的旧服务器。

<info>

> 参考文档：  
> [https://blog.csdn.net/u012207345/article/details/81449683](https://blog.csdn.net/u012207345/article/details/81449683 "https://blog.csdn.net/u012207345/article/details/81449683")
> [https://www.jianshu.com/p/2f264dac6f32](https://www.jianshu.com/p/2f264dac6f32 "https://www.jianshu.com/p/2f264dac6f32")
> [https://www.cnblogs.com/SilenceTom/p/6697484.html](https://www.cnblogs.com/SilenceTom/p/6697484.html "https://www.cnblogs.com/SilenceTom/p/6697484.html")

</info>


确保首先处理 CORS 的最简单方法是使用 CORSFilter。可以使用以下方法提供 CorsConfigurationSource，从而将 CorsFilter 与Spring Security 集成在一起：

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // by default uses a Bean by the name of corsConfigurationSource
            .cors().and()
            ...
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET","POST"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

如果项目中集成了 SpringMVC 的 CORS 支持，并且没有指定 CORSConfigurationSource，那么在 Spring Security 中可以例如如下设置提供 CORS 配置提供给 SpringMVC

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // if Spring MVC is on classpath and no CorsConfigurationSource is provided,
            // Spring Security will use CORS configuration provided to Spring MVC
            .cors().and()
            ...
    }
}
```

### CsrfFilter

该类用于 Spring Security 中提供对防范 CSRF 攻击的支持，其私有属性 requireCsrfProtectionMatcher 使用默认的匹配器 DefaultRequiresCsrfMatcher 仅限制了 POST 请求，对 GET、HEAD、TRACE、OPTIONS 请求均予以放行，可以通过 HttpSecurity 中的 CsrfConfigurer 配置类来设置路由的匹配，也可以自定义匹配器

```java
http.csrf()
```

### LogoutFilter

处理退出登录的 Filter，如果请求的 url 为 /logout 则会执行退出登录操作。注销后，将根据使用的构造函数，对由配置的LogoutSuccessHandler 或 LogoutSuccessURL 确定的 URL 执行重定向。

```java
http.logout()
```

### X509AuthenticationFilter

提供对 X509 的支持，可参考 [官方文档](X509AuthenticationFilter "官方文档")

### CasAuthenticationFilter

该 Filter 用于 Spring Security 提供对 CAS 的支持，具体工作原理和使用方法可参考 [官方文档](CasAuthenticationFilter "官方文档")

### UsernamePasswordAuthenticationFilter

该 Filter 用于处理来自表单提交用户名和密码，进行用户实际身份验证的类，是最常用的身份验证过滤器，也是最常定制的过滤器。具体使用方法可参考上面的 “自定义登录页” 或参考 [官方文档](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#form-login-filter "官方文档")

### DigestAuthenticationFilter & BasicAuthenticationFilter

这两个 Filter 分别负责 `摘要式身份验证` 和 `基本身份验证`，是Web应用程序中常用的备用身份验证机制。

关于摘要式身份验证可以参考文档：https://www.cnblogs.com/huey/p/5490759.html


```java

ConcurrentSessionFilter
OpenIDAuthenticationFilter
DefaultLoginPageGeneratingFilter
DefaultLogoutPageGeneratingFilter
ConcurrentSessionFilter
DigestAuthenticationFilter
BearerTokenAuthenticationFilter
BasicAuthenticationFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
JaasApiIntegrationFilter
RememberMeAuthenticationFilter
AnonymousAuthenticationFilter
OAuth2AuthorizationCodeGrantFilter
SessionManagementFilter
ExceptionTranslationFilter
FilterSecurityInterceptor

// 默认
WebAsyncManagerIntegrationFilter
SecurityContextPersistenceFilter
HeaderWriterFilter
CsrfFilter
LogoutFilter
UsernamePasswordAuthenticationFilter
BasicAuthenticationFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
AnonymousAuthenticationFilter
SessionManagementFilter
ExceptionTranslationFilter
FilterSecurityInterceptor
```

未完待续...
