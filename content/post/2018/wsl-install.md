---
title: "Windows下安装Linux子系统"
date: 2018-12-08T16:00:02+08:00
draft: false
categories:
    - software
tags:
    - wsl
    - windows
    - linux
coverColor: "#e3e3e3f7"
coverImage: https://static.acme.top/wp-content/uploads/2018/12/wsl.png
titleStyle: 1
---

<info>

>日常开发中总会遇到一些只能在linux下运行的软件或者框架，让像我这样的windows重度使用者甚是烦恼，估计是微软良心发现了，终于出手在windows下也可以跑linux了，从此什么虚拟机、cygwin都见鬼去吧！本文将介绍如何在windows下安装linux并使用。

</info>

## 什么是WSL？

WSL是Windows Subsystem for Linux的简称，就是Windows系统上的一个Linux子系统，安装后，不用再安装臃肿的Vmware和VirtualBox等虚机机系统，就可以直接在Windows上体验原生的Linux应用了，目前微软主要是和Canocical进行合作，推出的是Ubuntu系统，未来将会有更多的发行版供选择。

## 能做什么？
Windows子系统Linux允许开发人员直接在Windows上运行GNU / Linux环境 - 包括大多数命令行工具，实用程序和应用程序 - 未经修改，无需虚拟机的开销。

## 开始安装Linux

### 准备

<info>

> **提示**  
>- 本文适用于Windows build 16215或更高版本，建议升级windows 10到最新版本。
>- 在为WSL安装任何Linux发行版之前，必须确保启用“Windows Subsystem for Linux”可选功能，设置后需要重启电脑，注意重启前该保存的保存。

</info>

开启“Windows Subsystem for Linux”的两种方式：

1. 以管理员身份打开PowerShell并运行：

