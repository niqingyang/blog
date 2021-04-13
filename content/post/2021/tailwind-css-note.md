---
title: "Tailwind Css 笔记"
date: 2021-02-14T22:55:20+08:00
draft: false
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
# themeColor: 
## 封面颜色
# coverColor: "rgba(0,0,0,0.3)"
## 封面图片
# coverImage: 
---

## 安装

参考官方文档：[https://tailwindcss.com/docs/installation](https://tailwindcss.com/docs/installation)

## 核心概念

### 响应式设计

默认情况下，有五个断点，灵感来自常见的设备分辨率：


| 断点前缀  | 最小宽度  | Css  |
| ------------ | ------------ | ------------ |
| `sm`  | 640px  | `@media (min-width: 640px) { ... }`  |
| `md`  | 768px  | `@media (min-width: 768px) { ... }`  |
| `lg`  | 1024px  | `@media (min-width: 1024px) { ... }`  |
| `xl`  | 1280px  | `@media (min-width: 1280px) { ... }`  |
| `2xl`  | 1536px  | `@media (min-width: 1536px) { ... }`  |


若要添加实用程序，但仅在某个断点处生效，只需使用断点名称加上字符前缀该实用程序：`:`

```css
<!-- Width of 16 by default, 32 on medium screens, and 48 on large screens -->
<img class="w-16 md:w-32 lg:w-48" src="...">
```

通过配置文件 `tailwind.config.js` 自定义断点：

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'tablet': '640px',
      // => @media (min-width: 640px) { ... }

      'laptop': '1024px',
      // => @media (min-width: 1024px) { ... }

      'desktop': '1280px',
      // => @media (min-width: 1280px) { ... }
    },
  }
}
```

### 悬停、焦点和其他状态

|   |   |
| ------------ | ------------ |
| `hover:`  | 悬停  |
| `focus:`  | 焦点  |
| `active:`  | 激活  |
| `group` `group-hover:`  | 组悬停,子元素悬浮在指定的父元素上时需要设置子元素的样式时使用  |
| `group` `group-focus:`  | 类似组悬停  |
| `focus-within:`  | 仅在子元素获取焦点时起作用  |
| `focus-visible:`  | 仅在通过键盘获取到焦点时起作用  |
| `disabled:`  | 仅在元素被禁用时时起作用  |
| `visited:`  | 仅在链接被访问过时起作用  |
| `checked:`  | 仅在单元按钮和复选框被选中时起作用  |
| `first:`  | 仅在其父级的第一个子级元素上起作用  |
| `last:`  | 仅在其父级的最后一个子级元素上起作用  |
| `odd:`  | 仅在其父级的奇数子级元素上起作用  |
| `even:`  | 仅在其父级的偶数数子级元素上起作用  |
| `motion-safe:`  |   |
| `motion-reduce:`  |   |
| `dark:`  | 仅在暗黑模式下起作用  |

### 暗黑模式

由于文件大小考虑，默认情况下，在 Tailwind 中未启用暗模式变体.

修改配置文件 `tailwind.config.js` 的 `darkMode` 来启用暗黑模式：

```js
module.exports = {
    // false - 禁用暗黑模式
    // media - 跟随系统
    // class - 手动模式，通过在 html 标签上加入 class=“dark” 来激活暗黑模式
    darkMode: 'media',
    // ...
}
```

### 添加基本样式

- 在 html 中使用：仅设置一些全局样式

```html
<!doctype html>
<html lang="en" class="text-gray-900 leading-tight">
  <!-- ... -->
  <body class="min-h-screen bg-gray-100">
    <!-- ... -->
  </body>
</html>
```

- 使用 CSS

使用 `@layer base` `@apply` 定义或者覆盖一些样式

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  h1 {
    @apply text-2xl;
  }
  h2 {
    @apply text-xl;
  }
}
```

- 使用插件

通过编写插件并使用函数来添加基本样式：`addBase`

```js
// tailwind.config.js
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(function({ addBase, theme }) {
      addBase({
        'h1': { fontSize: theme('fontSize.2xl') },
        'h2': { fontSize: theme('fontSize.xl') },
        'h3': { fontSize: theme('fontSize.lg') },
      })
    })
  ]
}
```

### 提取组件

提取组件可以处理样式中重复的部分，并保持公共样式的可维护性

- 从 JS 层面提取模板组件
  
- 使用指令 `@apply` 提取组件类

```css
.btn-indigo {
    @apply py-2 px-4 bg-indigo-500 text-white font-semibold rounded-lg shadow-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-400 focus:ring-opacity-75;
  }
```

使用指令 `@layer components { ... }` 告诉 Tailwind 定义的样式属于哪个布局

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .btn-blue {
    @apply py-2 px-4 bg-blue-500 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-400 focus:ring-opacity-75;
  }
}
```

### 自定义 utilities

使用 css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  .scroll-snap-none {
    scroll-snap-type: none;
  }
  .scroll-snap-x {
    scroll-snap-type: x;
  }
  .scroll-snap-y {
    scroll-snap-type: y;
  }
}
```

生成响应式变体

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  @variants responsive {
    .scroll-snap-none {
      scroll-snap-type: none;
    }
    .scroll-snap-x {
      scroll-snap-type: x;
    }
    .scroll-snap-y {
      scroll-snap-type: y;
    }
  }
}
```

生成暗模式变体

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  @variants dark {
    .filter-none {
      filter: none;
    }
    .filter-grayscale {
      filter: grayscale(100%);
    }
  }
}
```

