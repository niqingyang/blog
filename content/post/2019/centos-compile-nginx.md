---
title: "CentOS7 下源码编译 Nginx1.14"
date: 2019-04-14T00:12:43+08:00
draft: false
categories:
    - ops
tags:
    - nginx
    - centos
    - linux
coverColor: "#13943df7"
coverImage: https://static.acme.top/wp-content/uploads/2019/03/install-nginx-cool.png
---

## 安装环境

环境使用的是 Win10 Linux 子系统的 CentOS 版，类似 Docker，重新初始化的一个新环境里面缺少好多东西，需要一个个都安装

nginx 以及所有的第三方包的源代码都统一放在 `/usr/local/src` 目录下

## 版本选择

官网的 nginx 有以下三个版本：

1. Mainline version - 主线版，该版本包含最新的特性和bug修改，并且总是保持更新。该版本是可靠的，但它可能会包含实验性的模块，以及一定数量的新 bug。

2. Stable version - 稳定版，该版本不包含新特性，但包含关键 bug 修复。**推荐**使用该版用于生产环境。

3. Legacy versions - 旧的稳定版本。

本文选择 Stable version 进行编译安装

## 安装编译工具

在编译安装 nginx 前，需要检查并安装一些必要的工具

```shell
yum -y install gcc gcc-c++ autoconf make automake wget vim httpd-tools unzip zip bzip2
```

## 安装依赖库

### 安装依赖库

```shell
yum install -y pcre-devel zlib-devel openssl-devel libxml2 libxml2-devel libxslt-devel gd-devel perl-devel perl-ExtUtils-Embed GeoIP GeoIP-devel GeoIP-data git
```

<!--begin.tip-->

> linux 下 devel 的作用
> devel 包主要是供开发用，至少包含 `头文件` 和 `链接库`，有的还含有开发文档或演示代码。
> 以 zlib 和 zlib-devel 为例：
> 如果你安装基于 zlib 开发的程序，只需要安装 zlib 包就行了。
> 但是如果你要编译使用了 zlib 的源代码，则需要安装 zlib-devel。
> 所以编译 nginx 时使用的相关依赖库，一般都需要安装相关库的 devel 包。

<!--end.tip-->

### 安装 OpenSSL 库

