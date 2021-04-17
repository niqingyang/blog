---
title: "Spring Security - 初体验"
date: 2019-11-02T18:03:03+08:00
draft: false
categories:
    - develop
tags:
    - spring security
    - spring
    - java
themeColor: "#137300"
themeColor: "#137300f7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410225114-spring-security.png
---

## 简介

Spring Security，这是一种基于 Spring AOP 和 Servlet 过滤器的安全框架。它为基于 Java EE 的企业软件应用程序提供全面的安全性解决方案，同时在 Web 请求级和方法调用级处理 用户认证（Authentication）和 用户授权（Authorization）。它提供了强大的企业安全服务，如：认证授权机制、Web资源访问控制、业务方法调用访问控制、领域对象访问控制 Access Control List（ACL）、单点登录（SSO）等等。

- **用户认证** 一般要求用户提供用户名和密码，系统通过校验用户名和密码来完成认证过程。
- **用户授权** 指的是验证某个用户是否有权限执行某个操作，一般系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

Spring Security的主要核心功能为 认证 和 授权，所有的架构也是基于这两个核心功能去实现的。

## 初体验

1. Gradle 依赖配置

build.gradle

```groovy

repositories {
	mavenCentral()
    maven { url 'https://repo.spring.io/milestone' }
}

dependencies {
	compile 'org.springframework.boot:spring-boot-starter-security'
	compile 'org.springframework.boot:spring-boot-starter-web'
}
```

由于Spring Boot提供 Maven BOM 来管理依赖版本，因此无需指定版本。如果您希望覆盖 Spring Security 版本，可以通过提供 Gradle 属性来实现：

```groovy
ext['spring-security.version']='5.1.4.RELEASE'
```

修改 Spring Boot 版本

```groovy
ext['spring.version']='5.1.5.RELEASE'
```

2. 创建程序入口 HelloWorldApplication

```java
package org.springframework.security.samples;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloWorldApplication {
	public static void main(String[] args) {
		SpringApplication.run(HelloWorldApplication.class, args);
	}
}
```

3. 创建一个控制器 MainController

```java
package org.springframework.security.samples.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MainController {
	@GetMapping("/hello")
	@ResponseBody
	public String hello() {
		return "Hello World";
	}
}
```

4. 不做任何配置，启动 Spring Boot 后，访问 http://localhost:8080/hello 时，会自动跳转到 Spring Security 默认的登录页面要求认证

![默认登录页面](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210234-paste-f7487c0168b8796f0cae13186f6df057-1.png)

项目启动后，控制台中会输出 Spring Security 生成的随机密码，用户名为 `user`

![默认控制输出随机的密码](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210242-paste-916b8fada307d016527bae3971427a91-1.png)

![UserDetailsServiceAutoConfiguration](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210250-paste-39f5c07cf40df701d01b95e3961719aa-1.png)

![SecurityProperties](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210300-paste-e86febe6cb21d05e81a1c2f2887565c9-1.png)

从控制台输出的日志可以追踪到，密码是在类 `UserDetailsServiceAutoConfiguration` 中输出的，进入此类后可以看到其使用的是基于内存的用户管理器 `InMemoryUserDetailsManager`，属性配置类 `SecurityProperties` 中定义了默认的用户名为 `user`，可以看到密码是随机生成的 UUID，因此我们也可以在配置中指定 用户名、密码、角色：

```properties
spring.security.user.name=user
spring.security.user.password=123456
# 角色不需要加前缀 “ROLE_”
spring.security.user.roles[0]=USER
spring.security.user.roles[1]=ADMIN
```

5. 使用 `user` 和 控制台的显示的 UUID 密码登录成功后访问 `/hello`，即可看到受保护的资源，一个简单的认证就完成了。

![Hello World](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210308-paste-2143c1110fd8779eea3946e14356904f-1.png)

## 自定义登录页

一般情况下，spring security 默认的登录页和配置是无法满足生产中的需求的，大多数都需要我们去自定义登录页和相关配置，例如修改登录页的风格、表单提交的相关参数和表单提交的地址等。

这里使用 thymeleaf 模板引擎开发登录页，并创建一个只有拥有 `USER` 权限的用户才能访问的地址 `/user`，和一个只有拥有 `ADMIN` 权限才能访问的地址 `/admin`，还有一个允许匿名用户访问的地址 `/index`

1. 首页引入依赖

```groovy
dependencies {
	compile 'org.springframework.boot:spring-boot-starter-thymeleaf'
	// thymeleaf-extras-springsecurity是thymeleaf模板框架与spring security框架的扩展包
	// 主要实现在thymeleaf的模板页面中可以很方便的调用security框架返回的用户验证和权限信息
	compile 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
}
```

默认情况下 thymeleaf 模板引擎会开启缓存，也就意味这开发时修改页面后不会自动更新必须重启服务后才能看到效果，为了开发方便此处在配置文件中关闭缓存：

```properties
spring.thymeleaf.cache=false
```