生成状态变体

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  @variants hover, focus {
    .filter-none {
      filter: none;
    }
    .filter-grayscale {
      filter: grayscale(100%);
    }
  }
}
```

### 功能和指令

参考官方文档：[https://tailwindcss.com/docs/functions-and-directives](https://tailwindcss.com/docs/functions-and-directives)

## 布局

### 容器

`container` 是用于将元素的宽度固定到当前响应式断点的组件。

<warning>

> 默认不是水平居中的，也没有任何填充。  
> 可以使用 `mx-auto` `px-{size}` 设置其水平居中和填充。

</warning>

### 盒子模型

用于控制浏览器如何计算元素的尺寸

| Class  | Properties  |
| ------------ | ------------ |
| `box-border`  | `box-sizing: border-box;`  |
| `box-content`  | `box-sizing: content-box;`  |

### Display

| Class  | Properties  |
| ------------ | ------------ |
| block  | display: block;  |
| inline-block  | display: inline-block;  |
| inline  | display: inline;  |
| flex  | display: flex;  |
| inline-flex  | display: inline-flex;  |
| table  | display: table;  |
| table-caption  | display: table-caption;  |
| table-cell  | display: table-cell;  |
| table-column  | display: table-column;  |
| table-column-group  | display: table-column-group;  |
| table-footer-group  | display: table-footer-group;  |
| table-header-group  | display: table-header-group;  |
| table-row-group  | display: table-row-group;  |
| table-row  | display: table-row;  |
| flow-root  | display: flow-root;  |
| grid  | display: grid;  |
| inline-grid  | display: inline-grid;  |
| contents  | display: contents;  |
| hidden  | display: none;  |

具体用法参考官方文档：[https://tailwindcss.com/docs/display](https://tailwindcss.com/docs/display)

### 浮动

| Class  | Properties  |
| ------------ | ------------ |
| float-right  | float: right;  |
| float-left  | float: left;  |
| float-none  | float: none;  |

### 清除浮动

| Class  | Properties  |
| ------------ | ------------ |
| clear-left  | clear: left;  |
| clear-right  | clear: right;  |
| clear-both  | clear: both;  |
| clear-none  | clear: none;  |

### 对象大小

用于控制如何调整被替换元素内容的大小

| Class  | Properties  |
| ------------ | ------------ |
| object-contain  | object-fit: contain;  |
| object-cover  | object-fit: cover;  |
| object-fill  | object-fit: fill;  |
| object-none  | object-fit: none;
| object-scale-down  | object-fit: scale-down;  |


### 对象位置

用于控制替换元素的内容应如何放置在其容器中

| Class  | Properties  |
| ------------ | ------------ |
| object-bottom  | object-position: bottom;  |
| object-center  | object-position: center;  |
| object-left  | object-position: left;  |
| object-left-bottom  | object-position: left bottom;  |
| object-left-top  | object-position: left top;  |
| object-right  | object-position: right;  |
| object-right-bottom  | object-position: right bottom;  |
| object-right-top  | object-position: right top;  |
| object-top  | object-position: top;  |

### 溢出

| Class Properties  |
| ------------ | ------------ |
| overflow-auto  | overflow: auto;  |
| overflow-hidden  | overflow: hidden;  |
| overflow-visible  | overflow: visible;  |
| overflow-scroll  | overflow: scroll;  |
| overflow-x-auto  | overflow-x: auto;  |
| overflow-y-auto  | overflow-y: auto;  |
| overflow-x-hidden  | overflow-x: hidden;  |
| overflow-y-hidden  | overflow-y: hidden;  |
| overflow-x-visible  | overflow-x: visible;  |
| overflow-y-visible  | overflow-y: visible;  |
| overflow-x-scroll  | overflow-x: scroll;  |
| overflow-y-scroll  | overflow-y: scroll;  |

### 滚动行为

用于控制浏览器到达滚动区域边界时的行为

| Class  | Properties  |
| ------------ | ------------ |
| `overscroll-auto`  | `overscroll-behavior: auto;`  |
| `overscroll-contain`  | `overscroll-behavior: contain;`  |
| `overscroll-none`  | `overscroll-behavior: none;`  |
| `overscroll-y-auto`  | `overscroll-behavior-y: auto;`  |
| `overscroll-y-contain`  | `overscroll-behavior-y: contain;`  |
| `overscroll-y-none`  | `overscroll-behavior-y: none;`  |
| `overscroll-x-auto`  | `overscroll-behavior-x: auto;`  |
| `overscroll-x-contain`  | `overscroll-behavior-x: contain;`  |
| `overscroll-x-none`  | `overscroll-behavior-x: none;`  |

- auto - 用于使用户在到达主滚动区域的边界时能够继续滚动父滚动区域
- contain - 用于防止目标区域中的滚动触发父元素中的滚动，但在支持该控件的操作系统中滚动通过容器的末尾时保留"弹跳"效果
- none - 用于防止目标区域中的滚动触发父元素中的滚动，并防止滚动超过容器末尾时出现"反弹"效果

### 定位

| Class  | Properties  |
| ------------ | ------------ |
| `static`  | `position: static;`  |
| `fixed`  | `position: fixed;`  |
| `absolute`  | `position: absolute;`  |
| `relative`  | `position: relative;`  |
| `sticky`  | `position: sticky;`  |

### z-index

`z-{index}`

## Flex Box

### flex direction

用于控制 flex items 的方向

| Class  | Properties  |
| ------------ | ------------ |
| `flex-row`  | `flex-direction: row;`  |
| `flex-row-reverse`  | `flex-direction: row-reverse;`  |
| `flex-col`  | `flex-direction: column;`  |
| `flex-col-reverse`  | `flex-direction: column-reverse;`  |


### flex wrap

用于控制 flex items 的换行

| Class  | Properties  |
| ------------ | ------------ |
| `flex-wrap`  | `flex-wrap: wrap;`  |
| `flex-wrap-reverse`  | `flex-wrap: wrap-reverse;`  |
| `flex-nowrap`  | `flex-wrap: nowrap;`  |


### flex

用于控制 flex items 的扩张和收缩：`flex: grow shirnk basis`

| Class  | Properties  |
| ------------ | ------------ |
| `flex-1`  | `flex: 1 1 0%;`  |
| `flex-auto`  | `flex: 1 1 auto;`  |
| `flex-initial`  | `flex: 0 1 auto;`  |
| `flex-none`  | `flex: none;`  |

- initial - 允许收缩，但不扩张
- flex-1 - 忽略初始尺寸，允许 flex item 根据需要进行扩张和收缩
- auto - 允许 flex item 根据其初始大小进行扩张和收缩
- none - 阻止 flex item 扩张和收缩

### flex grow

| Class  | Properties  | description  |
| ------------ | ------------ | ------------ |
| `flex-grow-0`  | `flex-grow: 0;`  | 阻止 flex item 扩张  |
| `flex-grow`  | `flex-grow: 1;`  | 允许 flex item 扩张以填充任何可用的空间  |

### flex shrink

| Class  | Properties  | description  |
| ------------ | ------------ | ------------ |
| `flex-shrink-0`  | `flex-shrink: 0;`  | 阻止 flex item 收缩  |
| `flex-shrink`  | `flex-shrink: 1;`  | 允许 flex item 根据需要进行收缩  |

### order

控制 flex 和 grid 的 items 的排序

| Class  | Properties  |
| ------------ | ------------ |
| `order-1`  | `order: 1;`  |
| `order-2`  | `order: 2;`  |
| `order-3`  | `order: 3;`  |
| `order-4`  | `order: 4;`  |
| `order-5`  | `order: 5;`  |
| `order-6`  | `order: 6;`  |
| `order-7`  | `order: 7;`  |
| `order-8`  | `order: 8;`  |
| `order-9`  | `order: 9;`  |
| `order-10`  | `order: 10;`  |
| `order-11`  | `order: 11;`  |
| `order-12`  | `order: 12;`  |
| `order-first`  | `order: -9999;`  |
| `order-last`  | `order: 9999;`  |
| `order-none`  | `order: 0;`  |

## Grid

### grid template columns

指定网格布局中的列。

使用 `grid-cols-{n}` 创建具有 `n` 个大小相等的列的网格。

| Class  | Properties  |
| ------------ | ------------ |
| `grid-cols-1`  | `grid-template-columns: repeat(1, minmax(0, 1fr));`  |
| `grid-cols-2`  | `grid-template-columns: repeat(2, minmax(0, 1fr));`  |
| `grid-cols-3`  | `grid-template-columns: repeat(3, minmax(0, 1fr));`  |
| `grid-cols-4`  | `grid-template-columns: repeat(4, minmax(0, 1fr));`  |
| `grid-cols-5`  | `grid-template-columns: repeat(5, minmax(0, 1fr));`  |
| `grid-cols-6`  | `grid-template-columns: repeat(6, minmax(0, 1fr));`  |
| `grid-cols-7`  | `grid-template-columns: repeat(7, minmax(0, 1fr));`  |
| `grid-cols-8`  | `grid-template-columns: repeat(8, minmax(0, 1fr));`  |
| `grid-cols-9`  | `grid-template-columns: repeat(9, minmax(0, 1fr));`  |
| `grid-cols-10`  | `grid-template-columns: repeat(10, minmax(0, 1fr));`  |
| `grid-cols-11`  | `grid-template-columns: repeat(11, minmax(0, 1fr));`  |
| `grid-cols-12`  | `grid-template-columns: repeat(12, minmax(0, 1fr));`  |
| `grid-cols-none`  | `grid-template-columns: none;`  |


```css
<div class="grid grid-cols-3 gap-4">
  <div>1</div>
  <!-- ... -->
  <div>9</div>
