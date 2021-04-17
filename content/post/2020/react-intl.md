---
title: "React-Intl 实践总结"
date: 2020-05-01T23:39:50+08:00
draft: false
categories:
    - web
tags:
    - react-intl
    - react
    - 国际化
themeColor: "#190843"
coverColor: "#190843f7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410225237-react-intl.png
---

## 前言

在使用 React 做开发的时候，一直是使用 react-intl 对 React 组件进行国际化，网上相关文档很多，遇到的问题基本都能很容易的找到解决办法。本文仅用于记录个人使用总结，以免时日一多有所遗忘而已 ~ 其实已经因为遗忘导致来来回回翻找相关文档好多次了😵。

<warning>

> 本文中 react-intl 使用的版本是 4.5，一些旧的 API，如 addLocalData 都已经被移除了，具体变化可以参考官方的 [升级指南](https://formatjs.io/docs/react-intl/upgrade-guide-3x "升级指南")

</warning>

## 安装

```bash
npm install --save react-intl
```

## 使用

1. 创建 i18n 上下文

所有使用 react-intl 的组件，都必须在 `IntlProvider` 组件的上下文中使用，所以最常见的用法就是将根组件用 `IntlProvider` 包装并对其使用当前用户所在区域的语言和转换后的消息进行[配置](https://formatjs.io/docs/react-intl/components#intlprovider "配置")。

```js
// 定义本地化语言
const messages = {
  "zh-CN": {
    "pages.menu.save": "保存",
    "pages.menu.cancel": "取消"
  },
  "en-US": {
    "pages.menu.save": "save",
    "pages.menu.cancel": "cancel"
  }
};

const usersLocale = "zh-CN";

// 创建上下文
ReactDOM.render(
  <IntlProvider locale={usersLocale} messages={messages["zh-CN"]}>
    <App />
  </IntlProvider>,
  document.getElementById('container')
);
```

2. API 及格式化数据

react-intl 支持使用 React组件 及其 API 两种方式格式化数据，使用 API 时需要通过 InjectIntl 高阶组件或者 useIntl hooks 来获取 intl 对象，具体用法可以参考官方文档：

- [API](https://formatjs.io/docs/react-intl/api "API")
- [injectIntl](https://formatjs.io/docs/react-intl/api#injectintl "injectIntl")
- [defineMessages](https://formatjs.io/docs/react-intl/api#definemessages "defineMessages")
- [IntlProvider](https://formatjs.io/docs/react-intl/components#intlproviderhttp:// "IntlProvider")
- [FormattedDate](https://formatjs.io/docs/react-intl/components#formatteddatehttp:// "FormattedDate")
- [FormattedTime](https://formatjs.io/docs/react-intl/components#formattedtimehttp:// "FormattedTime")
- [FormattedRelativeTime](https://formatjs.io/docs/react-intl/components#formattedrelativetime "FormattedRelativeTime")
- [FormattedNumber](https://formatjs.io/docs/react-intl/components#formattednumber "FormattedNumber")
- [FormattedPlural](https://formatjs.io/docs/react-intl/components#formattedpluralhttp:// "FormattedPlural")
- [FormattedMessage](https://formatjs.io/docs/react-intl/components#formattedmessage "FormattedMessage")

## 插件

在使用的过程中可能会面临下面两个问题：

<info>

> 1. 在定义本地化语音包时，每一句待翻译的文本都需要定义其唯一标识 id，这绝对不是一件令人愉快的编程体验。
> 2. 在项目开发过程中，有规范的将语言包按模块/功能等拆分开定义是一个不错的选择，但是语言包仍然和代码是隔离状态的。

</info>

幸好社区中有现成的解决方案。

### 自动生成 ID

`babel-plugin-react-intl-auto` 插件可以将你从繁琐的 id 管理中释放出来，它可以根据文件路径自动为待翻译的文本生成 id。

- 1️⃣安装

```js
npm install --save-dev babel-plugin-react-intl-auto
# Optional: TypeScript support
npm install --save-dev @babel/plugin-transform-typescript
```

- 2️⃣配置

.babel.config.js

```js
module.exports = function (api) {
    api.cache(true);

    return {
        presets: [
            // ....
        ],
        plugins: [
            // https://github.com/akameco/babel-plugin-react-intl-auto
            [
                "react-intl-auto",
                {
                    // 移除的前缀：true - ID 中将不包含任何文件路径前缀
                    removePrefix: "src.",
                    // 使用文件名生成 ID
                    filebase: false,
                    // 使用前导注释作为消息说明
                    // 仅适用于使用 defineMessages 定义语音包的时候
                    extractComments: true,
                    // https://github.com/akameco/babel-plugin-react-intl-auto#usekey
                    useKey: true,
                    // ID 中单词之间的分隔符
                    separator: ".",
                    // 是否使用自定义模块作为定义Messages等的源
                    // https://github.com/akameco/babel-plugin-react-intl-auto/issues/74#issuecomment-528562743
                    // moduleSourceName: "react-intl"
                }
            ],
        ]
    }
}
```

- 3️⃣定义

/pages/menu/messages.ts

```js
// @ts-nocheck
import {defineMessages, MessageDescriptor} from "react-intl";

export default defineMessages<MessageDescriptor, {[name: string]: MessageDescriptor}>({
    // 菜单编辑页面中保存按钮的显示名称
    save: "保存",
    // 菜单编辑页面中取消按钮的显示名称
    cancel: "取消",
})
```

- 4️⃣使用

/pages/menu/edit.tsx

```js
import {useIntl} from "react-intl";
import messages from "./messages";

const Index = () => {
	const intl = useIntl();
    return (<div><button>{intl.formatMessage(messages.save)}</button></div>)
}
```

### 自动提取消息

`babel-plugin-react-intl-extractor` 插件能够自动将` react-intl` 中的消息提取合并到单独的文件中。

<warning>

> 依赖 babel 7

</warning>

- 1️⃣安装

```js
npm install --save-dev babel-plugin-react-intl-extractor
```

- 2️⃣配置

.babel.config.js

```js
module.exports = function (api) {
    api.cache(true);

    return {
        presets: [
            // ....
        ],
        plugins: [
            [
               //  "react-intl-auto" 配置 ...
            ],
            [
                "react-intl",
                {
                    messagesDir: “./src/locales”
                }
            ],
            [
                "react-intl-extractor",
                {
                    // 文件需要提前创建好，确保内容为正确的 json 格式，例如：[]
                    // 提取后聚合文件，包含了消息的 id、defaultMessage、description
                    extractedFile: "./locales/default.json"
                    langFiles: [
                        {
                            // 文件需要提前创建好，确保内容为正确的 json 格式，例如：{}
                            path: "./locales/messages/zh-CN.json",
                            cleanUpNewMessages: false
                        },
                        {
                            // 文件需要提前创建好，确保内容为正确的 json 格式，例如：{}
                            path: "./locales/messages/en-US.json",
                            cleanUpNewMessages: true
                        }
                    ]
                }
            ]
        ]
    }
}
```

- 3️⃣使用

嗯，一切都是自动的，只需要在创建 `IntlProvider` 时注意引入的语音包文件即可~

<warning>

> 插件的配置顺序是有要求的，必须是如下顺序：  
> babel-plugin-react-intl-auto  
> react-intl  
> babel-plugin-react-intl-extractor  

</warning>

## 参考文档

- [https://formatjs.io/docs/react-intl](https://formatjs.io/docs/react-intl "https://formatjs.io/docs/react-intl")

- [https://github.com/akameco/babel-plugin-react-intl-auto](https://github.com/akameco/babel-plugin-react-intl-auto "https://github.com/akameco/babel-plugin-react-intl-auto")

- [https://github.com/Bolid1/babel-plugin-react-intl-extractor](https://github.com/Bolid1/babel-plugin-react-intl-extractor "https://github.com/Bolid1/babel-plugin-react-intl-extractor")