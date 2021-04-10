---
title: "下载利器 – Aria2"
date: 2020-06-27T21:05:54+08:00
draft: false
categories:
    - software
tags:
    - aira2
    - 下载
themeColor: "#404a5d"
coverColor: rgba(65, 73, 90, 0.97)
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410225219-aria2.png
coverStyle: 0
---

## 前言

之前在 windows 上的时候一直使用 迅雷 作为下载工具，切换到 mac 后，突然没了 GUI 下载工具后开始还有些不习惯，搜了一圈发现了 Aria2，发现这款命令行的下载工具体验也不错，用起来感觉比迅雷要好，至少没有烦人的广告和弹框了，一切可控，于是打算记录一下安装和使用~

## 简介

[Aria2](https://aria2.github.io/ "Aria2") 是一个免费、开源、轻量级的多协议和多源命令行下载实用程序。

- 支持 HTTP/HTTPS、FTP、SFTP、BitTorrent 和 Metalink。

- 支持通过内置的 JSON-RPC 和 XML-RPC 接口进行操作。

- 支持在下载文件时自动验证数据块。

- 支持从多个源/协议下载文件，并尝试利用您的最大下载带宽。

- 默认情况下，所有 Linux 发行版都包含 `Aria2`，因此可以轻松地从官方存储库进行安装。

## 安装

MacOS 下可以使用 `brew` 进行安装：

```bash
brew install aria2
```

Ubuntu 下可以使用 `apt` 进行安装：

```bash
apt install aria2
```

Windows 下可以在 GitHub 仓库中下载对应的 Windows 版本：[下载地址](https://github.com/aria2/aria2/releases "下载地址")

## 使用

- 从 web 下载

```bash
aria2c http://example.org/mylinux.iso
```

- 从 2 个来源下载：

```bash
aria2c http://a/f.iso ftp://b/f.iso
```

- 使用每个主机使用 2 个连接下载：

```bash
aria2c -x2 http://a/f.iso
```

- BT下载：

```bash
aria2c http://example.org/mylinux.torrent
```

- 磁力链接：

```bash
aria2c 'magnet:?xt=urn:btih:248D0A1CD08284299DE78D5C1ED359BB46717D8C'
```

- Metalink：

```bash
aria2c http://example.org/mylinux.metalink
```

- 批量下载文本文件中的 URI：

```bash
aria2c -i uris.txt
```

## 配置

`Aria2` 支持很多参数，例如：下载连接数、下载速度限制、下载后的目录 等等，这些即可以通过命令参数控制，也可以一劳永逸的通过配置文件来管理。

`Aria2` 默认配置文件的地址是 `$HOME/.aria2/aria2.conf`，支持通过参数 `--conf` 来指定配置文件的路径，也可以通过 `--no-conf` 来禁止加载配置文件。


<details>

<summary> aria2.conf <badge class="">常用配置</badge></summary>

```properties

## 基本配置 ##

# 文件的保存路径，默认：当前位置
dir=/Users/niqingyang/Downloads/
# 断点续传
continue=true

## 下载连接相关 ##

# 最大同时下载任务数，默认：5
max-concurrent-downloads=16
# 同一服务器连接数，默认：1
max-connection-per-server=10
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=1M
# 单个任务最大线程数, 添加时可指定, 默认:5
split=10
# 整体下载速度限制, 运行时可修改, 默认:0
# max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
# max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
max-overall-upload-limit=20kb
# 单个任务上传速度限制, 默认:0
max-upload-limit=5kb
# 禁用IPv6, 默认:false
disable-ipv6=true
# 禁用https证书检查
check-certificate=false
#运行覆盖已存在文件
allow-overwrite=true
#如果已存在同一文件，自动重命名
auto-file-renaming=true

## 进度保存相关 ##

# 从会话文件中读取下载任务
# input-file=/Users/niqingyang/.aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
# save-session=/Users/niqingyang/.aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
# save-session-interval=30

## BT/PT下载相关 ##

# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
#follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 单个种子最大连接数, 默认:55
#bt-max-peers=55
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=true
# 打开IPv6 DHT功能, PT需要禁用
enable-dht6=false
# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999
# 本地节点查找, PT需要禁用, 默认:false
bt-enable-lpd=true
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=true
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-UT341-
user-agent=uTorrent/341(109279400)(30888)
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=1.0
# 强制保存会话, 话即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
#bt-save-metadata=false
#仅下载种子文件
bt-metadata-only=true
#通过网上的种子文件下载，种子保存在内存
follow-torrent=mem

# 根据可用的带宽优化并发下载次数
optimize-concurrent-downloads=true
# 将间隔（以秒为单位）设置为输出下载进度摘要。设置将禁止显示输出
summary-interval=0

## RPC相关设置 ##

# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC 监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的 RPC 授权令牌，在设置 AriaNg 时需要用到，请手动更改
rpc-secret=<Secret Key>
```

</details>

## AriaNG

AriaNG 是一个利用 Aria2 JSONPRC 接口实现的可视化 WEB 界面，支持独立安装，也可以作为扩展安装在 Edge 和 Chrome 浏览器中（推荐这种方式），这样稍加配置后就可以拦截浏览器的下载任务，将所有下载任务都交给 aria2 来处理（特别是百度网盘的文件）。

### 安装

此处以 Edge 为例（无需翻墙），在官方扩展商店中搜索 aria2，安装如下图的扩展：

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223553-apps.32562.625b4f3d-bb75-494e-baf4-693b199e4ca7.a178a838-46e7-4016-bfa7-83ba339fd810.4ada587c-e694-4345-a869-2f1e45a7a627)

### 配置

打开扩展的选项页面，勾选各种自动拦截下载的选项，在 `JSON-RPC` 一行添加 `Secret Key`，也就上面配置中 `rpc-secret=<Secret Key>` 中的 `<Secret Key>`，然后点击 `Save` 按钮即可。

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223558-paste-375f855a7375252ae68fe9e240786762-1.png)

### 启动

- 第一种：通过配置文件启动（推荐）

```bash
## 通过配置文件配置启动
aria2c -D
```

- 第二种：通过命令行参数启动

```bash
aria2c --enable-rpc --rpc-listen-all=true --rpc-allow-origin-all -c -D
```

<warning>

> -D  用于后台执行，daemon 模式, 这样ssh断开连接后程序不会退出，和 screen 一样的效果。

</warning>

### 界面

配置无误后打开扩展界面，会显示已连接

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410223608-paste-eea4dbd8120310ba27d831eb7b9462f3-1.png)

这样配置完成后，再下载文件都会被 AriaNg 扩展拦截并通过 Aria2 下载了~


## 参考

- [安装Aria2+AriaNg](https://blog.haitianhome.com/install-aria2-web-ariang.htmlhttp:// "安装Aria2+AriaNg")

- [aria2的安装与配置](https://www.cnblogs.com/awakenedy/p/10985061.html "aria2的安装与配置")

- [aria2 使用说明](https://github.com/erasin/notes/blob/master/linux/soft/aria2.md "aria2 使用说明")