</div>
```

### grid column

| Class  | Properties  |
| ------------ | ------------ |
| `col-auto`  | `grid-column: auto;`  |
| `col-span-1`  | `grid-column: span 1 / span 1;`  |
| `col-span-2`  | `grid-column: span 2 / span 2;`  |
| `col-span-3`  | `grid-column: span 3 / span 3;`  |
| `col-span-4`  | `grid-column: span 4 / span 4;`  |
| `col-span-5`  | `grid-column: span 5 / span 5;`  |
| `col-span-6`  | `grid-column: span 6 / span 6;`  |
| `col-span-7`  | `grid-column: span 7 / span 7;`  |
| `col-span-8`  | `grid-column: span 8 / span 8;`  |
| `col-span-9`  | `grid-column: span 9 / span 9;`  |
| `col-span-10`  | `grid-column: span 10 / span 10;`  |
| `col-span-11`  | `grid-column: span 11 / span 11;`  |
| `col-span-12`  | `grid-column: span 12 / span 12;`  |
| `col-span-full`  | `grid-column: 1 / -1;`  |
| `col-start-1`  | `grid-column-start: 1;`  |
| `col-start-2`  | `grid-column-start: 2;`  |
| `col-start-3`  | `grid-column-start: 3;`  |
| `col-start-4`  | `grid-column-start: 4;`  |
| `col-start-5`  | `grid-column-start: 5;`  |
| `col-start-6`  | `grid-column-start: 6;`  |
| `col-start-7`  | `grid-column-start: 7;`  |
| `col-start-8`  | `grid-column-start: 8;`  |
| `col-start-9`  | `grid-column-start: 9;`  |
| `col-start-10`  | `grid-column-start: 10;`  |
| `col-start-11`  | `grid-column-start: 11;`  |
| `col-start-12`  | `grid-column-start: 12;`  |
| `col-start-13`  | `grid-column-start: 13;`  |
| `col-start-auto`  | `grid-column-start: auto;`  |
| `col-end-1`  | `grid-column-end: 1;`  |
| `col-end-2`  | `grid-column-end: 2;`  |
| `col-end-3`  | `grid-column-end: 3;`  |
| `col-end-4`  | `grid-column-end: 4;`  |
| `col-end-5`  | `grid-column-end: 5;`  |
| `col-end-6`  | `grid-column-end: 6;`  |
| `col-end-7`  | `grid-column-end: 7;`  |
| `col-end-8`  | `grid-column-end: 8;`  |
| `col-end-9`  | `grid-column-end: 9;`  |
| `col-end-10`  | `grid-column-end: 10;`  |
| `col-end-11`  | `grid-column-end: 11;`  |
| `col-end-12`  | `grid-column-end: 12;`  |
| `col-end-13`  | `grid-column-end: 13;`  |
| `col-end-auto`  | `grid-column-end: auto;`  |


参考官方文档：[https://tailwindcss.com/docs/grid-column](https://tailwindcss.com/docs/grid-column)


## 对齐

### justify content

控制整个内容区域在网格容器中的水平位置（左中右）

| Class  | Properties  |
| ------------ | ------------ |
| `justify-start`  | `justify-content: flex-start;`  |
| `justify-end`  | `justify-content: flex-end;`  |
| `justify-center`  | `justify-content: center;`  |
| `justify-between`  | `justify-content: space-between;`  |
| `justify-around`  | `justify-content: space-around;`  |
| `justify-evenly`  | `justify-content: space-evenly;`  |

### justify items

控制网格单元格内容的水平位置（左中右）

| Class  | Properties  |
| ------------ | ------------ |
| `justify-items-auto`  | `justify-items: auto;`  |
| `justify-items-start`  | `justify-items: start;`  |
| `justify-items-end`  | `justify-items: end;`  |
| `justify-items-center`  | `justify-items: center;`  |
| `justify-items-stretch`  | `justify-items: stretch;`  |

### justify self

控制单个网格项如何沿其内联轴对齐

| Class  | Properties  |
| ------------ | ------------ |
| `justify-self-auto`  | `justify-self: auto;`  |
| `justify-self-start`  | `justify-self: start;`  |
| `justify-self-end`  | `justify-self: end;`  |
| `justify-self-center`  | `justify-self: center;`  |
| `justify-self-stretch`  | `justify-self: stretch;`  |

### align content

控制整个内容区域在网格容器中的垂直位置（上中下）

| Class  | Properties  |
| ------------ | ------------ |
| `content-center`  | `align-content: center;`  |
| `content-start`  | `align-content: flex-start;`  |
| `content-end`  | `align-content: flex-end;`  |
| `content-between`  | `align-content: space-between;`  |
| `content-around`  | `align-content: space-around;`  |
| `content-evenly`  | `align-content: space-evenly;`  |

### align items

控制网格单元格内容的垂直位置（上中下）

| Class  | Properties  |
| ------------ | ------------ |
| `items-start`  | `align-items: flex-start;`  |
| `items-end`  | `align-items: flex-end;`  |
| `items-center`  | `align-items: center;`  |
| `items-baseline`  | `align-items: baseline;`  |
| `items-stretch`  | `align-items: stretch;`  |

### align self

控制单个网格项如何沿其内联轴对齐

| Class  | Properties  |
| ------------ | ------------ |
| `self-auto`  | `align-self: auto;`  |
| `self-start`  | `align-self: flex-start;`  |
| `self-end`  | `align-self: flex-end;`  |
| `self-center`  | `align-self: center;`  |
| `self-stretch`  | `align-self: stretch;`  |

### place content

| Class  | Properties  |
| ------------ | ------------ |
| `place-content-center`  | `place-content: center;`  |
| `place-content-start`  | `place-content: start;`  |
| `place-content-end`  | `place-content: end;`  |
| `place-content-between`  | `place-content: space-between;`  |
| `place-content-around`  | `place-content: space-around;`  |
| `place-content-evenly`  | `place-content: space-evenly;`  |
| `place-content-stretch`  | `place-content: stretch;`  |

### place items

| Class  | Properties  |
| ------------ | ------------ |
| `place-items-auto`  | `place-items: auto;`  |
| `place-items-start`  | `place-items: start;`  |
| `place-items-end`  | `place-items: end;`  |
| `place-items-center`  | `place-items: center;`  |
| `place-items-stretch`  | `place-items: stretch;`  |

### place self

| Class  | Properties  |
| ------------ | ------------ |
| `place-self-auto`  | `place-self: auto;`  |
| `place-self-start`  | `place-self: start;`  |
| `place-self-end`  | `place-self: end;`  |
| `place-self-center`  | `place-self: center;`  |
| `place-self-stretch`  | `place-self: stretch;`  |

## 间距

### padding

`p{t|r|b|l}-{size}`

### margin

`m{t|r|b|l}-{size}`

### space

- `space-x-{amount}`
- `space-y-{amount}`

## 大小

### width

| Class  | Properties  |
| ------------ | ------------ |
| `w-0`  | `width: 0px;`  |
| `w-0.5`  | `width: 0.125rem;`  |
| `w-1`  | `width: 0.25rem;`  |
| `w-1.5`  | `width: 0.375rem;`  |
| `w-2`  | `width: 0.5rem;`  |
| `w-2.5`  | `width: 0.625rem;`  |
| `w-3`  | `width: 0.75rem;`  |
| `w-3.5`  | `width: 0.875rem;`  |
| `w-4`  | `width: 1rem;`  |
| `w-5`  | `width: 1.25rem;`  |
| `w-6`  | `width: 1.5rem;`  |
| `w-7`  | `width: 1.75rem;`  |
| `w-8`  | `width: 2rem;`  |
| `w-9`  | `width: 2.25rem;`  |
| `w-10`  | `width: 2.5rem;`  |
| `w-11`  | `width: 2.75rem;`  |
| `w-12`  | `width: 3rem;`  |
| `w-14`  | `width: 3.5rem;`  |
| `w-16`  | `width: 4rem;`  |
| `w-20`  | `width: 5rem;`  |
| `w-24`  | `width: 6rem;`  |
| `w-28`  | `width: 7rem;`  |
| `w-32`  | `width: 8rem;`  |
| `w-36`  | `width: 9rem;`  |
| `w-40`  | `width: 10rem;`  |
| `w-44`  | `width: 11rem;`  |
| `w-48`  | `width: 12rem;`  |
| `w-52`  | `width: 13rem;`  |
| `w-56`  | `width: 14rem;`  |
| `w-60`  | `width: 15rem;`  |
| `w-64`  | `width: 16rem;`  |
| `w-72`  | `width: 18rem;`  |
| `w-80`  | `width: 20rem;`  |
| `w-96`  | `width: 24rem;`  |
| `w-auto`  | `width: auto;`  |
| `w-px`  | `width: 1px;`  |
| `w-1/2`  | `width: 50%;`  |
| `w-1/3`  | `width: 33.333333%;`  |
| `w-2/3`  | `width: 66.666667%;`  |
| `w-1/4`  | `width: 25%;`  |
| `w-2/4`  | `width: 50%;`  |
| `w-3/4`  | `width: 75%;`  |
| `w-1/5`  | `width: 20%;`  |
| `w-2/5`  | `width: 40%;`  |
| `w-3/5`  | `width: 60%;`  |
| `w-4/5`  | `width: 80%;`  |
| `w-1/6`  | `width: 16.666667%;`  |
| `w-2/6`  | `width: 33.333333%;`  |
| `w-3/6`  | `width: 50%;`  |
| `w-4/6`  | `width: 66.666667%;`  |
| `w-5/6`  | `width: 83.333333%;`  |
| `w-1/12`  | `width: 8.333333%;`  |
| `w-2/12`  | `width: 16.666667%;`  |
| `w-3/12`  | `width: 25%;`  |
| `w-4/12`  | `width: 33.333333%;`  |
| `w-5/12`  | `width: 41.666667%;`  |
| `w-6/12`  | `width: 50%;`  |
| `w-7/12`  | `width: 58.333333%;`  |
| `w-8/12`  | `width: 66.666667%;`  |
| `w-9/12`  | `width: 75%;`  |
| `w-10/12`  | `width: 83.333333%;`  |
| `w-11/12`  | `width: 91.666667%;`  |
| `w-full`  | `width: 100%;`  |
| `w-screen`  | `width: 100vw;`  |
| `w-min`  | `width: min-content;`  |
| `w-max`  | `width: max-content;`  |

### min-width

| Class  | Properties  |
| ------------ | ------------ |
| `min-w-0`  | `min-width: 0px;`  |
| `min-w-full`  | `min-width: 100%;`  |
| `min-w-min`  | `min-width: min-content;`  |
| `min-w-max`  | `min-width: max-content;`  |

### max-width

| Class  | Properties  |
| ------------ | ------------ |
| `max-w-0`  | `max-width: 0rem;`  |
| `max-w-none`  | `max-width: none;`  |
| `max-w-xs`  | `max-width: 20rem;`  |
| `max-w-sm`  | `max-width: 24rem;`  |
| `max-w-md`  | `max-width: 28rem;`  |
| `max-w-lg`  | `max-width: 32rem;`  |
| `max-w-xl`  | `max-width: 36rem;`  |
| `max-w-2xl`  | `max-width: 42rem;`  |
| `max-w-3xl`  | `max-width: 48rem;`  |
| `max-w-4xl`  | `max-width: 56rem;`  |
| `max-w-5xl`  | `max-width: 64rem;`  |
| `max-w-6xl`  | `max-width: 72rem;`  |
| `max-w-7xl`  | `max-width: 80rem;`  |
| `max-w-full`  | `max-width: 100%;`  |
| `max-w-min`  | `max-width: min-content;`  |
| `max-w-max`  | `max-width: max-content;`  |
| `max-w-prose`  | `max-width: 65ch;`  |
| `max-w-screen-sm`  | `max-width: 640px;`  |
| `max-w-screen-md`  | `max-width: 768px;`  |
| `max-w-screen-lg`  | `max-width: 1024px;`  |
| `max-w-screen-xl`  | `max-width: 1280px;`  |
| `max-w-screen-2xl`  | `max-width: 1536px;`  |

### height

| Class  | Properties  |
| ------------ | ------------ |
| `h-0`  | `height: 0px;`  |
| `h-0.5`  | `height: 0.125rem;`  |
| `h-1`  | `height: 0.25rem;`  |
| `h-1.5`  | `height: 0.375rem;`  |
| `h-2`  | `height: 0.5rem;`  |
| `h-2.5`  | `height: 0.625rem;`  |
| `h-3`  | `height: 0.75rem;`  |
| `h-3.5`  | `height: 0.875rem;`  |
| `h-4`  | `height: 1rem;`  |
| `h-5`  | `height: 1.25rem;`  |
| `h-6`  | `height: 1.5rem;`  |
| `h-7`  | `height: 1.75rem;`  |
| `h-8`  | `height: 2rem;`  |
| `h-9`  | `height: 2.25rem;`  |
| `h-10`  | `height: 2.5rem;`  |
| `h-11`  | `height: 2.75rem;`  |
| `h-12`  | `height: 3rem;`  |
| `h-14`  | `height: 3.5rem;`  |
| `h-16`  | `height: 4rem;`  |
| `h-20`  | `height: 5rem;`  |
| `h-24`  | `height: 6rem;`  |
| `h-28`  | `height: 7rem;`  |
| `h-32`  | `height: 8rem;`  |
| `h-36`  | `height: 9rem;`  |
| `h-40`  | `height: 10rem;`  |
| `h-44`  | `height: 11rem;`  |
| `h-48`  | `height: 12rem;`  |
| `h-52`  | `height: 13rem;`  |
| `h-56`  | `height: 14rem;`  |
| `h-60`  | `height: 15rem;`  |
| `h-64`  | `height: 16rem;`  |
| `h-72`  | `height: 18rem;`  |
| `h-80`  | `height: 20rem;`  |
| `h-96`  | `height: 24rem;`  |
| `h-auto`  | `height: auto;`  |
| `h-px`  | `height: 1px;`  |
| `h-1/2`  | `height: 50%;`  |
| `h-1/3`  | `height: 33.333333%;`  |
| `h-2/3`  | `height: 66.666667%;`  |
| `h-1/4`  | `height: 25%;`  |
| `h-2/4`  | `height: 50%;`  |
| `h-3/4`  | `height: 75%;`  |
| `h-1/5`  | `height: 20%;`  |
| `h-2/5`  | `height: 40%;`  |
| `h-3/5`  | `height: 60%;`  |
| `h-4/5`  | `height: 80%;`  |
| `h-1/6`  | `height: 16.666667%;`  |
| `h-2/6`  | `height: 33.333333%;`  |
| `h-3/6`  | `height: 50%;`  |
| `h-4/6`  | `height: 66.666667%;`  |
| `h-5/6`  | `height: 83.333333%;`  |
| `h-full`  | `height: 100%;`  |
| `h-screen`  | `height: 100vh;`  |

### min-height

| Class  | Properties  |
| ------------ | ------------ |
| `min-h-0`  | `min-height: 0px;`  |
| `min-h-full`  | `min-height: 100%;`  |
| `min-h-screen`  | `min-height: 100vh;`  |

### max-height

| Class  | Properties  |
| ------------ | ------------ |
| `max-h-0`  | `max-height: 0px;`  |
| `max-h-0.5`  | `max-height: 0.125rem;`  |
| `max-h-1`  | `max-height: 0.25rem;`  |
| `max-h-1.5`  | `max-height: 0.375rem;`  |
| `max-h-2`  | `max-height: 0.5rem;`  |
| `max-h-2.5`  | `max-height: 0.625rem;`  |
| `max-h-3`  | `max-height: 0.75rem;`  |
| `max-h-3.5`  | `max-height: 0.875rem;`  |
| `max-h-4`  | `max-height: 1rem;`  |
| `max-h-5`  | `max-height: 1.25rem;`  |
| `max-h-6`  | `max-height: 1.5rem;`  |
| `max-h-7`  | `max-height: 1.75rem;`  |
| `max-h-8`  | `max-height: 2rem;`  |
| `max-h-9`  | `max-height: 2.25rem;`  |
| `max-h-10`  | `max-height: 2.5rem;`  |
| `max-h-11`  | `max-height: 2.75rem;`  |
| `max-h-12`  | `max-height: 3rem;`  |
| `max-h-14`  | `max-height: 3.5rem;`  |
| `max-h-16`  | `max-height: 4rem;`  |
| `max-h-20`  | `max-height: 5rem;`  |
| `max-h-24`  | `max-height: 6rem;`  |
| `max-h-28`  | `max-height: 7rem;`  |
| `max-h-32`  | `max-height: 8rem;`  |
| `max-h-36`  | `max-height: 9rem;`  |
| `max-h-40`  | `max-height: 10rem;`  |
| `max-h-44`  | `max-height: 11rem;`  |
| `max-h-48`  | `max-height: 12rem;`  |
| `max-h-52`  | `max-height: 13rem;`  |
| `max-h-56`  | `max-height: 14rem;`  |
| `max-h-60`  | `max-height: 15rem;`  |
| `max-h-64`  | `max-height: 16rem;`  |
| `max-h-72`  | `max-height: 18rem;`  |
| `max-h-80`  | `max-height: 20rem;`  |
| `max-h-96`  | `max-height: 24rem;`  |
| `max-h-px`  | `max-height: 1px;`  |
| `max-h-full`  | `max-height: 100%;`  |
| `max-h-screen`  | `max-height: 100vh;`  |


## 排版

### 字体

- `font-sans`
- `font-serif`
- `font-mono`

### font size

`text-{size}`

### 平滑

| Class  | Description  |
| ------------ | ------------ |
| `antialiased`  | 使用灰度抗锯齿  |
| `subpixel-antialiased`  | 使用子像素抗锯齿  |

### font style

| Class  | Properties  |
| ------------ | ------------ |
| `italic`  | `font-style: italic;`  |
| `not-italic`  | `font-style: normal;`  |

### font weight

| Class  | Properties  |
| ------------ | ------------ |
| `font-thin`  | `font-weight: 100;`  |
| `font-extralight`  | `font-weight: 200;`  |
| `font-light`  | `font-weight: 300;`  |
| `font-normal`  | `font-weight: 400;`  |
| `font-medium`  | `font-weight: 500;`  |
| `font-semibold`  | `font-weight: 600;`  |
| `font-bold`  | `font-weight: 700;`  |
| `font-extrabold`  | `font-weight: 800;`  |
| `font-black`  | `font-weight: 900;`  |

### font variant numeric

参考官方文档：[https://tailwindcss.com/docs/font-variant-numeric](https://tailwindcss.com/docs/font-variant-numeric)

### letter spacing

| Class  | Properties  |
| ------------ | ------------ |
| `tracking-tighter`  | `letter-spacing: -0.05em;`  |
| `tracking-tight`  | `letter-spacing: -0.025em;`  |
| `tracking-normal`  | `letter-spacing: 0em;`  |
| `tracking-wide`  | `letter-spacing: 0.025em;`  |
| `tracking-wider`  | `letter-spacing: 0.05em;`  |
| `tracking-widest`  | `letter-spacing: 0.1em;`  |


### line height

| Class  | Properties  |
| ------------ | ------------ |
| `leading-3`  | `line-height: .75rem;`  |
| `leading-4`  | `line-height: 1rem;`  |
| `leading-5`  | `line-height: 1.25rem;`  |
| `leading-6`  | `line-height: 1.5rem;`  |
| `leading-7`  | `line-height: 1.75rem;`  |
| `leading-8`  | `line-height: 2rem;`  |
| `leading-9`  | `line-height: 2.25rem;`  |
| `leading-10`  | `line-height: 2.5rem;`  |
| `leading-none`  | `line-height: 1;`  |
| `leading-tight`  | `line-height: 1.25;`  |
| `leading-snug`  | `line-height: 1.375;`  |
| `leading-normal`  | `line-height: 1.5;`  |
| `leading-relaxed`  | `line-height: 1.625;`  |
| `leading-loose`  | `line-height: 2;`  |

### list style type

| Class  | Properties  |
| ------------ | ------------ |
| `list-none`  | `list-style-type: none;`  |
| `list-disc`  | `list-style-type: disc;`  |
| `list-decimal`  | `list-style-type: decimal;`  |

### list style position

| Class  | Properties  |
| ------------ | ------------ |
| `list-inside`  | `list-style-position: inside;`  |
| `list-outside`  | `list-style-position: outside;`  |


### placeholder color

`placeholder-{color}` 控制占位文本的颜色

### placeholder opacity

`placeholder-opacity-{amount}` 控制占位文本的透明度

### text align

| Class  | Properties  |
| ------------ | ------------ |
| `text-left`  | `text-align: left;`  |
| `text-center`  | `text-align: center;`  |
| `text-right`  | `text-align: right;`  |
| `text-justify`  | `text-align: justify;`  |

### text color

`text-{color}` 控制文本的颜色


### text opacity

`text-opacity-{amount}` 控制文本的透明度


### text decoration

文本装饰器

| Class  | Properties  | Description |
| ------------ | ------------ | ------------ |
| `underline`  | `text-decoration: underline;`  | 下划线 |
| `line-through`  | `text-decoration: line-through;`  | 中划线 |
| `no-underline`  | `text-decoration: none;`  | 无 |

### text transform

文本变化

| Class  | Properties  | Description |
| ------------ | ------------ | ------------ |
| `uppercase`  | `text-transform: uppercase;`  | 转大写 |
| `lowercase`  | `text-transform: lowercase;`  | 转小写 |
| `capitalize`  | `text-transform: capitalize;`  | 首字母大写 |
| `normal-case`  | `text-transform: none;`  | 正常 |


### text overflow

| Class  | Properties  | Description |
| ------------ | ------------ | ------------ |
| `truncate`  | `overflow: hidden;` <br/> `text-overflow: ellipsis;` <br/> `white-space: nowrap;`  | 用 `…` 来截断溢出的文本 |
| `overflow-ellipsis`  | `text-overflow: ellipsis;`  | 用 `…` 来截断溢出的文本 |
| `overflow-clip`  | `text-overflow: clip;`  | 截断溢出的文本 |

### vertical align

| Class  | Properties  |
| ------------ | ------------ |
| `align-baseline`  | `vertical-align: baseline;`  |
| `align-top`  | `vertical-align: top;`  |
| `align-middle`  | `vertical-align: middle;`  |
| `align-bottom`  | `vertical-align: bottom;`  |
| `align-text-top`  | `vertical-align: text-top;`  |
| `align-text-bottom`  | `vertical-align: text-bottom;`  |


### whitespace

| Class  | Properties  |
| ------------ | ------------ |
| `whitespace-normal`  | `white-space: normal;`  |
| `whitespace-nowrap`  | `white-space: nowrap;`  |
| `whitespace-pre`  | `white-space: pre;`  |
| `whitespace-pre-line`  | `white-space: pre-line;`  |
| `whitespace-pre-wrap`  | `white-space: pre-wrap;`  |


| 值  | 描述  |
| ------------ | ------------ |
| `normal`  | 默认。空白会被浏览器忽略。  |
| `pre`  | 空白会被浏览器保留。其行为方式类似 HTML 中的 `<pre>` 标签。  |
| `nowrap`  | 文本不会换行，文本会在在同一行上继续，直到遇到 `<br>` 标签为止。  |
| `pre-wrap`  | 保留空白符序列，但是正常地进行换行。  |
| `pre-line`  | 合并空白符序列，但是保留换行符。  |
| `inherit`  | 规定应该从父元素继承 white-space 属性的值。  |


### word break

| Class  | Properties  |
| ------------ | ------------ |
| `break-normal`  | `overflow-wrap: normal;` <br/> `word-break: normal;`  |
| `break-words`  | `overflow-wrap: break-word;`  |
| `break-all`  | `word-break: break-all;`  |

## backgrounds

### background attachment

| Class  | Properties  |
| ------------ | ------------ |
| `bg-fixed`  | `background-attachment: fixed;`  |
| `bg-local`  | `background-attachment: local;`  |
| `bg-scroll`  | `background-attachment: scroll;`  |

### background clip

控制背景的绘制区域

| Class  | Properties  |
| ------------ | ------------ |
| `bg-clip-border`  | `background-clip: border-box;`  |
| `bg-clip-padding`  | `background-clip: padding-box;`  |
| `bg-clip-content`  | `background-clip: content-box;`  |
| `bg-clip-text`  | `background-clip: text;`  |

### background color

`bg-{color}` 控制元素的背景颜色

### background opacity

`bg-opacity-{amount}` 控制元素的背景透明度


### background position

控制元素背景的位置

| Class  | Properties  |
| ------------ | ------------ |
| `bg-bottom`  | `background-position: bottom;`  |
| `bg-center`  | `background-position: center;`  |
| `bg-left`  | `background-position: left;`  |
| `bg-left-bottom`  | `background-position: left bottom;`  |
| `bg-left-top`  | `background-position: left top;`  |
| `bg-right`  | `background-position: right;`  |
| `bg-right-bottom`  | `background-position: right bottom;`  |
| `bg-right-top`  | `background-position: right top;`  |
| `bg-top`  | `background-position: top;`  |

### background repeat

| Class  | Properties  |
| ------------ | ------------ |
| `bg-repeat`  | `background-repeat: repeat;`  |
| `bg-no-repeat`  | `background-repeat: no-repeat;`  |
| `bg-repeat-x`  | `background-repeat: repeat-x;`  |
| `bg-repeat-y`  | `background-repeat: repeat-y;`  |
| `bg-repeat-round`  | `background-repeat: round;`  |
| `bg-repeat-space`  | `background-repeat: space;`  |

### background size

| Class  | Properties  |
| ------------ | ------------ |
| `bg-auto`  | `background-size: auto;`  |
| `bg-cover`  | `background-size: cover;`  |
| `bg-contain`  | `background-size: contain;`  |

### background image

- `bg-gradient-{direction}` - 背景颜色线性渐变

### gradient color stops

- `from-{color}` - 起始渐变色
- `to-{color}` - 结束渐变色
- `via-{color}` - 中间颜色

## Border

### border radius

- `rounded-{t|r|b|l}{-size?}` - 控制 border 的圆角
- `rounded-sm` `rounded-lg` `rounded` - 控制 border 的圆角
- `rounded-full` - 药丸型或者圆圈
- `rounded-none` - 无圆角

### border width

- `border-{amount}` - 控制所有 border 的宽度
- `border-{side}` `border-{side}-{amount}` -  - 控制指定方向 border 的宽度

### border color

`border-{color}` 控制 border 颜色

### border opacity

`border-opacity-{amount}` 控制 border 的透明度

### border style

控制 border 的样式

| Class  | Properties  |
| ------------ | ------------ |
| `border-solid`  | `border-style: solid;`  |
| `border-dashed`  | `border-style: dashed;`  |
| `border-dotted`  | `border-style: dotted;`  |
| `border-double`  | `border-style: double;`  |
| `border-none`  | `border-style: none;`  |

### divide width

`divide-{x|y}-{amount}` 控制 x、y 轴分割线的宽度

### divide color

`divide-{color}` 控制分割线的颜色

### divide opacity

`divide-opacity-{amount}` 控制分割线的透明度

### divide style

`divide-{style}` 控制分割线的样式

### ring width

`ring-{width}` 为元素创建一个带阴影的轮廓

### ring color

`ring-{color}` 为元素阴影轮廓定义颜色

### ring opacity

`ring-opacity-{amount}` 控制阴影轮廓的透明度

### ring offset width

`ring-offset-{width}` 控制轮廓边距的宽度

### ring offset color

`ring-offset-{color}` 控制轮廓边距的颜色

## 效果

### box shadow

`shadow-{sm|md|lg|xl|2xl|inner|none}` 控制元素的阴影

### opacity

`opacity-{amount}` 控制元素的透明度


## 表格

### border collapse

控制表格边框的样式

| Class  | Properties  |
| ------------ | ------------ |
| `border-collapse`  | `border-collapse: collapse;`  |
| `border-separate`  | `border-collapse: separate;`  |

### table layout

控制表格内容的布局

| Class  | Properties  |
| ------------ | ------------ |
| `table-auto`  | `table-layout: auto;`  |
| `table-fixed`  | `table-layout: fixed;`  |

## 过度、动画

### transition property

`transition-{properties}` 控制元素指定的属性改变时存在过度动画

### transition duration

`duration-{amount}` 控制过度动画持续的时间

### transition timing function

控制过度动画的时间函数

| Class  | Properties  |
| ------------ | ------------ |
| `ease-linear`  | `transition-timing-function: linear;`  |
| `ease-in`  | `transition-timing-function: cubic-bezier(0.4, 0, 1, 1);`  |
| `ease-out`  | `transition-timing-function: cubic-bezier(0, 0, 0.2, 1);`  |
| `ease-in-out`  | `transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);`  |

### transition delay

`delay-{amount}` 控制过度动画的延迟时间

### animation

- `animate-spin` - 向元素添加一个线性的旋转动画

```css
<button type="button" class="bg-rose-600 ... " disabled>
  <svg class="animate-spin h-5 w-5 mr-3 ... " viewBox="0 0 24 24">
    <!-- ... -->
  </svg>
  Processing
