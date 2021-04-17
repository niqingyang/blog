---
title: "go-astilectron-demo 编译后启动不了"
date: 2019-08-15T22:35:53+08:00
draft: false
categories:
    - develop
tags:
    - go
themeColor: "#00bcd4"
coverColor: "#00bcd4"
---

## 前言

这几天学习 Go 语言，看着 helloWorld 的几个示例，突然冒出既然 Go 编译这么方便，其 GUI 库会是什么样的？于是左搜右查，发现 Go 居然没有官方的 GUI 库！网评的意思是：“官方本来就没有心发展 Go 的 GUI 这块” ~ 好在互联网上已经涌现出不少成熟、好用的第三方界面库，使用它们，就同样可以写出同 C#、C++ 的界面，而且效率还更胜一筹，例如 [用于构建 GUI 应用程序的 GO 库](https://github.com/avelino/awesome-go#gui)，对比下来，感觉 go-astilectron 看上去不错，于是尝试了一下，期间遇到了些问题，记录于此。

## 简介

`go-astilectron` 使用 GO 和 HTML/JS/CSS 构建跨平台 GUI 应用程序（由 Electron 提供支持）。简单说就是基于 Electron 使用 H5 构建 GUI，对于了解前端的开发人员着实容易上手，免去了了解各种 GUI 函数的麻烦。

## 示例

`go-astilectron` 提供了一个 demo，Github地址：

[gitcard url='https://github.com/asticode/go-astilectron-demo']

参考 READEME 文档，下载编译后可快速体验

## 问题

😭 但是，我按照步骤操作，编译后的 exe 文件居然启动不了，双击没有任何反应，搞了好久，就快要放弃的时候，从其 [Issues](https://github.com/asticode/go-astilectron-demo/issues/41#issuecomment-436901857 "Issues") 发现了些端倪 ~

## 排查

从 [Issues](https://github.com/asticode/go-astilectron-demo/issues/41#issuecomment-436901857 "Issues") 处可见，编译后的 exe 文件双击运行后，会在 `C:\\Users\\{用户名}\\AppData\\Roaming\\Astilectron demo` 目录下解压相关 H5 资源文件（我双击后此处确实会生成临时文件，可见双击 exe 文件后确实运行来着）

本着好奇的心，查看了目录下所有文件，点来点去，发现此目录下的压缩文件`vendor/electron-windows-amd64-v4.0.1.zip` 双击居然提示文件已损坏！于是删除这些缓存文件又编译了几次，还依旧是文件已损坏...，这就不淡定了，难道版本文件问题？！🙄

于是就查找怎么替换 `electron-windows-amd64-v4.0.1.zip` 这个文件，翻了一圈 Go 库，莫有找到... 😣

于是又回头示例目录下 `astilectron-bundler -v` 重新编译了一下，此时从输出的日志中颤抖的发现了 `electron-windows-amd64-v4.0.1.zip` 几个大字：

![](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410205627-paste-c1e368b8ef7892411463c57cba5cad47-1.png)

原来 `electron-windows-amd64-4.0.1.zip` 会被缓存在目录 `D:\\Users\\Temp\\astibundler\\cache` 下，缓存的文件也是已损坏，初步判断 `electron-windows-amd64-4.0.1.zip` 都是从这里复制过去的，于是删除此目录下的缓存，重新编译，果然，此文件会重新下载！！🎉😁🎉

然后颤抖着双击生成的 exe 文件，终于启动起来了！！此处泪流满面... 😭

![](https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410205634-paste-51caeca8cbaf3dd1ab72200834956711-1.png)

## 总结

可能是某一时刻下载文件 `electron-windows-amd64-4.0.1.zip` 时，网络不好，造成文件没下载完整导致损坏，然后就发生了上面这起事故，不过好在找到问题根源，解决了！！！

最后一句：凡事多从自身找问题~ 😋
