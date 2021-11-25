---
title: "Mac 下查看端口占用情况"
date: 2021-05-29T11:59:29+08:00
draft: false
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
# themeColor: 
## 封面颜色
# coverColor: "rgba(0,0,0,0.3)"
## 封面图片
# coverImage: 
## 文章页封面图分割：0-默认小图模式 1-大图模式
# coverStyle: 0
---

## 查看端口占用的命令

```bash
sudo lsof -i :8080
```

## 杀掉占用端口的进程

```bash
sudo kill -9 <PID>
```

## 关于 `lsof` 命令更多参考

- [linux lsof命令详解 - ggjucheng - 博客园 (cnblogs.com)](https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316599.html)
- [Linux 命令神器：lsof - 简书 (jianshu.com)](https://www.jianshu.com/p/a3aa6b01b2e1)
