---
title: "Git fork 后与原仓库同步"
date: 2021-04-15T22:16:46+08:00
draft: false
categories: 
  - develop
tags: 
  - git
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
themeColor: "#fa541c"
## 封面颜色
coverColor: "rgba(0,0,0,0.7)"
## 封面图片
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210417211044-git-sync-forked-repo.png
## 文章页封面图分割：0-默认小图模式 1-大图模式
coverStyle: 1
---

<info>

> Git fork 了其他的仓库后，后续与原仓库的同步操作记录如下

</info>

## 说明

| 名词 | 解析                                    |
| ---- | --------------------------------------- |
| 上游 | 命令中定义的 upstream，即被 fork 的仓库 |
| 下游 | fork 之后的仓库                         |

## 命令

```bash
# 查看当前远程仓库的信息
git remote -v
# git remote add <name> <url>
# url 使用 ssh 或者 https 均可
git remote add upstream xxxxx
# 从上游（upstream）拉取代码合并到自己的分支上
git pull upstream master
# 推送本地变更到自己的远端仓库
git push
# 推送到远端指定分支
git push <远程主机名> <本地分支名>:<远程分支名>
# 如下
git push origin master:master
```

## 参考

- https://blog.csdn.net/shenshaoming/article/details/115295943