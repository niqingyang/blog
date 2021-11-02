---
title: "Mac OS 下安装 PHP XDebug"
date: 2021-04-26T00:17:58+08:00
draft: true
categories:
  - develop
tags:
  - php
  - xdebug
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
# themeColor: 
## 封面颜色
# coverColor: "rgba(0,0,0,0.3)"
## 封面图片
# coverImage: 
## 文章页封面图分割：0-默认小图模式 1-大图模式
# coverStyle: 0
---



<info>

> Xdebug 是 PHP 的调试工具。它提供步调调试和一系列开发辅助工具，如堆栈跟踪、代码分析器、将脚本的完整执行转储到文件的功能等。通过开启 Xdebug 的自动跟踪（ auto_trace ）和分析器功能，更可以直观察PHP源代码的性能数据，从而优化PHP代码。

</info>

## 要求

Xdebug需要一个[支持版本](https://www.php.net/supported-versions.php)的PHP。安装时，它需要 pecl 工具（通过 php-pear 包提供），除非您的 Linux 分布有 Xdebug 包 （php-xdebug）。

## 安装

在大多数Linux分布中，可以通过其包管理器安装 Xdebug。还可以通过 pecl 安装 xdbug。后者也适用于 MacOS，只要 PHP 安装与自制。

在 Windows 上，可以[下载](https://xdebug.org/download#releases)二进制文件。

除非已经在 Linux 上安装了带有包管理器的 Xdebug，否则还需要将以下行添加到您的 php .ini 文件中，或在 conf.d 目录中创建新的 Xdebug 特定的 ini 文件 xdebug .ini。在这两种情况下，它都需要添加以下行：

```bash
zend_extension=xdebug
```

有关更广泛的安装说明，请参阅https://xdebug.org/docs/install文档



使用 pecl 安装 xdebug

```bash
pecl install xdebug
```

安装完成后会有类似如下提示，其中包括了 xdebug 的安装地址

```bash
Build process completed successfully
Installing '/usr/local/Cellar/php@7.4/7.4.16/pecl/20190902/xdebug.so'
install ok: channel://pecl.php.net/xdebug-3.0.4
Extension xdebug enabled in php.ini
```

## 配置

Xdebug 中的大多数功能必须选择进入。每个功能都有一个特定的选择加入。例如，要使用[步骤调试器](https://xdebug.org/docs/remote)，您需要在配置文件中设置xdebug.remote_enable=1。步骤调试器需要IDE（客户端），其中有许多[可用](https://xdebug.org/docs/remote#clients)。

文档包含针对 Xdebug 的每个功能的说明[：https://xdebug.org/docs/，](https://xdebug.org/docs/)那里还提供完整的[设置](https://xdebug.org/docs/all_settings)列表。



- xdebug 2

```ini
[xdebug]
zend_extension="/usr/local/Cellar/php@7.4/7.4.16/pecl/20190902/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_host=127.0.0.1
xdebug.remote_port="9000"
xdebug.idekey=PHPSTORM
```

- Xdebug 3

```bash
[xdebug]
zend_extension="/usr/local/Cellar/php@7.4/7.4.16/pecl/20190902/xdebug.so"
xdebug.mode=debug
xdebug.client_host=127.0.0.1
xdebug.client_port="9003"
xdebug.idekey=PHPSTORM
```



<info>

> Mac 下 PHP 的安装目录位于 `/usr/local/Cellar/php@7.4`，`php.ini` 的目录位于 `/user/local/etc/php/7.4`

</info>   

## 原理

![Xdebug 调试原理](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210426223856-5394123-d3a83c3dcaddf614.gif)

## 参考

- https://www.cnblogs.com/ashaff/p/11609343.html

