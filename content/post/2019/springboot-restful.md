---
title: "SpringBoot RESTful"
date: 2019-03-20T21:11:58+08:00
draft: false
categories:
    - develop
tags:
    - springboot
    - java
    - restful
coverColor: "#575a56f7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410225144-springboot-restful.png
---

## 前言

Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新 Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot 致力于在蓬勃发展的快速应用开发领域 (rapid application development) 成为领导者。

Spring Boot 主要优点有：

1. 创建独立的Spring应用程序，可以以 jar 包的形式独立运行

2. 直接嵌入Tomcat，Jetty或Undertow（无需部署WAR文件）

3. 开箱即用，提供各种默认的 “starter” 依赖项以简化构建配置

4. 尽可能自动配置 Spring 和第三方库

5. 提供生产就绪功能，例如指标，运行状况检查和外部化配置

6. 绝对没有代码生成，也不需要XML配置

## 本章目的

使用 Spring Boot 来创建一个 RESTful 服务的 Hello World

实现的功能如下：

创建一个接受 `HTTP GET` 请求的服务

```shell
http://localhost:8080/greeting
```

使用 `JSON` 格式的响应结果作为问候的内容

```json
{"id":1,"content":"Hello, World!"}
```

支持可选参数 `name` 查询字符串自定义问候的内容

```shell
http://localhost:8080/greeting?name=XXX
```

`name` 参数会覆盖掉 `World` 默认值，并反馈在响应结果上

```json
{"id":1,"content":"Hello, XXX!"}
```

## 环境配置

IDE：STS4
JDK：1.8
Spring Boot：2.1.3
Gradle：4+

## 创建项目

通过 STS 创建一下新的 Spring Starter Project

![创建项目](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223402-paste-b876dd0c7e6b2ba4710df29329b4bad1-1.png)

... 构建过程可能会比较慢

## 项目目录结构

![目录结构](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223410-paste-14bca5ff20c21020edf647ebd59ef7e7-1.png)

目录及文件说明：

1. /src/main/java/  存放项目所有源代码目录
2. /src/main/resources/  存放项目所有资源文件以及配置文件目录
3. /src/main/resources/templates 是默认存放视图文件的目录
4. /src/main/resources/static 是默认存放视静态资源文件的目录
5. /src/test/  存放测试代码目录
6. Application 类为应用程序的入口，main 函数位于此类中，可以通过 run 运行
7. application.properties 是项目的核心配置文件

## 编写程序内容

### 创建模型类

构建一个问候语的模型类，其中：

`id` 是每次请求的以为标识
`content` 表示每次响应的问候内容


```java
package top.acme.example;

public class Greeting {

	private final long id;
	private final String content;

	public Greeting(long id, String content) {
		this.id = id;
		this.content = content;
	}

	public long getId() {
		return id;
	}

	public String getContent() {
		return content;
	}
}
```

<!--begin.tip-->

> Spring 会使用 Jackson JSON 库自动将 Greeting 的对象转换为 JSON

<!--end.tip-->

### 创建控制器

```java
package top.acme.example;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";

	private final AtomicLong counter = new AtomicLong();

	@RequestMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
```

分析一下这个看上去简洁而简单的控制器 ~ 其实里面不少东西

- Spring 构建的 RESTful Web 服务，其 HTTP 请求是由标识了 `@RestController` 注解的控制器来进行处理的。`GreetingController ` 的实例接受 `GET` 请求并由 `/greeting` 进行处理响应。

- `@RequestMapping` 注解可以确保 `HTTP` 请求 `/greeting` 被映射到 `greeting()` 方法上。

<!--begin.tip-->

> 上面的示例未指定 `GET`、`PUT`，`POST` 等等，因为 `@RequestMapping` 默认映射所有 `HTTP` 操作。使用 `@RequestMapping(method=GET)` 限定这种映射。

<!--end.tip-->

- `@RequestParam` 将查询字符串参数 `name` 的值绑定到方法 `greeting()` 的 `name` 参数中。如果 `name` 请求中不存在该参数，`defaultValue` 则使用 “World”。

- 传统 MVC 控制器和上面的 RESTful Web 服务控制器之间的关键区别在于创建 HTTP 响应主体的方式。这个 RESTful Web 服务控制器只是填充并返回一个对象，而不是依靠视图技术来执行服务器端的问候数据呈现 Greeting。对象数据将作为 JSON 直接写入 HTTP 响应。

- 此代码使用Spring 4的新 `@RestController` 注释，它将类标记为控制器，其中每个方法都返回一个域对象而不是视图。它是 `@Controller` 和 `@ResponseBody` 拼凑在一起的速记。

- 该 Greeting 对象必须转换为 JSON。由于 Spring 的 HTTP 消息转换器支持，开发人员无需手动执行此转换。因为 `Jackson 2` 在类的路径上，所以 `MappingJackson2HttpMessageConverter` 会自动选择 Spring 来将 Greeting 实例转换为 JSON。

## 生成可执行的应用程序

虽然可以将此服务打包为传统的 WAR 文件以部署到外部应用程序服务器，但 Spring Boot 提供了更简单的方法以创建了一个独立的应用程序。将所有内容打包在一个可执行的 JAR 文件中，由一个Java main() 方法驱动。在此过程中，Spring 支持将 Tomcat servlet 容器嵌入为 HTTP 运行时，而不是部署到外部实例。

```java
package top.acme.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

`@SpringBootApplication` 是一个方便注释，如果阅读源码会发现其包含了以下所有内容：

- `@Configuration` 注解该类作为应用程序上下文的 bean 定义的来源。

- `@EnableAutoConfiguration` 告诉 Spring Boot 开始根据类路径设置，其他 bean 和各种属性设置添加 bean。

- 一个Spring MVC应用程序通常会添加 `@EnableWebMvc` 注解，但 Spring Boot 会在类路径上看到 `spring-webmvc` 时`自动`添加它。这会将应用程序标记为 Web 应用程序并激活关键行为，例如设置 a DispatcherServlet。

- `@ComponentScan` 告诉 Spring 在包中寻找其他组件，配置和服务，允许它找到控制器。

该 `main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法启动应用程序。这期间没有一行XML，也没有 web.xml 文件。此 Web 应用程序是100％纯 Java，无需处理配置任何管道或基础结构。

## 构建可执行的 JAR

在项目的根目录下执行 `gradlew bootRun`。或者，在项目的根目录下执行 `gradlew build` 来构建可执行的 JAR 包，然后执行：

```shell
java -jar build/libs/spring-boot-rest-service-0.0.1.jar
```

随着显示记录输出，服务会在几秒内启动并运行

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223418-paste-9799ab6ba7c01295fcf89cd9e43bbee1-1.png)

## 测试

服务启动后，访问 `http//localhost8080/greeting`

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223426-paste-74f8ca915eb8ca1ad8b93f49a59341f0-1.png)

访问使用 `name` 提供查询字符串参数的 `http//localhost8080/greetingname=acme.top`, 会发现 `content` 属性的值从 “Hello，World！” 变为了 “hello, acme.top！”

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223436-paste-c0bfb714c7ba2201064683fbb45d061f-1.png)

说明 `GreetingController` 上的 `@RequestParam` 起作用了。`name` 参数被赋予默认值 “World”，然后可以通过查询字符串显式覆盖。

另外 `id` 属性从更改1为2。说明在 `GreetingController` 上跨多个请求针对同一实例工作，并且其 `counter` 字段在每次调用时按预期递增。

## 结尾

使用Spring开发了RESTful Web服务到这里就结束了。
