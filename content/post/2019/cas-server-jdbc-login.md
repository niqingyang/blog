---
title: "CAS 5.3.x 搭建和使用 – 使用 JDBC 单点登录"
date: 2019-02-25T00:28:49+08:00
draft: false
categories:
    - develop
tags:
    - cas
    - sso
    - jdbc
coverColor: "#4c5354f7"
coverImage: https://static.acme.top/wp-content/uploads/2019/02/cas-jdbc.png
---

<info>

> CAS 支持多种数据库身份验证，本文使用的是 MySql 进行的实践

</info>

### 一、创建数据库表

```sql
CREATE TABLE `acme_user` (
  `user_id` bigint(20) unsigned NOT NULL COMMENT '用户编号',
  `user_name` varchar(60) NOT NULL COMMENT '账户名称',
  `user_pass` varchar(255) NOT NULL COMMENT '账户密码',
  `user_email` varchar(100) NOT NULL DEFAULT '' COMMENT '账户邮箱',
  `expired` tinyint(1) DEFAULT '0' COMMENT '账户是否已过期',
  `disabled` tinyint(1) DEFAULT '0' COMMENT '账户是否被禁用',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

插入数据，密码采用 BCrypt 算法，明文为 <code>123456</code>

```sql
## 正常账户
insert into acme_user values ('1', 'acme', '$2a$08$jvlM7HkGKeec1lkS7JoQW.q8zfROC5DZtBk7ix/AZP687RTN5pGhO', 'acme-top@qq.com',  0, 0, 0);
## 过期账户
insert into acme_user values ('2', 'expired', '$2a$08$jvlM7HkGKeec1lkS7JoQW.q8zfROC5DZtBk7ix/AZP687RTN5pGhO', 'expired@qq.com',  1, 0, 0);
## 禁用账户
insert into acme_user values ('2', 'disabled', '$2a$08$jvlM7HkGKeec1lkS7JoQW.q8zfROC5DZtBk7ix/AZP687RTN5pGhO', 'disabled@qq.com', 0, 1, 0);
```

### 二、增加 JDBC 相关依赖

修改 <code>/cas/build.gradle</code> 文件，增加如下依赖

```groovy
compile "org.apereo.cas:cas-server-support-jdbc:${project.'cas.version'}"
compile "org.apereo.cas:cas-server-support-jdbc-authentication:${project.'cas.version'}"
compile "org.apereo.cas:cas-server-support-jdbc-drivers:${project.'cas.version'}"
// 我的 mysql 版本是 5.7
compile "mysql:mysql-connector-java:5.1.47"
```

<warning>

> mysql 的 jdbc 驱动包需要选择和 mysql 版本想对应的版本，否则会报错，对应关系可以参考文章：[https://blog.csdn.net/hchhan/article/details/81106992](https://blog.csdn.net/hchhan/article/details/81106992 "https://blog.csdn.net/hchhan/article/details/81106992")

</warning>

### 三、修改 application.properties 配置

1. 禁用默认的静态用户名密码

```properties
##
# CAS Authentication Credentials
#
# 当启用该配置时，为默认静态认证，登陆名为casuser密码为Mellon - 禁用此处配置
# cas.authn.accept.users=casuser::Mellon
```

2. 增加 jdbc 的相关配置

```properties
cas.authn.jdbc.query[0].sql=SELECT * FROM acme_user WHERE user_name=?
cas.authn.jdbc.query[0].fieldPassword=user_pass
cas.authn.jdbc.query[0].fieldExpired=expired
cas.authn.jdbc.query[0].fieldDisabled=disabled
cas.authn.jdbc.query[0].principalAttributeList=sn,cn:commonName,givenName

cas.authn.jdbc.query[0].user=root
cas.authn.jdbc.query[0].password=
cas.authn.jdbc.query[0].driverClass=com.mysql.jdbc.Driver
cas.authn.jdbc.query[0].url=jdbc:mysql://localhost:3306/acme_account?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false
cas.authn.jdbc.query[0].dialect=org.hibernate.dialect.MySQLDialect

# 配置密码，此处配置的加密算法为 BCRYPT
# NONE|DEFAULT|STANDARD|BCRYPT|SCRYPT|PBKDF2
cas.authn.jdbc.query[0].passwordEncoder.type=BCRYPT
cas.authn.jdbc.query[0].passwordEncoder.strength=8
```

CAS 支持的密码类型

<!--begin.mdui-table-fluid.mdui-table-hoverable-->

|  类型  |  描述  |
| ------------ | ------------ |
|  NONE  |  不进行密码编码（即纯文本）- 不推荐  |
|  DEFAULT  |  使用DefaultPasswordEncoderCAS。对于通过characterEncoding和的消息摘要算法encodingAlgorithm  |
|  BCRYPT  |  使用BCryptPasswordEncoder基于strength提供的和可选的secret  |
|  SCRYPT  |  使用SCryptPasswordEncoder  |
|  PBKDF2  |  使用Pbkdf2PasswordEncoder基于strength提供的和可选的secret  |
|  STANDARD  |  使用StandardPasswordEncoder基于secret提供的  |
|  org.example.MyEncoder  |  的实现PasswordEncoder你自己的选择  |

<!--end.mdui-table-fluid.mdui-table-hoverable-->

<info>

> **提示**  
> 1. jdbc 的配置类是 <code>org.apereo.cas.configuration.model.support.jdbc.JdbcAuthenticationProperties</code>
> 2. jdbc 数据查询验证的类是 <code>org.apereo.cas.adaptors.jdbc.QueryDatabaseAuthenticationHandler</code>
> 3. 可以继承 <code>org.springframework.security.crypto.password.PasswordEncoder</code> 来实现自定义密码验证

</info>

<info>

> **参考文档**  
> 1. jdbc 的配置可以参考官方文档 [CAS 5.3 - Database Authentication](https://apereo.github.io/cas/5.3.x/installation/Database-Authentication.html#database-authentication)
> 2. 密码相关配置可参考官方文档 [CAS 5.3 - Password Encoding](https://apereo.github.io/cas/5.3.x/installation/Configuration-Properties-Common.html#password-encoding)

</info>

### 四、测试访问

1. 正常账户 acme 登录，密码 123456，登录成功

![登录成功](https://static.acme.top/wp-content/uploads/2019/02/acme-login-cas.gif)

2. expired 账户登录失败

![账户已过期](https://static.acme.top/wp-content/uploads/2019/02/paste-264ec30667023a742625024d23ddccd2-1.png?w=676&h=119)

3. disabled 账户登录失败

![账户已被禁用](https://static.acme.top/wp-content/uploads/2019/02/paste-a51d884637424d9b5bba8876e029b656-1.png?w=702&h=124)