```shell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

2. 打开“启用或关闭windows功能”（从微软小娜或者控制面板中搜索打开）：

![](https://static.acme.top/wp-content/uploads/2018/12/paste-539412671e87211bbae3d1e9c93f2b7d-1.png?w=415&h=418)

### 选择Linux安装

1. 打开Microsoft Store搜索Linux，会出现Linux的专题，点击“获取这些应用”按钮。

![](https://static.acme.top/wp-content/uploads/2018/12/paste-a12839c7a1c91f431594289a0fb5dc77-1.png?w=1089&h=896)

2. 目前支持的Linux系统有 Ubuntu、SUSE、Debian、Kali，估计以后会支持更多的Linux发行版本，选择一个喜欢的Linux安装，此处我选择的是 Ubuntu 系统。

![](https://static.acme.top/wp-content/uploads/2018/12/paste-ed4fbf3e9d6e56ed8461a29aac78ce14-1.png?w=1099&h=714)

3. 安装后需要进行初始化，启动一个新的实例，可以通过单击Windows应用商店应用中的“启动”按钮，或从“开始”菜单启动已安装的 Ubuntu 来执行此操作。

![](https://static.acme.top/wp-content/uploads/2018/12/paste-ed15cfca305679eb470fee1539e14f55-1.png?w=324&h=405)

4. 第一次启动运行时，将打开一个控制台窗口，需要等待一两分钟才能完成安装。

<info>

> **提示**  
> 初始化过程中，Linux系统文件被解压缩并存储在电脑上，以后可以随时使用。这可能需要大概一分钟左右，以后再次运行基本都是很快的。

</info>

5. 安装完成后，系统将提示您创建新的用户帐户及密码，此账户和密码与windows系统没有任何关系。

![](https://static.acme.top/wp-content/uploads/2018/12/paste-afdf5660d357c5be71d0f51c6ace1d74-1.png?w=941&h=535)

当打开一个新的Linux命令行窗口后，是不需要输入用户名、密码的，但是如果使用`sudo`命令则将被要求输入密码，建议密码尽量设置的简单一些。

6. 更新软件包，执行以下命令：

```shell
sudo apt update && sudo apt upgrade
```

## 用户账户和权限

1. 创建的Linux账户将是Linux系统的默认账户，会在Linux系统启动后自动登录，此帐号默认隶属于Linux管理员。
2. 不同的Linux子系统的账户是相互独立的，每次安装新的Linux子系统时都需要创建新的账户。
3. Windows和Linux的权限是相互独立互不影响的。Windows权限模型管理进程对Windows资源的权限；Linux权限模型控制进程对Linux资源的权限。

## WSL管理和配置

1. Unbutu 系统安装的 windows 目录

```shell
C:/Users/XXX/AppData/Local/Packages/CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc/LocalState/rootfs/
```

![Windows下Linux子系统的目录结构](https://static.acme.top/wp-content/uploads/2018/12/paste-e16e0e673a22721838e5d116b1dd50c3-1.png?w=759&h=446)


2. 在Windows下运行Linux子系统的方法有：

```shell
- 在开始菜单中搜已安装的Linux子系统点击直接启动运行
- wsl.exe 或者 bash.exe
- wsl [command] 或者 bash -c [command]
```

3. 在命令行中运行`wsl`会直接进入默认的Linux子系统，并保留当前的工作目录，而在命令行中运行Linux子系统（例如：ubuntu）则会进入主目录。

4. 常用命令

- Linux子系统启动后会自动将各个盘符下的目录挂载到 `/mnt/` 目录下

```shell
访问C盘：/mnt/c/
访问D盘：/mnt/d/
...
```

- 列出所有可用的Linux子系统
```shell
wslconfig /list
```

- 列出所有的Linux子系统，包括不可用的
```shell
wslconfig /list /all
```

- 设置默认的Linux子系统，设置后执行 wsl [command] 将会在此默认的Linux子系统中运行
```shell
wslconfig /setdefault <DistributionName>
```

- 卸载Linux子系统
```shell
wslconfig /unregister <DistributionName>
```

<warning>

> **警告**  
> 卸载后，与该Linux子系统相关的所有数据、设置和软件将永久丢失。从商店重新安装将安装新的Linux子系统。

</warning>

## 构建自定义的Linux子系统

WSL不仅仅支持 Windows Store 里的那几个 Linux 系统，其实还支持 RedHat、CentOS 等 Linux 系统，只不过微软官方没有去搞，而是留给了开发者自己去自定义创建和维护，具体创建可以参考其Github [Distro Launcher GitHub存储库](https://github.com/Microsoft/WSL-DistroLauncher "Distro Launcher GitHub存储库") 上的说明创建自定义Linux发行版程序包。

但我觉得大部分有需求的基本都是想要个现成的系统来安装，Github [https://github.com/yuk7/wsldl](https://github.com/yuk7/wsldl "https://github.com/yuk7/wsldl") 这个项目就维护了不少 Linux 系统，包括：Alpine Linux、Arch Linux、Artix Linux、CentOS，参考说明即可很方便的安装。

![](https://static.acme.top/wp-content/uploads/2018/12/paste-a8912b71f223a14564f9a62e66daf583-1.png?w=834&h=534)

由于服务器端一般都选择 CentOS 系统，所以为了保持开发和生产环境一直，我本机安装的是 CentOS，安装非常方便，下载下来安装包后，解压到安装目录下，`以管理员身份运行`CentOS.exe即可，安装完成后通过执行命令 `wslconfig /list` 就会发现wsl下多了个 CentOS 系统。

![](https://static.acme.top/wp-content/uploads/2018/12/paste-4f0479e74eee351a299bfbe01f8f2c3a-1.png?w=859&h=452)

这个 CentOS 其实是从 docker 里镜像过来的，是个精简版的操作系统，很多命令都是缺失的，如果想用需要自己去 `yum` 安装。

<info>

> **提示**
> 安装一定要以管理员身份去运行，否则可能会安装失败。

<info>

参考文档：[https://docs.microsoft.com/zh-cn/windows/wsl/about](https://docs.microsoft.com/zh-cn/windows/wsl/about "https://docs.microsoft.com/zh-cn/windows/wsl/about")