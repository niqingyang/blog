---
title: "Spring Security – 基本原理"
date: 2019-11-02T18:03:56+08:00
draft: false
categories:
    - develop
tags:
    - spring security
    - spring
    - java
themeColor: "#137300"
themeColor: "#137300f7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410225058-spring-security.png
---

## 前言

在上一篇文章《Spring Security - 初体验》中简单的使用了一下 Spring Security，了解了一些简单的配置，初步使用起来感觉还是蛮简单的，不过看似简单的用法，其底层肯定有着精妙的构造，下面就来聊一聊。

## 核心构建

Spring Security 中有很多类，继承来继承去的，刚开始接触的时候看的脑袋疼，先在这里捋一遍，认识认识，免得后面看代码的时候脸盲症猝发~

### SecurityContextHolder

在看官方文档的时候会发现有介绍，当前使用该应用程序的用户详细信息会存储在应用程序的安全上下文中，这个`安全上下文`指的就是 `SecurityContext`，默认情况下，`SecurityContext` 被 `SecurityContextHolder` 存储在 `ThreadLocal` 中，这就意味着在同一线程中我们可以从 ThreadLocal 中获取到当前的 SecurityContext，考虑到线程池的原因，需要在每次请求完成后对 ThreadLocal 进行清理才能保证 SecurityContext 存放在 ThreadLocal 中是安全的，而这部分工作 Spring Security 已经自动帮我们做了，每次请求结束后他都会去清理 ThreadLocal。

SecurityContext 的存储方式是通过 SecurityContextHolder 的属性 `SecurityContextHolderStrategy` 来实现的，Spring Security 提供了三种存储方式，并为每种方式定义了名称，分别是：

- MODE_THREADLOCAL - 缺省工作模式，实现类为 `ThreadLocalSecurityContextHolderStrategy`，使用 ThreadLocal 存储安全上下文，适合 Servlet Web 这种 B/S 架构的应用，线程安全。
- MODE_GLOBAL - 实现类为 `GlobalSecurityContextHolderStrategy`，JVM 中所有线程使用同一个安全上下文，适合 C/S 架构客户端应用。
- MODE_INHERITABLETHREADLOCAL - 实现类为 `InheritableThreadLocalSecurityContextHolderStrategy`， 支持在子线程中使用父线程中的变量。

可以通过 `SecurityContextHolder.setStrategyName()` 或者设置系统属性 `spring.security.strategy` 为对应的 StrategyName 名称即可。

常见的获取当前已验证用户的名称：

```java
// 通过 Authentication.getPrincipal() 可以获取到代表当前用户的信息，这个对象通常是 UserDetails 的实例
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
	String username = ((UserDetails)principal).getUsername();
} else {
	String username = principal.toString();
}
```

### Authentication

Authentication 是一个接口，用来表示用户认证信息的（例如用户密码、密码），在用户登录认证之前相关信息会封装为一个 Authentication 具体实现类的对象，在登录认证成功之后又会生成一个信息更全面，包含用户权限等信息的 Authentication 对象，然后把它保存在 SecurityContextHolder 所持有的 SecurityContext 中，供后续的程序进行调用，如访问权限的鉴定等。

![Authentication 的子类](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205710-paste-14c1241c35eae595c084cbd88b7de5ed-1.png)

### UserDetailsService

通过 Authentication.getPrincipal() 的返回类型是 Object，但很多情况下其返回的其实是一个 UserDetails 的实例。UserDetails 是 Spring Security 中一个核心的接口。它代表一个 Principal，可以认为它是我们具体用户数据与 SpringSecurityContextHolder 之间的适配器，其中定义了一些可以获取用户名、密码、权限等与认证相关的信息的方法，在具体业务中可以转换为应用程序的原始对象，从而获得调用具体业务的方法（例如：getEmail()，getEmployeeNumber()）。

UserDetails 对象可以通过 UserDetailsService 获取，很多时候都需要我们根据具体业务去自定义实现：

```java
UserDetails loadUserByUsername（String username）throws UsernameNotFoundException;
```

> UserDetailsService 的功能很纯粹，仅仅是用于用户信息的加载，除了将用户数据提供给框架内的其他组件外，不执行任何其他功能。不应该在这里进行用户的身份验证，用户的身份验证是由 AuthenticationManager 完成的，如果仅仅是想自定义身份验证，那么实现一个自定义的 AuthenticationProvider 更有意义。

在 Spring Security 内部很多地方需要使用用户信息的时候基本上都是使用的 UserDetails，比如在登录认证的时候。登录认证的时候 Spring Security 会通过 UserDetailsService 的 loadUserByUsername() 方法获取对应的 UserDetails 进行认证，认证通过后会将该 UserDetails 赋给认证通过的 Authentication 的 principal，然后再把该 Authentication 存入到 SecurityContext 中。之后如果需要使用用户信息的时候就是通过 SecurityContextHolder 获取存放在 SecurityContext 中的 Authentication 的 principal。

