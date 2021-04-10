---
title: "Let’s Encrypt-申请免费通配符证书"
date: 2018-12-19T00:33:04+08:00
draft: false
categories:
    - ops
tags:
    - certbot
    - letsencrypt
    - linux
    - ubuntu
    - win10
themeColor: "#28426e"
coverColor: "#28426ef7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224247-letsencrypt.png
---

<info>

> 本文安装教程是在 Ubuntu 系统上进行的操作，如果您是使用的 Windows10 操作系统，可以参考文章 [windows下安装linux子系统](https://acme.top/wsl-install "windows下安装linux子系统") 来安装 Ubuntu 系统。

</info>

## 网站升级为HTTPS的必要性

1. 使用了 https 的网站，浏览器和服务器间的数据传输是加密的，更安全。
2. 使用了 https 的网站才能使用 http2（支持多路复用、压缩头信息、请求划分优先级、支持服务器端主动推送等新特性）。
3. 2018年1月1日起微信公众号 API 仅支持 https 调用。
4. 2018年7月起，Chrome 浏览器的地址栏将把所有 http 网站标识为不安全的网站。
5. 搜索引擎对 https 网站评分更高，有利于搜索引擎排名。
   ...

## Let's Encrypt

网站要支持 https 需要安装证书，而证书是由 CA 机构签发的，大部分传统的 CA 机构（例如：GlobalSign、GeoTrust、Verisign ）签发证书一般都是需要按年收费的，价格不菲。很多小公司、创业团队、博客或者个人开发者往往都不是很原因承担这笔费用，这样不利于推动 https 协议的使用。

Let's Encrypt 是一个**免费**、开放，自动化的 CA 机构，由 ISRG（Internet Security Research Group）运作。ISRG 是一个关注网络安全的公益组织，其赞助商包括 Mozilla、Akamai、Cisco、EFF、Chrome、IdenTrust、Facebook等公司。ISRG 的目的是消除资金和技术领域的障碍，全面推进网站从HTTP到HTTPS过度的进程。

截至2018年7月底，Let's Encrypt root（ISRG Root X1）直接受到Microsoft产品的信任。目前受到所有主要root程序的信任，包括Microsoft，Google，Apple，Mozilla，Oracle和Blackberry。

Let’s Encrypt 是一个非盈利性的组织，为了控制开支，他们搞了一个非常有创意的事情，设计了一个 ACME 协议，目前该协议的版本是 v2，也只有升级到 v2 协议才能支持通配符证书。

## 什么是通配符证书

在没有出现通配符证书之前，Let’s Encrypt 仅支持两种证书：

1. 单域名证书：证书仅仅包含一个主机。
2. SAN 证书：也称为域名证书，一张证书可以包括多个主机（Let’s Encrypt 限制是 20）。

对于个人用户来说，由于主机并不是太多，所以使用 SAN 证书完全没有问题，但是对于大公司来说有一些问题：

1. 子域名非常多，而且过一段时间可能就要使用一个新的主机。
2. 注册域也非常多。

对于大企业来说，SAN 证书可能并不能满足需求，而所有的主机全部包含在一张证书中，使用 Let’s Encrypt 证书（限制20）是无法满足的。

通配符证书就是证书中可以包含一个通配符，比如 *.example.com, *.example.cn，用 * 号来自动匹配所有子域名，大型企业也可以使用通配符证书了，一张证书可以放置更多的主机了。

## 申请 Let’s Encrypt 通配符证书

Let’s Encrypt 对 ACME 协议的实现进行了升级，只有 v2 协议才能支持通配符证书，任何客户端只要支持 ACME v2 版本，就可以申请通配符证书。

官方介绍 Certbot 0.22.0 版本支持新的协议版本

在了解该协议之前有几个注意点：

1. 客户在申请 Let’s Encrypt 证书的时候，需要校验域名的所有权，证明操作者有权利为该域名申请证书，目前支持三种验证方式：

- dns-01：给域名添加一个 DNS TXT 记录。
- http-01：在域名对应的 Web 服务器下放置一个 HTTP well-known URL 资源文件。
- tls-sni-01：在域名对应的 Web 服务器下放置一个 HTTPS well-known URL 资源文件。

**而申请通配符证书，只能使用 dns-01 的方式**

## 在 Ubuntu 系统下安装 Certbot

首先安装 Certbot。打开官网 [https://certbot.eff.org/](https://certbot.eff.org/ "https://certbot.eff.org/") ，选择申请证书的使用方式后，就会出现相关的安装命令了。

![选择自己的运行环境和操作系统版本](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204543-paste-ddacdb0f1cd409ccb492967bd140957c-1.png)

![自动生成的安装命令](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204556-paste-3f5ec54c65fed0c59f0df4ef92154dab-1.png)

我当前的 Ubuntu 版本为 Ubuntu 18.04.1 LTS，看到的安装命令为：

```shell
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot 
```

这些命令执行成功后，申请证书的安装工具 certbot 就安装完成了。

我打算给 acme.top 这个域名申请通配符证书，执行命令如下：

```shell
$ sudo certbot certonly  -d *.acme.top --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

相关参数释义：

- certonly，表示安装模式，Certbot 有安装模式和验证模式两种类型的插件。
- --manual 表示手动安装插件，Certbot 有很多插件，不同的插件都可以申请证书，用户可以根据需要自行选择。
- -d 为那些主机申请证书，如果是通配符，输入 *.acme.top（可以替换为你自己的域名）。
- --preferred-challenges dns，使用 DNS 方式校验域名所有权。
- --server，Let's Encrypt ACME v2 版本使用的服务器不同于 v1 版本，需要显示指定。

命令执行后输出如下图：

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204608-paste-9921c9053833eb559cf67c1e67e0abde-1.png)

上图交互中有两个交互的提示：

- 是否同意 Let's Encrypt 协议要求
- 询问是否对域名和机器（IP）进行绑定

确认同意后才能继续，同意后要求配置 DNS TXT 记录，也就是验证域名的所有权，判断证书申请者是否有域名的所有权。

上面输出要求给 `_acme-challenge.acme.top` 配置一条 TXT 记录，在没有确认 TXT 记录生效前不要按回车执行。

否则会验证失败如下图：

![DNS验证失败](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204808-paste-5fff6665685b69f28b03f4f13162a9b5-1.png)

我使用的是阿里云的域名服务器，登录控制台配置如下图：

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204815-paste-418a153f2680edce320e1c2af85a2667-1.png)

然后打开一个新的终端输入下面的命令来确认 DNS 解析是否生效：

```shell
dig -t txt _acme-challenge.acme.top
```

输出结果如下则表示 DNS 记录已经生效了

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204825-paste-ee55e4169a94b1b6e41c177881555f45-1.png)

然后回到上一个终端中敲回车键执行，输出如下图，则表示证书生成成功：

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204831-paste-8001ed1b178ded5a15dcf32a998d44f1-1.png)

输出中表示把相关证书保存在了 `/etc/letsencrypt/live/acme.top/` 这个目录下，生成的文件如下图：

![需要root用户才能进入](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410204839-paste-21ebbcfe4ea164c04160939381711a0b-1.png)

至此，申请通配符证书就完成了。