2. 定义目录结构

在 resources 目录下，建 static 和 templates 两个目录，static 目录用于存放静态资源，templates 用于存放 thymeleaf 模板页面，同时配置 MVC 的静态资源映射

```properties
## 定义静态资源访问的路径以 "/static/" 开头
spring.mvc.static-path-pattern=/static/**
```

3. 开发首页、登录页面的跳转地址，`/login` API 用于向登录页面传递登录相关的数据，如用户名、、错误消息等。

```java
package org.springframework.security.samples.web;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MainController {

	@RequestMapping("/hello")
	@ResponseBody
	public String hello() {
		return "Hello World";
	}

	@RequestMapping("/")
	public String root() {
		return "redirect:/index";
	}

	@RequestMapping("/index")
	public String index() {
		return "index";
	}

	@RequestMapping("/login")
	public String login() {
		return "login";
	}

	@RequestMapping("/login-error")
	public String loginError(Model model) {
		model.addAttribute("loginError", true);
		return "login";
	}

	@RequestMapping("/user")
	public String user() {
		return "user";
	}

	@RequestMapping("/admin")
	public String admin() {
		return "admin";
	}
}

```

4. 编写配置类，继承适配器类 WebSecurityConfigurerAdapter，覆盖其默认配置 ~ 此处列出了一些常用的配置，大部分使用的都是默认值

```java
package org.springframework.security.samples.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.User.UserBuilder;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

	@Bean
	public BCryptPasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder(8);
	}

	@Bean
	public UserDetailsService userDetailsService() {
		PasswordEncoder encoder = this.passwordEncoder();

		// ensure the passwords are encoded properly
		UserBuilder builder = User.builder();
		builder.passwordEncoder(encoder::encode);

		InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
		manager.createUser(builder.username("user").password("123456").roles("USER").build());
		manager.createUser(builder.username("admin").password("123456").roles("USER", "ADMIN").build());

		return manager;
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// @formatter:off
		http.authorizeRequests()
				.antMatchers("/static/**", "/login", "/index").permitAll() // 允许匿名访问的地址
				.antMatchers("/admin/**").hasRole("ADMIN") // 仅允许 ADMIN 角色访问的地址
				.antMatchers("/user/**").hasRole("USER") // 仅允许 USER 角色访问的地址
			.and()
				.authorizeRequests().anyRequest().authenticated() // 其他页面都需要认证
			.and()
				.formLogin() // 启用表单登录
				.usernameParameter("username") // 表示登录时用户名使用的是哪个参数，默认为 “username”
				.passwordParameter("password") // 表示登录时密码使用的是哪个参数，默认为 “password”
				.loginProcessingUrl("/login") // 表示登录时表单提交的地址，默认为 “/login”
				.loginPage("/login") // 设置登录页面
				.failureUrl("/login?error") // 设置登录失败的页面，默认为 “/login?error”
				.defaultSuccessUrl("/index") // 设置登录成功后默认跳转的页面
			.and()
				.logout() // 设置登出
				.logoutUrl("/logout") // 设置登出的地址，默认为“/logout”
				.logoutSuccessUrl("/login?logout") // 设置登出成功后的跳转页面，默认为 “/login?logout”
			.and()
				.httpBasic(); // 启用 http basic 方式登录
		// @formatter:on
	}

}
```

5. 启动项目后，访问受限资源后跳转到登录页面，使用不同的用户后仅能访问授权的页面

[![](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210534-spring-security-demo.gif)](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210534-spring-security-demo.gif)

6. 默认登录页及其默认配置的代码解析

通过追踪函数 formLogin()，可以发现其返回表单配置类 FormLoginConfigurer

![HttpSecurity.formLogin()](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210628-paste-836ef2a3628b3a1a4c64f3c50022de6a-1.png)

在 FormLoginConfigurer 的构造函数中可以看到上面配置中涉及的 `usernameParameter` 和 `passwordParameter` 两个函数在这个构造函数中设置了默认值，也是通过这两个函数自定义了表单提交的参数名。其他，例如：loginProcessingUrl、loginPage、failureUrl、defaultSuccessUrl 等设置都是在父类 `AbstractAuthenticationFilterConfigurer` 中的函数。

此处的 `UsernamePasswordAuthenticationFilter` 就是用于 用户名和密码 进行登录认证的过滤器。

![FormLoginConfigurer](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210636-paste-853555a77378d48231e05759c4001404-1.png)

`FormLoginConfigurer` 中的函数 `initDefaultLoginFilter`，就是用于初始化默认登录页面的地方，`DefaultLoginPageGeneratingFilter` 类中包含了默认登录页面的 HTML 代码。此处可以看到，仅在未配置自定义登录页面时默认的登录页面才会起作用。

![FormLoginConfigurer.initDefaultLoginFilter](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410210644-paste-5d9e59673fbf2d838c71aa77588dbbee-1.png)


## 参考文档

[https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/)

