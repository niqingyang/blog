---
title: "linux yum 命令"
date: 2018-12-10T00:35:19+08:00
draft: false
categories: 
    - develop
tags:
    - linux
    - yum
    - centos
themeColor: "#04948c"
coverColor: "#04948cf7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224513-linux-yum.png
---

## yum 是什么

yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。这里所谓的RPM包就是类似Windows系统里的安装文件。其他的比如 Ubuntu 系统里的 apt-get 命令也是用于管理安装包的，功能与 yum 类似，只是在 Ubuntu 中叫 deb包。

yum 主要功能是更方便的添加/删除/更新RPM包，他能够自动解决包的依赖性问题，便于管理大量系统更新问题，作用感觉和 Java 中的 Maven、PHP 中的 Composer、NodeJS 中的 NPM 类似。

yum 可以通过修改配置文件 `/etc/yum.conf`，来配置多个资源库。

## yum 和 apt-get 区别

一般来说著名的 linux 系统基本上分两大类：

1. RedHat系列：Redhat、CentOS、Fedora等
2. Debian系列：Debian、Ubuntu等

RedHat 系列
1. 常见的安装包格式 rpm包，安装rpm包的命令是“rpm -参数”
2. 包管理工具 yum
3. 支持tar包

Debian系列
1. 常见的安装包格式 deb包,安装deb包的命令是“dpkg -参数”
2. 包管理工具 apt-get
3. 支持tar包

## 什么是 rpm包

tar 只是一种压缩文件格式，所以，它只是把文件压缩打包而已。

rpm 相当于windows中的安装文件，它会自动处理软件包之间的依赖关系。一般都是预先编译好的文件，它可能已经绑定到某种CPU或者发行版上面了。

tar 一般包括编译脚本，你可以在你的环境下编译，所以具有通用性。如果你的包不想开放源代码，你可以制作成rpm，如果开源，用tar更方便了。

tar 一般都是源码打包的软件，需要自己解包，然后进行安装三部曲，./configure, make, make install. 来安装软件。

rpm 是redhat公司的一种软件包管理机制，直接通过rpm命令进行安装删除等操作，最大的优点是自己内部自动处理了各种软件包可能的依赖关系。

## yum 的语法

```shell
yum [options] [command] [package ...]
```

- options：可选，选项包括-h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。
- command：要进行的操作。
- package：操作的对象。


## yum 的使用

更新所有 rpm 包（系统升级）
```shell
yum update
```

升级内核
```shell
yum update kernel
```

安装 rpm 包
```shell
yum install <package_name>
```

移除指定 rpm 包
```shell
yum remove <package_name>
```

清理已安装过的缓存数据（相当于 `yum clean packages` + `yum clean oldheaders`）
```shell
yum clean all
```

搜索指定包的详细信息
```shell
yum search <keyword>
```

列出所有包
```shell
yum list [keyword]
```

显示包的详细信息
```shell
yum info <package_name>
```

列出所有可更新的软件包
```shell
yum list updates
```

列出所有已安装的 rpm 包
```shell
yum list installed
```

列出所有已安装但不在 Yum Repository 内的软件包
```shell
yum list extras
```

列出指定的 rpm 包
```shell
yum list <package_name>
```

检查可更新的 rpm 包
```shell
yum check-update
```

清理缓存中的 rpm 包文件
```shell
yum clean packages
```

清理缓存中的 rpm 头文件
```shell
yum clean headers
```

清理缓存中旧的 rpm 头文件
```shell
yum clean oldheaders
```

按照提供商根据指定关键词进行搜索 npm 包
```shell
yum provides <keyword>
```

## yum-fastestmirror 插件

类似java中的maven，软件包可以存在很多仓库中，有的仓库慢有的仓库快，对此，可以安装 yum-fastestmirror 这个插件，它会自动选择最快的镜像进行下载安装：
```shell
yum install yum-fastestmirror
```
安装插件后就会在每次安装 rpm 包时，都会先加载 fastestmirror 插件寻找最快的镜像了

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205239-paste-31fbbb0d40dafa5b93bbe48fba9652aa-1.png)

## centos 下安装 SCL

CentOS上的一些库比较古老，特别是我这种对新版本的PHP、Java等有强烈需求的开发者，于是，出现了许多第三方软件库，如EPEL、RPM Fusion、Remi等，这些库提供了很多安装包的新版本，方便我们的系统能与时俱进，但是第三方软件库有几个缺点：

1. 他们提供的软件没有经过CentOS官方测试，在安装软件的同时，可能会替换掉系统的一些核心文件，造成系统不稳定。
2. 第三方库安装的软件可能不保证兼容性，也许对系统升个级就会导致某个软件没法使用。

所以推荐用SCL（Software Collections）软件库。SCL属于CentOS官方的软件库，经过充分测试，安装软件时不会替换系统的核心文件，保证了系统的稳定性。安装很简单，命令就这一条：
```shell
install centos-release-scl-rh
```

此时如果你运行php命令，系统依然会提示你command not found。这是因为，SCL的风格就是把软件对系统的影响减少到最小，甚至安装完PHP，php命令都不会被添加到 $PATH 变量中，所以你没法直接执行软件中的命令的。需要通过 scl enable 命令显示执行：

查看 scl 下安装了哪些软件的命令：
```shell
scl -l
```

启用命令
```shell
scl enable XXX bash
```




