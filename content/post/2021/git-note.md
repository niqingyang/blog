---
title: "Git 笔记"
date: 2021-11-13T23:47:38+08:00
draft: false
categories: 
  - develop
tags: 
  - git
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
themeColor: "#363534"
## 封面颜色
coverColor: "rgba(0,0,0,0.75)"
## 封面图片
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/11/20211113234817-git.png
## 文章页封面图分割：0-默认小图模式 1-大图模式
coverStyle: 1
---

## 子模块

git submodule - 子模块，允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持这两个项目提交的独立性。

比如：工作中的某个项目需要包含并使用另一个项目， 也许是第三方库，或者你独立开发的用于多个父项目的库。通过子模块就可以把它们当做两个独立的项目，同时又在一个项目中使用另一个。

### 常用操作



- 添加

```bash
# 进入父级仓库中
git submodule add <submodule_url> [submodules_path]
```

命令执行后，会 clone `submodule_url` 到`submodules_path`下，然后会在父仓库根目录下创建 `.gitmodules` 文件记录了`submodule_url`与本地目录之间的映射信息，如果有多个子模块，则该文件中会有多条记录。

```config
[submodule "submodules_path/sub"]
	path = submodules_path/sub
	url = submodule_url
```

并且会在 `.git/config`文件中增加 submodule 的信息

```config
[submodule "submodules_path/sub"]
	url = submodule_url
	active = true
```

- 检出

    克隆一个含有子模块的项目后，默认只是克隆了 .gitmodules  文件和会包含该子模块目录，但其中还没有任何文件，需要执行如下命令取出子模块中的文件：

```bash
# 初始化本地配置文件
git submodule init
# 从该项目中抓取所有数据并检出父项目中列出的合适的提交
git submodule update
```

​		或者一条组合命令：

```bash
# 组合命令
git submodule update --init
# 如需初始化、抓取并检出任何嵌套的子模块，可增加参数 `--recursive`
git submodule update --init --recursive
```

- 更新

    子模块的维护者提交了更新后，使用子模块的项目必须手动更新才能包含最新的提交。

    在项目中，进入到子模块目录下，执行 `git pull` 更新，查看 `git log` 查看相应提交。

    完成后返回到项目目录，可以看到子模块有待提交的更新，使用`git add`，提交即可

- 删除

```bash
# 如果未指定 path 或者 --all 参数，并不会删除所有而是报错，以防止误删除
# --force 参数将强制移除子模块的工作树，及时它包含了本地修改
git submodule deinit [-f|--force] (--all|[--] <path>…)
# 从工作树和索引中删除文件
git rm --cached <submodules_path/submodule_name>
```

​		上述命令可以卸载一下子模块，会自动将 `.git/config` 和 `.gitmodules` 文件中的相关子模块内容移除。

- 克隆带有子模块的项目

```bash
# 递归克隆整个项目
git clone <repository> --recursive
```

- 子模块遍历

    `foreach` 子模块命令，它能在每一个子模块中运行任意命令。 如果项目中包含了大量子模块，这会非常有用。

```bash
git submodule [--quiet] foreach [--recursive] <command>
```

​		例如：

```bash
# 拉取所有子模块
git submodule foreach git pull
# 创建一个新分支，并将所有子模块都切换过去
git submodule foreach 'git checkout -b featureA'
# 生成一个主项目与所有子项目的改动的统一差异
git diff; git submodule foreach 'git diff'
```

### 参考

- [Git中submodule的使用](https://zhuanlan.zhihu.com/p/87053283)
- [Git - 子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块)
