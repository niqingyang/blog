---
title: "CAS 5.3.x 搭建和使用 – SpringBoot内部运行"
date: 2019-02-17T11:54:59+08:00
draft: false
categories:
    - develop
tags:
    - cas
    - java
    - springboot
    - sso
coverColor: "#6cbc44f7"
coverImage: https://static.acme.top/wp-content/uploads/2019/02/sso.png
---

<info>

> 使用 cas-gradle-overlay-template 搭建单点登录服务端

</info>

### 前言

一直打算利用 SpringBoot + CAS 简单的实现一下单点登录的功能，中间耽搁了好长时间，最近算是有点眉目了，趁着还能想起中间踩过的坑，索性写点东西，把过程记录一下，方便以后再次上手。

### 一、CAS 简介

CAS是Central Authentication Service的缩写，中央认证服务，一种独立开放指令协议。CAS 是 Yale 大学发起的一个开源项目，旨在为 Web 应用系统提供一种可靠的单点登录方法，CAS 在 2004 年 12 月正式成为 JA-SIG 的一个项目，现在CAS已被apereo社区托管。

github地址为：[https://github.com/apereo/cas](https://github.com/apereo/cas "https://github.com/apereo/cas")

### 二、CAS 原理

如果对 SSO 和 CAS 不是特别熟悉，可以看看掘金上的这篇文章 [《前端需要了解的 SSO 与 CAS 知识》](https://juejin.im/post/5a002b536fb9a045132a1727 "《前端需要了解的 SSO 与 CAS 知识》")，介绍的比较详细。

### 三、环境说明

```properties
操作系统：windows10
Java 版本：1.8
Gradle 版本：4.8
项目目录：安装在 E 盘
CAS 服务域名：cas.example.org
CAS 服务端口：8443
```

### 四、下载和构建

#### 1、部署说明

CAS 由服务端和客户端构成，服务端是一个 WEB 工程，可以将构建的 war 包放在 Web 容器中运行，比如 tomcat，也可以以 springboot 的方式启动。

CAS 的服务端搭建有两种常用的方式：

1. 基于源码的基础上构建出来的
2. 使用 WAR overlay 的方式来安装

官方推荐使用第二种，配置管理方便，以后升级也容易，具体可参考 [官方文档](https://apereo.github.io/cas/5.3.x/installation/Maven-Overlay-Installation.html "官方文档")。本文选择使用第二种方式。

CAS Overlay 是官方提供的一个模板，使用者可以在这个模板的基础上进行覆盖式的修改，而无需修改源码，这样的好处显而易见，方便后期的升级维护。具体的覆盖方法后面会介绍。

#### 2、下载 cas-gradle-overlay-template

CAS Overlay 有 gradle 和 maven 两个版本的，本文使用的是 gradle 版本

```shell
git clone -b 5.2 https://github.com/apereo/cas-overlay-template.git
```

#### 3、构建

进入 cas-overlay-template 目录下执行

```shell
gradlew build
```

### 五、生成证书

CAS 5 默认需要HTTPS，HTTP不能访问，所以需要配置证书路径，如果没有进行证书的配置启动会报错：Caused by: java.io.FileNotFoundException: \\etc\\cas\\thekeystore (系统找不到指定的文件。)

![](https://static.acme.top/wp-content/uploads/2019/02/paste-cb4f609674c58255414cf3dad4c8684b-1.png?w=859&h=452)

解决办法有以下两种：利用 JDK 的 keytool 工具生成证书，或者用 CAS Overlay 生成证书。

#### 1. 利用 JDK 的 keytool 工具生成证书（路径需要根据实际情况变更）

```shell
# 注：%JAVA_HOME%\\jre\\lib\\security 目录下的 cacerts
keytool -list -keystore cacerts -alias cas

# 生成证书，此处CN为前面设定的host的域名
keytool -genkey -alias cas -keyalg RSA -keysize 2048 -keypass changeit -storepass changeit -keystore E:\\etc\\cas\\keystore -dname "CN=cas.example.org,OU=Example,O=Org,L=Beijing,ST=BeiJing,C=US"
# 导出证书
keytool -exportcert -alias cas -keystore E:\\etc\\cas\\keystore -file E:\\etc\\cas\\cas.cer -storepass changeit
# 导入证书到java cacerts中（这个步骤一定要做，否则后面客户端请求CAS服务时会报证书错误的异常）
keytool -import -trustcacerts -alias cas -keystore .\\cacerts -file E:\\etc\\cas\\cas.cer -storepass changeit
# 删除证书
keytool -delete -alias cas -keystore .\\cacerts

-alias       产生别名
-keystore    指定密钥库的名称(产生的各类信息将不在.keystore文件中)
-keyalg      指定密钥的算法 (如 RSA  DSA（如果不指定默认采用DSA）)
-validity    指定创建的证书有效期多少天
-keysize     指定密钥长度
-storepass   指定密钥库的密码(获取keystore信息所需的密码)
-keypass     指定别名条目的密码(私钥的密码)
-dname       指定证书拥有者信息 例如：  "CN=名字与姓氏,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=州或省份名称,C=单位的两字母国家代码"
-list        显示密钥库中的证书信息      
-v           显示密钥库中的证书详细信息
-export      将别名指定的证书导出到文件  
-file        参数指定导出到文件的文件名
-delete      删除密钥库中某条目         
-printcert   查看导出的证书信息          
-keypasswd   修改密钥库中指定条目口令   
-storepasswd 修改keystore口令     
-import      将已签名数字证书导入密钥库
```

#### 2. 利用 CAS Overlay 生成证书

修改 cas-overlay-template 目录下的 build.sh 文件，修改下图中标红的地方，将相关域名、IP、DNS等按自己的配置修改好。DNS 要和 CAS 服务的域名一致，否则会导致客户端调用服务时出现错误：

![build.sh中修改生成证书的配置](https://static.acme.top/wp-content/uploads/2019/02/paste-aae4dff165ef48e8784ecd1473cbe7e5-1.png?w=1146&h=288)

修改不正确可能会出现的额外错误：

```java
// 原因是服务器端创建证书的时候dname和SAN中的域名不匹配。
javax.net.ssl.SSLHandshakeException: java.security.cert.CertificateException: No subject alternative DNS name matching cas.example.org found.
```

修改后<code>以管理员的身份</code> 打开命令行窗口执行

```shell
## build.sh 文件的 gencert() 函数中，定义了要生成的证书的相关参数（例如：域名、密码等），可以自行修改
build.sh gencert
```

默认会在 <code>E:/etc/cas/</code> 目录下生成 cas.cer 和 thekeystore 两个文件，如果是使用的 git bash 执行的，会在 <code>Git安装目录/etc/cas/</code> 目录下生成这两个文件，需要将整个 <code>cas</code> 自行复制到 <code>E:/etc/</code> 目录下。

### 六、修改配置文件

修改上一步骤中的文件 <code>E:/etc/cas/config/cas.properties</code>，次配置可以自定义 CAS 服务的端口号和服务端域名，我这里直接使用了默认配置，保证 CAS 服务端域名与证书中的域名一致

```properties
# server.port = 8443
cas.server.name: http://cas.example.org:8443
cas.server.prefix: http://cas.example.org:8443/cas

cas.adminPagesSecurity.ip=127\.0\.0\.1

logging.config: file:/etc/cas/config/log4j2.xml
# cas.serviceRegistry.config.location: classpath:/services

# SSL
# server.ssl.enabled=false
```

### 七、运行

进入 cas-overlay-template 目录下执行

```shell
// 或者 build.sh bootrun
gradlew run
```

出现如下图中的 “READY” 就代表启动成功了

![cas server 启动成功的信息](https://static.acme.top/wp-content/uploads/2019/02/paste-90a5260177020ca42babde6b9b70895c-1.png?w=859&h=452)

#### 八、访问并登录

访问 第六步 中的服务链接地址 <code>http://cas.example.org:8443/cas</code> 会出现如下图

![登录页面](https://static.acme.top/wp-content/uploads/2019/02/paste-f9bb8b769f41424e11e7d4be0d2ccd3e-1.png?w=1120&h=631)

输入默认的用户名/密码：casuser/Mellon，登录成功

![登录成功](https://static.acme.top/wp-content/uploads/2019/02/paste-a75e752a329c5a98e25528973656dba8-1.png?w=1133&h=454)


### 九、自定义相关配置

待续...
