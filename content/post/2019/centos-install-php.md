---
title: "CentOS 安装 PHP 7.3"
date: 2019-04-23T00:26:12+08:00
draft: false
categories:
    - ops
tags:
    - php
    - centos
    - linux
coverColor: "#343434f7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224800-centos-install-php.png
---

## 简介

PHP，即“PHP: Hypertext Preprocessor”，是一种被广泛应用的开源通用脚本语言，尤其适用于 Web 开发并可嵌入 HTML 中去。它的语法利用了 C、Java 和 Perl，易于学习。其主要目标是允许 web 开发人员快速编写动态生成的 web 页面，但 PHP 的用途远不只于此。

## 环境说明

- CentOS 7.6 64位 操作系统下
- 本文档包括使用 PHP-FPM 为 Nginx 1.4.x HTTP 服务器安装和配置 PHP 的说明和提示
- 本指南假定您已经从源代码成功构建 Nginx，并且其二进制文件和配置文件都位于 /usr/local/nginx。 如果您使用其他方式获取的 Nginx，请参考 Nginx Wiki >> [安装](http://wiki.nginx.org/Install "安装") 并对照本文档完成安装。
- 本文档仅包含 Nginx 服务器的基本配置，它将通过 80 端口提供 PHP 应用的处理能力。 如果您需要超出本文档范围的安装配置指导，建议您查阅 Nginx 和 PHP-FPM 的文档。

## 安装依赖库

```shell
yum install -y gcc gcc-c++ autoconf make automake wget vim httpd-tools unzip zip libxml2-devel bzip2-devel curl-devel libjpeg-devel libpng-devel freetype-devel libxslt-devel libzip-devel
```

## 创建用户、用户组

```shell
groupadd www
useradd -s /sbin/nologin -g www www
```

## 获取并解压源码

从[官网](https://www.php.net/ "官网")获取 PHP 源代码

```shell
# 进入src目录
cd /usr/local/src
# 下载源码包
wget https://www.php.net/distributions/php-7.3.4.tar.gz
# 解压包文件
tar -zxvf php-7.3.4.tar.gz
# 进入刚解压的包目录
cd php-7.3.4
```

## 配置并构建

在此步骤可以使用很多选项自定义 PHP，例如启用某些扩展等。 运行 `./configure --help` 命令来获得完整的可用选项清单。 在本示例中，仅进行包含 PHP-FPM 和 MySQL 支持的简单配置。

```shell
./configure \\
--prefix=/usr/local/php \\
--with-config-file-path=/usr/local/php/bin \\
--with-mysqli=mysqlnd \\
--with-pdo-mysql=mysqlnd \\
--enable-fpm \\
--enable-static \\
--enable-maintainer-zts \\
--enable-inline-optimization \\
--enable-sockets \\
--enable-wddx \\
--enable-zip \\
--enable-calendar \\
--enable-bcmath \\
--enable-soap \\
--with-zlib \\
--with-iconv \\
--without-iconv \\
--with-gd \\
--with-xmlrpc \\
--enable-mbstring \\
--with-curl \\
--enable-ftp \\
--with-freetype-dir \\
--with-jpeg-dir \\
--with-png-dir \\
--with-xsl \\
--with-gettext \\
--with-pear \\
--enable-sockets \\
--enable-exif \\
--disable-rpath \\
--disable-ipv6 \\
--disable-debug \\
--with-openssl
//检测安装环境，指定安装目录，加载gd，zlib等模块
# make
//编译
# make install
```

<info>

> **核心配置选项**
> PHP 的 configure 脚本使用的部分选项的列表，可以参考 [官方文档](https://www.php.net/manual/zh/configure.about.php "官方文档")、[PHP 编译参数详解](https://www.cnblogs.com/hubing/p/3735452.html "PHP 编译参数详解")

</info>

## 常见的编译错误

configure: error: libxml2 not found. Please check your libxml2 installation

```shell
yum install -y libxml2-devel
```

configure: error: Please reinstall the BZip2 distribution

```shell
yum install -y bzip2-devel
```

configure: error: cURL version 7.15.5 or later is required to compile php with cURL support

```shell
yum install -y curl-devel
```

configure: error: jpeglib.h not found.
```shell
yum install -y libjpeg-devel
```

configure: error: png.h not found.

```shell
yum install -y libpng-devel
```

configure: error: freetype-config not found.

```shell
yum install -y freetype-devel
```

configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution

```shell
yum install -y libxslt-devel
```

configure: error: Please reinstall the libzip distribution

```shell
yum install -y libzip-devel
```

configure: WARNING: unrecognized options: --with-mysql

```shell
# php7 不支持 mysql 模块，改用--with-pdo-mysql
```

configure: WARNING: unrecognized options: --with-mcrypt

```shell
# php7.3 不支持 --with-mcrypt 模块，去掉即可
```

configure: error: off_t undefined; check your library configuration

```shell
# off_t 类型是在 头文件 unistd.h中定义的，
# 在32位系统 编程成 long int ，64位系统则编译成 long long int ，
# 在进行编译的时候 是默认查找64位的动态链接库，
# 但是默认情况下 centos 的动态链接库配置文件/etc/ld.so.conf里并没有加入搜索路径，
# 这个时候需要将 /usr/local/lib64 /usr/lib64 这些针对64位的库文件路径加进去。
vim /etc/ld.so.conf
# 添加如下几行
/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64
# 保存退出
:wq
# 使之生效
ldconfig -v
```

/usr/local/include/zip.h:59:21: fatal error: zipconf.h: No such file or directory

```shell
# 解决方法：手动复制过去
cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h
```

## 添加到环境变量中

```shell
ln -s /usr/local/php/bin/php /usr/local/sbin/php
ln -s /usr/local/php/sbin/php-fpm /usr/local/sbin/php-fpm
```

## 配置 PHP

1. 复制配置文件

```shell
# 复制 php.ini-developmen 或者 php.ini-production 到 /usr/local/php/bin/php.ini 下变成php.ini配置文件
cp php.ini-development /usr/local/php/bin/php.ini
## 复制 php-fpm.conf
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
```

2. 将 php.ini 文件中的配置项 cgi.fix_pathinfo 设置为 0 。

```shell
# 打开 php.ini:
vim /usr/local/php/bin/php.ini
# 输入“\\”搜索并定位到 cgi.fix_pathinfo= 并将其修改为如下所示
cgi.fix_pathinfo=0
```

3. 编辑 /usr/local/php/etc/php-fpm.d/www.conf ，找到以下配置项， 配置如下

```shell
user = www
group = www
```

4. 启动 php-fpm

```shell
/usr/local/php/sbin/php-fpm
```

## 配置 Nginx

1. 编辑配置文件

```shell
vim /usr/local/nginx/conf/nginx.conf
```

2. 修改默认的 location 块，使其支持 .php 文件：

```nginx
location / {
    root   html;
    index  index.php index.html index.htm;
}
```

3. 配置对 .php 文件的请求将被传送到后端的 PHP-FPM 模块， 取消默认的 PHP 配置块的注释，并修改为下面的内容：

```nginx
location ~ \.php$ {
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    fastcgi_param  SCRIPT_NAME      $fastcgi_script_name;
    include        fastcgi_params;
}
```

4. 重启 Nginx

```shell
nginx -s reload
```

5. 创建测试文件

```shell
echo "<?php phpinfo(); ?>" >> /usr/local/nginx/html/index.php
```

6. 打开浏览器，访问 http://localhost，将会显示 phpinfo()

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205600-paste-d346f556257b7b798e86c5c9fd0689d0-1.png)

通过以上步骤的配置，Nginx 服务器现在可以以 SAPI SAPI 模块的方式支持 PHP 应用了

## 参考文档

[https://www.php.net/manual/zh/install.unix.nginx.php](https://www.php.net/manual/zh/install.unix.nginx.php "https://www.php.net/manual/zh/install.unix.nginx.php")

[https://www.php.net/manual/zh/configure.about.php](https://www.php.net/manual/zh/configure.about.php "https://www.php.net/manual/zh/configure.about.php")

[https://segmentfault.com/a/1190000017570008](https://segmentfault.com/a/1190000017570008 "https://segmentfault.com/a/1190000017570008")