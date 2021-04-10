---
title: "Eclipse常用项目属性"
date: 2018-09-15T22:09:22+08:00
draft: false
tags:
    - eclipse
    - java
categories:
    - develop
coverColor: "#444444f7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224416-eclipse-01.png
---

<info>

> 将Java项目导入到Eclipse中后，有时候会出现一些莫名其妙的语法错误，或者编译不通过，那么可能是项目设置的问题，可以右键项目打开菜单`properties`检查相关设置是否正确

</info>

## 检查 Project Facets

Facets英文意思为方面。
在 Eclipse 中的 facets 可以理解为：项目的特性，某一方面功能。主流 IDE (Eclipse、IDEA) 都提供了 facet 的配置。

Eclipse 中 Project Facets 配置界面如下图：

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204917-ac228de3f724355d8edc474ce8250a32.png)

- 如果项目是一个最简单Java Project，那么应该只会有Java被勾选；
- 如果项目是一个Java Web Project，那么 Dynamic Web Module 也会被勾选，同时 JavaScript 也会被勾选，Java Web 项目在目录结构上会多出 WebContent 或者 WebRoot 目录，并在项目属性中多出 Deployment Assembly、Web Content Settings、Web Page Editor、Web Project Settings 等只有Web项目才需要的选项；

<!--begin.tip-->

> 此选项下需要检查 Java对应的版本是否正确，很多时候默认选中的是 “1.5”，也就是按 JDK1.5版本编译，这里建议选择与你所需运行环境一致的JDK版本号。
> 如果你的项目是 Web项目，那么要就要确认一下 Dynamic Web Module 是否勾选，勾选后项目将编程一下 Java Web Project。

<!--end.tip-->

## 检查 Java Compiler

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204926-93108bcee5464efd20c003d6e91a533a.png)

<!--begin.tip-->

> 此选项下也是需要检查 Java对应的版本是否正确，这里建议选择与你所需运行环境一致的JDK版本号，和 Project Facets 中 Java 的版本一致即可。

<!--end.tip-->

## 检查 Java Build Path

如果导入的项目在左侧项目 Package Exploer 中看不到项目中正常的包结构，那么肯定缺少指定哪些目录为代码源文件、哪些目录为资源文件、哪些目录是放测试代码和资源的设置。

重新选择并设置一下源码目录即可：

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204935-b7f1d2000c4ece8ea854dd3df2f656f6.png)

<!--begin.tip-->

> 在选项卡 Source 中设置相关源码的构建目录。
> 在选项卡 Libraries 可以添加 Add-Library > JRE System Library、 Server Runtime（Web项目可能需要）、Maven Managed Dependencies（如果你的 Maven 项目中不显示所依赖的 Maven 配置的 Jar 包则需要添加此项）。

<!--end.tip-->

## Deployment Assembly

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205216-032d9cba752df7feb241028be4e4a32e.png)

<!--begin.tip-->

> 这个选项用于 Java Web Project。
> 可配置项目中的代码和配置文件在发布到例如 Tomcat 上时目录文件的映射关系。

<!--end.tip-->

## Web Project Setttings

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205223-c471b0c3793f9cd023c1678d2ed637fa.png)

<!--begin.tip-->

> 配置中 `Context root` 设置为 `demo` 则启动 server 后，则所有请求路径都需要以 `/demo/` 开始；
> 如果想以 `/` 开头，则设置为 `/` 即可。

<!--end.tip-->

</info>