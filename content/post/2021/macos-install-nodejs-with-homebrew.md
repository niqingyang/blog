---
title: "MacOS 安装指定版本的 NodeJS"
date: 2021-02-19T21:35:56+08:00
draft: false
categores:
    - 前端开发
tags:
    - nodejs
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
themeColor: "#389e0d"
## 封面颜色
# coverColor: "rgba(0,0,0,0.3)"
## 封面图片
## coverImage: 
---

## 安装

查找可用的 node 版本

```bash
brew searh node
```

根据需要选择适合的版本，我这里现在 node 12 进行安装

```bash
brew install node@12
```

<warning>

> 如果之前安装过 node，可以使用命令 `brew uninstall node` 卸载掉，再使用 `brew unlink node` 解绑 node

</warning>

## 链接

install 后还不算安装完成，此时在命令行中是无法使用 node 和 npm 的，需要链接安装的node，执行此命令：

```bash
brew link node@12
```

执行成功后会提示：

```bash
Linking /usr/local/Cellar/node@12/12.20.2... 3857 symlinks created.

If you need to have this software first in your PATH instead consider running:
  echo 'export PATH="/usr/local/opt/node@12/bin:$PATH"' >> ~/.zshrc
```

按照提示添加环境变量：

```bash
vim ~/.zshrc

# node
export PATH="/usr/local/opt/node@12/bin:$PATH"
```


## 完成

在命令中检查版本

```bash
node -v

v12.20.2
```

## 安装 cnpm

受限于 npm 服务器在国外，下载包时网络速度慢，建议使用淘宝的 npm 镜像，其官方声明：这是一个完整 npmjs.org 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10分钟 一次以保证尽量与官方服务同步。

安装 cnpm

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 设置 electron 淘宝镜像

如果开发 electron 的话，建议设置此镜像

```bash
npm config set ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/
```

<info>

> 上述命令修改了用户目录下的文件 `.npmrc`，追加了：`ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/`

</info>