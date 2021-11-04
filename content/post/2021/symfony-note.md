---
title: "Symfony 笔记"
date: 2021-04-18T22:49:01+08:00
draft: true
categories:
  - develop
tags: 
  - php
  - symfony
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
themeColor: "#343432"
## 封面颜色
coverColor: "rgba(27,27,27,0.97)"
## 封面图片
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/11/20211104000121-symfony.png
## 文章页封面图分割：0-默认小图模式 1-大图模式
# coverStyle: 0
---

## 安装CLI

```bash
# 下载并安装 Symfony CLI - https://symfony.com/download
curl -sS https://get.symfony.com/cli/installer | bash
```

## 创建应用

- 构建传统的 web 应用程序

```bash
symfony new --full my_project
```

- 构建微服务、控制台程序或者 API

```bash
symfony new my_project
```

## 安装组件



```bash
# 安装 Doctrine ORM
symfony composer req "orm:^2"
# 用了管理注解
symfony composer req annotations
# 验证器组件
symfony composer req validator
# 字符串组件 - https://symfony.com/doc/current/components/string.html
symfony composer req string
# 安装日志记录器
symfony composer req logger
# 安全组件
symfony composer req security
# 安装 twig 的支持
symfony composer req twig
# 安装开发环境下的分析器
symfony composer req profiler --dev
# 安装开发环境下的调试工具（页面底部的调试工具栏）
symfony composer req debug --dev
# 使用 Maker Bundle 可以在开发时很多类文件
symfony composer req maker --dev
# 插件包
symfony composer req maker-bundle
```

## 启动服务

```bash
# 进行项目目录启动服务
symfony serve
```

## 一些命令

```bash
# 一个可以很方便查看最新的日志的命令（来自 web 服务器、PHP 和应用程序的日志）
symfony server:log
# Symfony Console 自带 list 命令，它会列出某个命名空间下的所有可用命令
# 例如用它来查看 maker bundle 提供的所有生成器
symfony console list make
# 创建数据迁移
symfony console make:migration
# 迁移数据库
symfony console doctrine:migrations:migrate
# 展示出当前的所有路由
symfony console debug:router
```

## 创建控制器

```bash
# 创建控制器
symfony console make:controller IndexController
# 创建实体类
symfony console make:entity User
# 创建订阅器 - https://symfony.com/doc/current/the-fast-track/zh_CN/12-event.html
symfony console make:subscriber TwigEventSubscriber
```

## 数据迁移

```bash
# 创建数据库迁移
php bin/console make:migration
# 执行数据库迁移
php bin/console doctrine:migrations:migrate
# 更新数据库模式
php bin/console doctrine:schema:update
```

