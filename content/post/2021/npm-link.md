---
title: "NPM Link"
date: 2021-11-23T00:18:34+08:00
draft: false
categories:
    - web
tags: 
    - npm link
    - npm
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
# themeColor: 
## 封面颜色
coverColor: "#942b2bf7"
## 封面图片
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410225005-npm-libs.png
## 文章页封面图分割：0-默认小图模式 1-大图模式
# coverStyle: 0
---

## 前言

在本地进行前端开发时，如果想在项目中依赖另一个本地项目进行开发或者调试，那么可以尝试使用 `npm link `命令，使用起来简单、方便。

## 准备

例如，我要在本地项目 web-app 中引入自己封装的 my-ui 库

- my-ui 库，位于目录 `/my-ui` ，其 `package.json` 中声明的包名称为 `@my/ui`
- web-app  项目，位于目录 `/web-app`

## 创建链接

进入需要被依赖的库目录 `/my-ui` 中执行命令

```bash
npm link
```

执行命令后，`my-ui` 库会根据 `package.json`中声明的包名 `@my/ui`  被链接到全局，路径为 `{prefix}/lib/node_modules/<package>`



<info>

> 可以通过命令 `npm config get prefix` 获取到 `{prefix}`

</info>

然后进入项目 `web-app` 所在的目录 `/web-app` 中，执行命令

```bash
npm link @my/ui
```

执行后，会在 `/web-app/node_modules` 目录下创建 `@my/ui` 的快捷方式。这样，链接就创建完成了，现在对 `/my-ui` 的任何更改都将反应到 `/web-app/node_modules/@my/ui` 中。

## 删除链接

如果要删除全局安装的包的链接本身，则执行以下命令

```bash
npm uninstall -g @my/ui
```



## 参考

- [npm-link (官方文档)](https://docs.npmjs.com/cli/v6/commands/npm-link)