[OpenSSL](https://www.openssl.org/ "OpenSSL") 库用于支持 HTTPS 协议，nginx 的 SSL 模块依赖该库：

```shell
wget https://www.openssl.org/source/openssl-1.1.1b.tar.gz
tar -zxvf openssl-1.1.1b.tar.gz

# 重命名
mv openssl-1.1.1b openssl

cd openssl

./config -fPIC --prefix=/usr/local/openssl enable-shared
./config -t
make && make install
# 写入环境变量，以便可以直接使用 openssl 命令
# 如果之前已经存在老版本的 /usr/bin/openssl 软连接，可以通过 rm -rf 命令进行删除
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
ldconfig -v
# 查看版本
openssl version
```

<!--begin.warning-->

> **注意**
> - config 的时候，需要加上 `-fPIC` 和 `-t`，部分环境依赖一些头文件和 lib（比如 ruby）。
> - openssl 编译时间较长...
> - 通过 `/usr/local/openssl/bin/openssl version` 可以查看版本
> - 如果报错缺少 `libcrypto.so.1.1` 或者 `libssl.so.1.1`，可以到安装目录下的 `/lib/` 目录中将相关文件 `cp` 到 `/usr/lib64` 目录下即可

<!--end.warning-->

### 安装 jemalloc 库

[jemalloc](http://jemalloc.net/ "jemalloc") 库是用于内存优化的管理程序

```shell
# 建议到Github上下载最新版本，本文以5.1.0为例
wget https://github.com/jemalloc/jemalloc/releases/download/5.1.0/jemalloc-5.1.0.tar.bz2
tar -jxvf jemalloc-5.1.0.tar.bz2
cd jemalloc-5.1.0
./configure
make && make install
# 生产动态链接库到 /usr/local/lib/libjemalloc.so
ldconfig
cd ..
rm -rf jemalloc*
```

### 安装 PCRE
PCRE 是一个轻量级的 Perl 函数库，包括 perl 兼容的正则表达式库，可以从 [PCRE官网](http://www.pcre.org/ "PCRE官网")下载源码包 ~ 至少目前这个版本的 nginx 还不支持 pcre-10，需要下载 8 版本的...

```shell
# 下载源码包
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
# 解压
tar -zxvf pcre-8.38.tar.gz
# 重命名
mv pcre-8.38 pcre

cd pcre
# 构建
./configure
make && make install
```

### 安装 Lua

[Lua](http://www.lua.org/ "Lua") 一般 Linux 下都自带了 lua，但是版本可能比较低，可以通过 `lua -v` 查看，建议升级到最新版的

```shell
wget http://www.lua.org/ftp/lua-5.3.5.tar.gz

tar -zxvf lua-5.3.5.tar.gz

# 重命名
mv lua-5.3.5 lua

make linux
```

### 安装 LuaJIT

[LuaJIT](http://luajit.org/ "LuaJIT") 安装 lua-nginx-module-0.10.14 模块依赖 LuaJIT，但如果安装官方的版本在启动 nginx 的时候会提示如下图：

![](https://static.acme.top/wp-content/uploads/2019/03/paste-d6b7bc7a96a1c8267b12259efbdeb8b5-1.png?w=785&h=473)

lua-nginx-module 的作者单独 fork 了一个分支自己维护，如果不用 https://github.com/openresty/luajit2 这个，那么需要参数都将失效

安装 openresty 官方的 LUAJIT

```shell
yum install -y lua-devel libtermcap-devel ncurses-devel libevent-devel readline-devel

wget -O luajit2-2.1-20190302.tar.gz  https://github.com/openresty/luajit2/archive/v2.1-20190302.tar.gz
tar -zxvf luajit2-2.1-20190302.tar.gz

# 重命名
mv luajit2-2.1-20190302 luajit

cd luajit
make install

# 设置环境变量 ~ 必须在同一个终端下执行，否则找不到 Lib 库
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.1

# 链接库
echo "/usr/local/lib" >> /etc/ld.so.conf
ldconfig
```

## 安装第三方模块

### ngx_devel_kit

[ngx_devel_kit](https://github.com/simplresty/ngx_devel_kit "ngx_devel_kit")

```shell
wget -O ngx_devel_kit-0.3.0.tar.gz https://github.com/simplresty/ngx_devel_kit/archive/v0.3.0.tar.gz
tar -zxvf ngx_devel_kit-0.3.0.tar.gz

# 重命名
mv ngx_devel_kit-0.3.0 ngx_devel_kit
```

### lua-nginx-module

```shell
wget -O lua-nginx-module-0.10.13.tar.gz https://github.com/openresty/lua-nginx-module/archive/v0.10.13.tar.gz
tar -zxvf lua-nginx-module-0.10.13.tar.gz

# 重命名
mv lua-nginx-module-0.10.13 lua-nginx-module
```

## 下载源码包

[nginx 官网](http://nginx.org/en/download.html "nginx 官网") 下载源码包。

```shell
wget http://nginx.org/download/nginx-1.14.2.tar.gz
tar -zxvf nginx-1.14.2.tar.gz
cd nginx-1.14.2
```

## 创建用户、用户组

```shell
groupadd www
useradd -s /sbin/nologin -g www www
```


## 编译配置

```shell
./configure \\
--user=www \\
--group=www \\
--prefix=/usr/local/nginx \\
--with-openssl=/usr/local/src/openssl \\
--with-pcre=/usr/local/src/pcre \\
--add-module=/usr/local/src/ngx_devel_kit \\\
--add-module=/usr/local/src/lua-nginx-module \\
--with-http_stub_status_module \\
--with-http_ssl_module \\
--with-http_v2_module \\
--with-http_image_filter_module \\
--with-http_gzip_static_module \\
--with-http_gunzip_module \\
--with-stream \\
--with-stream_ssl_module \\
--with-ipv6 \\
--with-http_sub_module \\
--with-http_flv_module \\
--with-http_addition_module \\
--with-http_realip_module \\
--with-http_mp4_module \\
--with-ld-opt="-Wl,-E" \\
--with-ld-opt=-ljemalloc

make && make install
```

## 添加到环境变量中

```shell
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/nginx
```

## 启动、停止、重启

```shell
## 检查配置文件
nginx -t
## 启动
nginx
## 以特定目录下的配置文件启动 nginx:
nginx -c /path/to/nginx.conf
## 快速停止
nginx -s stop
## 重新打开日志文件，这样 nginx 会把新日志信息写入这个新的文件中
nginx -s reopen
## 修改配置后重新加载生效（平滑重启）
nginx -s reload
## 完成当前所有工作任务后再停止
nginx -s quit
## 批量强制杀死 nginx
killall nginx
```

> 检查配置文件、启动和重启的命令可以通过参数 ` -c /usr/local/nginx/conf/nginx.conf` 指定配置文件

## 添加到系统服务中

也可以配置一个 `systemctl` 命令来方便操作 nginx 的启动、停止和重启命令 ~ `wsl` 下无法使用 `systemctl` 可以选择使用 `service`

### systemctl

在 /usr/lib/systemd/system/ 目录下面新建一个 nginx.service 文件，并赋予其可执行权限

```shell
vim /usr/lib/systemd/system/nginx.service
chmod +x /usr/lib/systemd/system/nginx.service
```

服务文件内容

```shell
[Unit]
Description=nginx - high performance web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

在启动服务之前，需要先重载systemctl命令

```shell
## 重载systemctl命令
systemctl daemon-reload
## 启动服务
systemctl start nginx
## 停止命令
systemctl stop nginx
## 重新载入配置文件
systemctl reload nginx
```

### service 服务

执行 `vim /etc/init.d/nginx`，粘帖以下内容，注意文件内容中的安装目录根据实际情况进行编辑

```shell
#! /bin/sh
# chkconfig: 2345 55 25
# Description: Startup script for nginx webserver on Debian. Place in /etc/init.d and
# run 'update-rc.d -f nginx defaults', or use the appropriate command on your
# distro. For CentOS/Redhat run: 'chkconfig --add nginx'

### BEGIN INIT INFO
# Provides:          nginx
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
NAME=nginx
NGINX_BIN=/usr/local/nginx/sbin/$NAME
CONFIGFILE=/usr/local/nginx/conf/$NAME.conf
PIDFILE=/usr/local/nginx/logs/$NAME.pid
ulimit -n 8192
case "$1" in
    start)
        echo -n "Starting $NAME... "
		if [ -f $PIDFILE ];then
			mPID=`cat $PIDFILE`
			isStart=`ps ax | awk '{ print $1 }' | grep -e "^${mPID}$"`
			if [ "$isStart" != '' ];then
				echo "$NAME (pid `pidof $NAME`) already running."
				exit 1
			fi
		fi

        $NGINX_BIN -c $CONFIGFILE

        if [ "$?" != 0 ] ; then
            echo " failed"
            exit 1
        else
            echo " done"
        fi
        ;;

    stop)
        echo -n "Stoping $NAME... "
		if [ -f $PIDFILE ];then
			mPID=`cat $PIDFILE`
			isStart=`ps ax | awk '{ print $1 }' | grep -e "^${mPID}$"`
			if [ "$isStart" = '' ];then
				echo "$NAME is not running."
				exit 1
			fi
		else
			echo "$NAME is not running."
			exit 1
        fi
        $NGINX_BIN -s stop

        if [ "$?" != 0 ] ; then
            echo " failed. Use force-quit"
            exit 1
        else
            echo " done"
        fi
        ;;

    status)
		if [ -f $PIDFILE ];then
			mPID=`cat $PIDFILE`
			isStart=`ps ax | awk '{ print $1 }' | grep -e "^${mPID}$"`
			if [ "$isStart" != '' ];then
				echo "$NAME (pid `pidof $NAME`) already running."
				exit 1
			else
				echo "$NAME is stopped"
				exit 0
			fi
		else
			echo "$NAME is stopped"
			exit 0
        fi
        ;;
    restart)
        $0 stop
        sleep 1
        $0 start
        ;;

    reload)
        echo -n "Reload service $NAME... "
		if [ -f $PIDFILE ];then
			mPID=`cat $PIDFILE`
			isStart=`ps ax | awk '{ print $1 }' | grep -e "^${mPID}$"`
			if [ "$isStart" != '' ];then
				$NGINX_BIN -s reload
				echo " done"
			else
				echo "$NAME is not running, can't reload."
				exit 1
			fi
		else
			echo "$NAME is not running, can't reload."
			exit 1
		fi
        ;;

    configtest)
        echo -n "Test $NAME configure files... "
        $NGINX_BIN -t
        ;;

    *)
        echo "Usage: $0 {start|stop|restart|reload|status|configtest}"
        exit 1
        ;;
esac
```

修改文件权限并启动服务

```shell
chmod 755 /etc/init.d/nginx
## 启动服务
service nginx start
## 停止服务
service nginx stop
## 重新载入配置文件
service nginx reload
## 强制停止然后再重启服务
service nginx restart
## 查看状态
service nginx status
```

## 测试并访问

浏览器访问 http://localhost

![](https://static.acme.top/wp-content/uploads/2019/04/paste-fbc375cd9f7c6fcc50eb833f46b94cac-1.png?w=549&h=228)

如上图说明安装成功！