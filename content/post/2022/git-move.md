---
title: "Git 迁移到新地址"
date: 2022-11-06T14:02:22+08:00
draft: false
categories: 
  - develop
tags: 
  - git
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
## 主题色
themeColor: "#363534"
## 封面颜色
coverColor: "rgba(0,0,0,0.75)"
## 封面图片
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/11/20211113234817-git.png
## 文章页封面图分割：0-默认小图模式 1-大图模式
coverStyle: 1

---

## 操作步骤



1. 从原地址克隆一份裸版本库，比如原本托管于 `GitHub`

   ```bash
   git clone --bare git://github.com/username/old-project.git
   ```

2. 然后到新的 `Git` 服务器上创建一个新项目，比如 `new-project`。

3. 以镜像推送的方式上传代码到新的 `Git` 服务器上

   ```bash
   cd old-project.git
   git push --mirror git://github.com/username/new-project.git
   ```

4. 删除本地裸版本库

   ```bash
   cd ..
   rm -rf old-project.git
   ```

5. 或者 到新服务器 `Git` 上找到 `Clone` 地址，直接 `Clone` 到本地就可以了

   ```bash
   git clone git://github.com/username/new-project.git
   ```

   