![UserDetailsService 实现类](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205718-paste-a1f7cf5eae7ff958b56e9b957fb59564-1.png)

### GrantedAuthority

Authentication 的 getAuthorities() 可以返回当前 Authentication 对象拥有的权限，即当前用户拥有的权限。其返回值是一个 GrantedAuthority 类型的数组，每一个 GrantedAuthority 对象代表赋予给当前用户的一种权限。GrantedAuthority 是一个接口，其通常是通过 UserDetailsService 进行加载，然后赋予给 UserDetails 的。

GrantedAuthority 中只定义了一个 getAuthority() 方法，该方法返回一个字符串，表示对应权限的字符串表示，如果对应权限不能用字符串表示，则应当返回 null。

Spring Security 针对 GrantedAuthority 有一个简单实现 SimpleGrantedAuthority。该类只是简单的接收一个表示权限的字符串。Spring Security 内部的所有 AuthenticationProvider 都是使用 SimpleGrantedAuthority 来封装 Authentication 对象。

![GrantedAuthority 实现类](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205724-paste-8cfd4b99089c166ac5741025bf065e2c-1.png)

### AuthenticationManager

AuthenticationManager 是一个用来处理认证（Authentication）请求的接口。在其中只定义了一个方法 authenticate()，该方法只接收一个代表认证请求的 Authentication 对象作为参数，如果认证成功，则会返回一个封装了当前用户权限等信息的 Authentication 对象进行返回。

```java
Authentication authenticate(Authentication authentication) throws AuthenticationException;
```

### AuthenticationProvider

在 Spring Security 中，AuthenticationManager 的默认实现是 ProviderManager，而且它不直接自己处理认证请求，而是委托给其所配置的 AuthenticationProvider 列表，然后会依次使用每一个 AuthenticationProvider 进行认证，如果有一个 AuthenticationProvider 认证后的结果不为 null，则表示该 AuthenticationProvider 已经认证成功，之后的 AuthenticationProvider 将不再继续认证。然后直接以该 AuthenticationProvider 的认证结果作为 ProviderManager 的认证结果。如果所有的 AuthenticationProvider 的认证结果都为 null，则表示认证失败，将抛出一个 ProviderNotFoundException。

校验认证请求最常用的方法是根据请求的用户名加载对应的 UserDetails，然后比对 UserDetails 的密码与认证请求的密码是否一致，一致则表示认证通过。Spring Security 内部的 DaoAuthenticationProvider 就是使用的这种方式。其内部使用 UserDetailsService 来负责加载 UserDetails]。在认证成功以后会使用加载的 UserDetails 来封装要返回的 Authentication 对象，加载的 UserDetails 对象是包含用户权限等信息的。认证成功返回的 Authentication 对象将会保存在当前的 SecurityContext 中。

### 概要

对上门涉及的类做个简单的概要说明：

- `SecurityContextHolder`，提供获取安全上下文实例 SecurityContext。
- `SecurityContext`，保存特定请求的安全信息 Authentication。
- `Authentication`，在 Spring Security 中表示认证用户信息的主体。
- `GrantedAuthority`，表示应用程序授予已认证主体的权限范围。
- `UserDetails`，提供从应用程序的 DAO 或其他安全数据源构建 Authentication 对象所需的信息。
- `UserDetailsService`，创建一个基于 String 类型的用户名（或证书ID等）的 UserDetails 实例。
- `AuthenticationManager`，用来处理认证请求的接口，但不直接进行认证。
- `AuthenticationProvider`，真正处理认证请求的接口。

## 基本原理

Spring Boot 项目中通过引入依赖 `spring-boot-starter-security`，就自动开启 spring 对 spring security 的支持，默认会对访问的所有资源要求提供身份认证（提供默认的登录页面并会在控制台输出随机生成的密码）。

Spring Security 中通过继承适配器类 `WebSecurityConfigurerAdapter` 来覆盖默认配置，为其实现类加上 `@EnableWebSecurity` 注解后，自定义的配置才会生效。在 `WebSecurityConfigurerAdapter` 中可以通过 getHttp() 获取到 HttpSecurity 对象，可以通过覆盖 `userDetailsService` 方法实现自定义获取用户，通过覆盖 `authenticationManager` 方法实现自定义认证管理。

Spring Security 的核心是 `FilterChain` （即 过滤器链），一个 `FilterChain` 中可以包含多个 `Filter` （即 过滤器），Spring Security 底层通过 Filter 来工作，每个 Filter 都有起各自的功能，而且各个 Filter 之前还有关联关系，所以他们的顺序非常重要，特别是 Spring Security 已经定义了的一些 Filter，他们的执行顺序是固定的，从 `FilterComparator` 类中可以看到默认的 Filter 及其顺序如下：

