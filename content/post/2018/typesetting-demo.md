---
title: "排版示例"
date: 2018-10-03T16:43:44+08:00
draft: false
categories: 
    - record
tags:
    - 排版
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
# themeColor: 
## 封面颜色
# coverColor: "rgba(0,0,0,0.3)"
## 封面图片
# coverImage: 
---



<info>

> Markdown 现在这么流行，并且确实提升写文章的效率，可读性很强，我也毫不犹豫的加入了 Markdown 的大军。在 Markdown 的基础上，借助HTML注释和插件的功能，对 Markdown 进行了一些扩展，使文章内容更漂亮，下面是一些 Demo。

</info>



## 超链接

[超链接示例](#)

```markdown
[链接名称](链接地址)
```

## 徽章

<badge>徽章</badge>
<badge class="success">徽章</badge>
<badge class="warning">徽章</badge>
<badge class="danger">徽章</badge>

```html
<badge>徽章</badge>
<badge class="success">徽章</badge>
<badge class="warning">徽章</badge>
<badge class="danger">徽章</badge>
```

## 详情

<details open>
	<summary> 标题  </summary>
  内容
</details>

```markdown
<details open>
	<summary> 标题 </summary>
	内容
</details>
```


## 表格



| 序号 | 名称        | 内容             |
| :--- | :---------- | ---------------- |
| 1    | Spring Boot | 快速搭建Web工程  |
| 2    | Wordpress   | 主题和插件的开发 |

```markdown
|  序号  |  名称  |  内容  |
| ------------ | ------------ | ------------ |
|  1  |  Spring Boot  |  快速搭建Web工程  |
|  2  |  Wordpress |  主题和插件的开发  |
```

## 引用：默认

> 这是默认引用

```markdown
> 这是默认引用
```

## 引用：提示

<success>

> **Tip**  
> 这是提示引用

</success>

```markdown
<success>

> **Tip**  
> 这是提示引用

</success>
```

## 引用：信息

<info>

> **Info**  
> 这是信息引用

</info>

```markdown
<info>

> **Info**  
> 这是信息引用

</info>
```

## 引用：警告

<warning>

> **Warning**  
> 这是警告引用

</warning>

```markdown
<warning>

> **Warning**
> 这是警告引用

</warning>
```

## 引用：危险

<danger>

> **Danger**  
> 这是危险引用

</danger>

```markdown
<danger>

> **Danger**  
> 这是危险引用

</danger>
```


