---
title: "CAS 5.3.x 搭建和使用 – 修改默认的静态用户名和密码"
date: 2019-02-24T17:25:27+08:00
draft: false
categories:
    - develop
tags:
    - cas
    - sso
coverColor: "#f48313f7"
coverImage: https://static.acme.top/wp-content/uploads/2019/02/sso-user.png
---

<info>

> CAS 默认的用户名/密码是 casuser/Mellon，默认配置肯定是无法实际项目的需求的，一般都需要一定的二次开发，本文将通过修改默认的用户名、密码来说明如何自定义配置。

</info>

## 使用 WAR Overlay 方式进行覆盖式修改安装

参考文档：[https://apereo.github.io/cas/5.3.x/installation/Maven-Overlay-Installation.html](https://apereo.github.io/cas/5.3.x/installation/Maven-Overlay-Installation.html "https://apereo.github.io/cas/5.3.x/installation/Maven-Overlay-Installation.html")

## 将项目 cas-gradle-overlay-template 导入 Eclipse

1. 在项目目录下执行命令，生成 eclipse 项目相关配置文件

```shell
gradlew eclipse
```

2. 导入项目到 eclipse 中

![cas项目目录结构](https://static.acme.top/wp-content/uploads/2019/02/paste-0f8e40835588d94e219bb860a0ac92d1-1.png?w=457&h=401)

<info>

> 覆盖 CAS 的配置主要放在 <code>/cas/src/main/reources/</code> 目录下，自定义的代码放在 <code>/cas/src/main/java/</code> 目录下

</info>

## 修改默认的用户名、密码

1. 将构建好的默认配置 <code>cas/build/libs/cas.war!WEB-INF/classes/application.properties</code> 复制到 <code>/cas/src/main/reources/</code> 下

2. 修改 <code>application.properties</code> 中的 <code>cas.authn.accept.users</code>

```properties
# 默认配置
cas.authn.accept.users=casuser::Mellon
# 修改后的
cas.authn.accept.users=用户名::密码
```

3. 重新构建并运行

```shell
gradlew build

gradlew run
```

4. 用修改后的用户名/密码即可登录成功！