</button>
```

- `animate-ping` - 使元素缩放、褪色像雷达波或者波纹一样，一般用于徽章

```css
<span class="flex h-3 w-3">
  <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-purple-400 opacity-75"></span>
  <span class="relative inline-flex rounded-full h-3 w-3 bg-purple-500"></span>
</span>
```

- `animate-pulse` - 使元素淡入淡出，一般用于骨架屏的动画

```css
<div class="border border-light-blue-300 shadow rounded-md p-4 max-w-sm w-full mx-auto">
  <div class="animate-pulse flex space-x-4">
    <div class="rounded-full bg-light-blue-400 h-12 w-12"></div>
    <div class="flex-1 space-y-4 py-1">
      <div class="h-4 bg-light-blue-400 rounded w-3/4"></div>
      <div class="space-y-2">
        <div class="h-4 bg-light-blue-400 rounded"></div>
        <div class="h-4 bg-light-blue-400 rounded w-5/6"></div>
      </div>
    </div>
  </div>
</div>
```

- `animate-bounce` - 使元素上下弹跳，一般用于“向下滚动”之类的指示器

```css
<svg class="animate-bounce w-6 h-6 ... ">
  <!-- ... -->
</svg>
```

## 变换

### transform

- `transform` - 如果想启用变换则必须加的样式
- `transform-gpu` - 使用硬件加速实现，会有更好的性能
- `transform-none` - 无

### transform origin

控制变换的原点

| Class  | Properties  |
| ------------ | ------------ |
| `origin-center`  | `transform-origin: center;`  |
| `origin-top`  | `transform-origin: top;`  |
| `origin-top-right`  | `transform-origin: top right;`  |
| `origin-right`  | `transform-origin: right;`  |
| `origin-bottom-right`  | `transform-origin: bottom right;`  |
| `origin-bottom`  | `transform-origin: bottom;`  |
| `origin-bottom-left`  | `transform-origin: bottom left;`  |
| `origin-left`  | `transform-origin: left;`  |
| `origin-top-left`  | `transform-origin: top left;`  |

### scale

配合 `transform` 一起使元素缩放

- `scale-{percentage}` - 按百分比进行缩放
- `scale-{x|y}-{percentage}` - 水平或垂直按百分比缩放

### rotate

通过 `transform` `rotate-{angle}` 配合一起使元素旋转

### translate

通过 `transform` `translate-{x|y}-{amount}` 配合一起平移元素

### skew

通过 `transform` `skew-{x|y}-{amount}` 配合一起对元素进行2D转换

## 交互

### 外观

重置指定元素的浏览器默认外观，用于自定义组件样式

| Class  | Properties  |
| ------------ | ------------ |
| `appearance-none`  | `appearance: none;`  |


### cursor

控制鼠标悬浮元素上时的鼠标样式

| Class  | Properties  |
| ------------ | ------------ |
| `cursor-auto`  | `cursor: auto;`  |
| `cursor-default`  | `cursor: default;`  |
| `cursor-pointer`  | `cursor: pointer;`  |
| `cursor-wait`  | `cursor: wait;`  |
| `cursor-text`  | `cursor: text;`  |
| `cursor-move`  | `cursor: move;`  |
| `cursor-help`  | `cursor: help;`  |
| `cursor-not-allowed`  | `cursor: not-allowed;`  |

### outline

控制元素的 outline 的样式

| Class  | Properties  |
| ------------ | ------------ |
| `outline-none`  | `outline: 2px solid transparent;`  |
| `outline-offset: 2px;`  |
| `outline-white`  | `outline: 2px dotted white;`  |
| `outline-offset: 2px;`  |
| `outline-black`  | `outline: 2px dotted black;`  |
| `outline-offset: 2px;`  |

### pointer events

控制元素是否响应鼠标的事件

| Class  | Properties  |
| ------------ | ------------ |
| `pointer-events-none`  | `pointer-events: none;`  |
| `pointer-events-auto`  | `pointer-events: auto;`  |

### resize

控制元素应该如何被调整大小

| Class  | Properties  |
| ------------ | ------------ |
| `resize-none`  | `resize: none;`  |
| `resize-y`  | `resize: vertical;`  |
| `resize-x`  | `resize: horizontal;`  |
| `resize`  | `resize: both;`  |

### user select

控制用户是否能够在元素中选中文本内容

| Class  | Properties  |
| ------------ | ------------ |
| `select-none`  | `user-select: none;`  |
| `select-text`  | `user-select: text;`  |
| `select-all`  | `user-select: all;`  |
| `select-auto`  | `user-select: auto;`  |


## SVG

### fill

设置 SVG 填充的样式

| Class  | Properties  |
| ------------ | ------------ |
| `fill-current`  | `fill: currentColor;`  |


```css
<svg class="fill-current text-green-600 ..."></svg>
```


### stroke

设置 SVG 笔画的样式

| Class  | Properties  |
| ------------ | ------------ |
| `stroke-current`  | `stroke: currentColor;`  |

```css
<svg class="stroke-current text-purple-600 ..."></svg>
```