```java
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

Spinrg Security 中可以配置多个 HttpSecurity，每个 HttpSecurity 的配置都会创建相应的 FilterChain 来处理对应的请求，每个请求都经过 FilterChainProxy，根据请求选择一个合适的过滤器链来处理该请求

```java
	/**
	 * Returns the first filter chain matching the supplied URL.
	 *
	 * @param request the request to match
	 * @return an ordered array of Filters defining the filter chain
	 */
	private List<Filter> getFilters(HttpServletRequest request) {
		for (SecurityFilterChain chain : filterChains) {
			if (chain.matches(request)) {
				return chain.getFilters();
			}
		}
		return null;
	}
```

查看过几个过滤器的源码后，会发现大部分过滤器都继承自 GenericFilterBean

![GenericFilterBean 的子类](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205823-paste-c45638a211fedcc4fd85887ffbdcbf31-1.png)

上图中黄色标出的类，也就是以 `AuthenticationFilter` 关键词结尾的过滤器，都是用于用户身份认证的过滤器，但默认仅配置了 `UsernamePasswordAuthenticationFilter` 和 `BasicAuthenticationFilter` 这两个过滤器，其他的如果有需要需要自行进行配置。

在过过滤器链中，这些过滤器会根据当前的请求检查是否有这个过滤器所需的信息，如果有则进入该过滤器进行身份认证，没有则不会进入下一个过滤器。例如：

`UsernamePasswordAuthenticationFilter` 只会处理 `POST /login` 请求

![AbstractAuthenticationProcessingFilter](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205830-paste-e5357577f4468f28c4c7bc1edc04d5c9-1.png)

![UsernamePasswordAuthenticationFilter](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205837-paste-9d319d9ed7ed0ff29f792d5a864f3e4b-1.png)

`BasicAuthenticationFilter` 只会处理请求头中包含 `Authorization` 并且以 `basic ` 开头的请求

![BasicAuthenticationFilter](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205843-paste-b1f8722052149ab3c6b2caa303ca8f87-1.png)

类似的这些用于身份认证的 `Filter` 在一次请求过程中仅会进入其中的一个进行处理，而先进入哪一个取决与他们的排序，我们自定义的身份认证过滤器一般都通过 `HttpSecurity` 的 `addFilter`、`addFilterAfter`、`addFilterBefore` 添加在某个身份认证过滤器的前后来控制器验证的先后顺序。

认证过滤器最后会通过 `AuthenticationManager` 的 `authenticate()` 函数对 `Authentication` 进行身份认证

![进行身份认证](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205850-paste-296cca41d6b62f63c94dbb13ff92f812-1.png)

通过源码可以看出，每个类型的身份认证过滤器在执行认证过程中都根据认证信息生成 `Authentication`。使用用户名和密码登录时，就会生成 `UsernamePasswordAuthenticationToken`。

![Authentication 的子类](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205857-paste-14c1241c35eae595c084cbd88b7de5ed-1-20210410205857470.png)

经过前面的过滤器后，最后会进入到 `FilterSecurityInterceptor`，这是整个 spring security 过滤器链的最后一环，在它身后就是服务的 API。`FilterSecurityInterceptor` 通过属性 `AuthenticationManager` (认证管理器) 、 `AccessDecisionManager` （访问授权决策器）和一些配置来决定当前的请求是否可以访问的到真正的资源，其主要功能是在父类 `AbstractSecurityInterceptor` 中实现的。

![AbstractSecurityInterceptor](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205902-paste-ec4fb03d6afc559e42d274462801b453-1.png)

1️⃣ 获取权限的配置属性，用于后面交给 `AccessDecisionManager` 来觉得是否有权访问目标资源
2️⃣ 当前上下文必须有用户认证信息 `Authentication`，匿名访问的请求会通过过滤器 `AnonymousAuthenticationToken` 来生成 `Authentication`。
3️⃣ Authentication 是通过函数 `authenticateIfRequired()` 获取到的，其内部会根据上下文中的 authentication 和属性 `alwaysReauthenticate` 来判断是否需要重新进行认证，不需要则直接返回上下文中已存在的 authentication，否则重新认证并更新上下文中的 authentication 并返回。

![authenticateIfRequired() 中判断是否有必要进行认证](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410210011-paste-053e65ba32a8afab32214a6768c5f828-1.png)

4️⃣ 获取到认证过的 `Authentication` 后，会使用 访问决策管理器 `AccessDecisionManager` 判断是否有权限，管理器会管理者多个 访问决策投票器 `AccessDecisionVoter`，通过投票器来决定是否有权限访问资源。我们也可以自定义投票器来判断用户是否有权限访问某个资源。

![AffirmativeBased](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410210020-paste-57cccc4ac291ba21743f8e571694aa3a-1.png)

5️⃣ 最后，如果未认证通过或没有权限，FilterSecurityInterceptor 则抛出相应的异常，异常会被 ExceptionTranslationFilter 捕捉到，进行统一的异常处理分流，比如未登录时，重定向到登录页面；没有权限的时候抛出403异常等。

![ExceptionTranslationFilter](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410210030-paste-bfb4ffa4058c6516c0898998a02ae4e7-1.png)
