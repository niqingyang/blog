---
title: "NPM - 检查并更新项目依赖的版本"
date: 2019-07-28T21:39:06+08:00
draft: false
categories:
    - web
tags:
    - nodejs
    - npm
coverColor: "#942b2bf7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410224944-npm-libs.png
---

## 前言

经常会遇到 package.json 中的库有更新，但是太多一个一个的来很费事，幸好有个工具 `npm-check-updates` 可以帮助我们检查版本是否有变化

## 安装

```shell
npm install -g npm-check-updates
```

## 用法

在当前目录中显示项目的任何新依赖项：

```shell
## 检查当前目录下可更新的依赖项
ncu
## 升级 package.json
ncu -u
## 根据更新的 package.json 安装新版本
npm install
```

<info>

> `npm-check-updates -u` 仅修改 package.json 文件。运行 `npm install` 以更新已安装的软件包和package-lock.json

</info>

检查全局包：

```shell
## 添加 -u 以获取升级的单行命令
ncu -g
```

使用 `--filter` 更新指定依赖

```shell
## 以下四中写法作用相同
ncu --filter one, two, three
nuc -f one, two, three
ncu one, two, three
ncu one two three
```

使用 `--reject` 排除指定依赖

```shell
ncu --reject one, two, three
ncu -x one, two, three
```

使用正则表达式匹配

```shell
## 匹配以 “gulp-” 开头的依赖项
ncu '/^gulp-.*$/'
## 匹配不以 “gulp-” 开头的依赖项
ncu '/^(?!gulp-).*$/'
```

<warning>

> 使用这则表达式匹配时，正则表达式放在 `单引号` 内

</warning>

## 配置文件

使用 `.ncurc.{json,yml,js}` 文件指定配置信息。可以指定文件名和路径使用 `--configFileName` 和 `--configFilePath` 命令行选项

例如 `.ncurc.json`：

```json
{
	"upgrade": true,
	"filter": "express",
	"reject": [
		"@types/estree",
		"ts-node"
	]
}
```

## 项目地址

以上应该可以满足日常需求了，更多用法请移步项目主页：

[gitcard url='https://github.com/tjunnone/npm-check-updates']

## 参考文档

[https://github.com/tjunnone/npm-check-updates](https://github.com/tjunnone/npm-check-updates)