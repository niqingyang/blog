---
title: "搭建 react + mobx + antd 应用"
slug: "create-react-mobx-and-app"
date: 2019-06-18T22:01:36+08:00
draft: false
categories:
    - web
tags:
    - react
    - antd
    - mobx
coverColor: "#252525f7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410224816-react.png
---

## 前言

Create React App 是 FaceBook 的 React 团队官方出的一个构建 React 单页面应用的脚手架工具。它本身集成了 Webpack，并配置了一系列内置的 loader 和默认的 npm 的脚本，可以很轻松的实现零配置就可以快速开发 React 的应用。

## 环境

- 使用的技术：react + mobx + antd

- 相关版本（不容忽视的存在）：

<!--begin.mdui-table-fluid.mdui-table-hoverable-->

|  技术 |  版本 |
| :------------ | :------------ |
|  create-react-app  |  3.0.1  |
|  react  |  16.8.6  |
|  antd  |  3.19.3  |
|  babel |  7  |
|  mobx |  5.10.1  |
|  webpack |  4.29.6  |

<!--end.mdui-table-fluid.mdui-table-hoverable-->

## 安装

```shell
# 全局安装
npm install -g create-react-app

# 构建一个 my-app 的项目
npx create-react-app my-app
cd my-app

# eject
npm run eject

# 启动编译当前的React项目，并自动打开 http://localhost:3000/
npm start
```
<!--begin.warning-->

> 如果不能确保最新版本，可以先尝试卸载： npm uninstall -g create-react-app，然后再全局安装。

<!--end.warning-->

## 默认脚本

```shell
# 启动开发
npm start

# 启动测试
npm test

# 构建生产版本
npm run build

# 将所有配置文件和传递依赖项（Webpack，Babel，ESLint等）作为依赖项复制到项目中
npm run eject
```

## 安装常用库


```shell
# 常用库
npm i prop-types axios classnames invariant ismobilejs lodash lodash-decorators lodash.groupby history moment multer path-to-regexp qs react-hot-loader react-intl react-router react-router-dom slash2
# mobx 库
npm i mobx mobx-react mobx-react-router mobx-react-stores
# mock 服务
npm i -D @acme-top/express-mock-middleware
# antd 相关
npm i antd antd-form-plus react-document-title react-copy-to-clipboard react-fittext react-media
# antd pro 相关
npm i @antv/data-set bizcharts bizcharts-plugin-slider memoize-one debug mustache numeral nzh rc-animate
# 开发依赖库
npm i -D react-container-query
```

## 安装 webpack 插件

安装

```shell
# 编译进度插件
npm i -D progress-bar-webpack-plugin
# antd 主题插件（不使用 antd 可以忽略）
npm i -D antd-theme-webpack-plugin
# 将所有 less 合并为一个供 themePlugin使用
npm i -D antd-pro-merge-less
```

配置

```js
## 修改 /config/webpack.config.js，在相应的位置加入如下代码

// 编译进度插件
const ProgressBarPlugin = require('progress-bar-webpack-plugin');
const IgnorePlugin = require('webpack/lib/IgnorePlugin');
const HashedModuleIdsPlugin = require('webpack/lib/HashedModuleIdsPlugin');
// AntD 主题插件
// https://github.com/mzohaibqc/antd-theme-webpack-plugin
// https://www.npmjs.com/package/antd-pro-merge-less
const MergeLessPlugin = require('antd-pro-merge-less');
const AntDesignThemePlugin = require('antd-theme-webpack-plugin');

// CSS Module Local Ident
// antdpro /config/config.js
// 使用固定值 localIdentName: '[name]__[local]__[hash:base64:5]' 会导致 less 在页面中无法修改部分样式的主题色
const slash = require('slash2');
const getCSSModuleLocalIdent = (context, localIdentName, localName) => {
    if (
        context.resourcePath.includes('node_modules') ||
        context.resourcePath.includes('ant.design.pro.less') ||
        context.resourcePath.includes('global.less')
    ) {
        return localName;
    }
    const match = context.resourcePath.match(/src(.*)/);
    if (match && match[1]) {
        const antdProPath = match[1].replace('.less', '');
        const arr = slash(antdProPath)
            .split('/')
            .map(a => a.replace(/([A-Z])/g, '-$1'))
            .map(a => a.toLowerCase());
        return `antd-pro${arr.join('-')}-${localName}`.replace(/--/g, '-');
    }
    return localName;
}

// 将所有 less 合并为一个供 themePlugin使用
const outFile = path.join(paths.appPath, '/.temp/ant-design-pro.less');
const stylesDir = path.join(paths.appPath, '/src/');

## 注释掉这段代码

// const getCSSModuleLocalIdent = require('react-dev-utils/getCSSModuleLocalIdent');

## 在 webpack 的 plugins 中配置

new ProgressBarPlugin(),
new IgnorePlugin(/^\\.\\/locale$/, /moment$/),
new HashedModuleIdsPlugin(),
// Antd Pro Theme
isEnvDevelopment && new MergeLessPlugin({
	stylesDir,
	outFile,
}),
isEnvDevelopment && new AntDesignThemePlugin({
	antDir: path.join(paths.appPath, '/node_modules/antd'),
	stylesDir,
	varFile: path.join(paths.appPath, '/node_modules/antd/lib/style/themes/default.less'),
	mainLessFile: outFile, // themeVariables: ['@primary-color'],
	indexFileName: 'index.html',
	generateOne: true,
	lessUrl: 'https://gw.alipayobjects.com/os/lib/less.js/3.8.1/less.min.js'
}),
```

## 安装 DLL 插件

在 `/config/` 目录下新建文件 `dll.entry.js`，内容如下：

```js
// 具体打包内容自定义
module.exports = {
	// index.html 加入的 js 文件名称，如需打包成多个文件需要再定义一个元素
    vendors: [
        'react',
        'react-dom',
        'react-router',
        'react-router-dom',
        'react-copy-to-clipboard',
        'react-intl',
        'mobx',
        'mobx-react',
        'moment',
        'slash2',
        'lodash',
        'lodash-decorators',
    ]
};
```

在 `/config/` 目录下新建文件 `webpack.dll.config.js`，内容如下：

```js
const path = require('path');
const webpack = require('webpack');
const paths = require('./paths');

module.exports = {
    entry: require('./dll.entry'),
    output: {
        path: path.join(paths.appPublic, '/static/js'), // 文件输出的路径
        filename: "[name].js",
        library: "[name]_dll_[hash]" // 生成一个变量名供 dllreference 调用要与 dllPlugin.name 保持一致
    },
    plugins: [
        new webpack.DllPlugin({
            // 动态链接库的全局变量名称，需要和 output.library 中保持一致
            // 该字段的值也就是输出的 manifest.json 文件 中 name 字段的值
            name: "[name]_dll_[hash]",
            path: path.join(paths.appPublic, '/static/js/[name].manifest.json'),// 生成manifest.json文件的路径
            context: __dirname
        })
    ]
}
```

修改文件 `/config/webpack.config.js`，增加如下内容

```js
## 修改 /config/webpack.config.js，在相应的位置加入如下代码

const DllReferencePlugins = Object.keys(require('./dll.entry')).map(name => {
    return new webpack.DllReferencePlugin({
        context: __dirname,
        manifest: require(path.join(paths.appPublic, `/static/js/${name}.manifest.json`))
    });
});

## 在 webpack 的 plugins 中配置

...DllReferencePlugins
```

修改文件 `/public/index.html`，在 `body` 末尾加入如下内容：

```html
<script src="%PUBLIC_URL%/static/js/vendors.js"></script>
```

修改 `package.json` 在 `scripts` 中加入

```js
"dll": "webpack --config config/webpack.dll.config",
```

<!--begin.tip-->

> 使用命令 `npm run dll` 即可生成 dll 文件

<!--end.tip-->

## 配置 Less

安装

```shell
npm i -D less less-loader
```

参考 css 和 sass 的配置，编写 less 的配置，加入如下代码：

```diff
// style files regexes
const cssRegex = /\.css$/;
const cssModuleRegex = /\.module\.css$/;
const sassRegex = /\.(scss|sass)$/;
const sassModuleRegex = /\.module\.(scss|sass)$/;
+ const lessRegex = /\.less$/;
+ const lessModuleRegex = /\.module\.less$/;
```

在下面代码处开启 `javascriptEnabled`

```diff
if (preProcessor) {
	loaders.push({
		loader: require.resolve(preProcessor),
		options: {
			sourceMap: isEnvProduction && shouldUseSourceMap,
+			javascriptEnabled: true,
		},
	});
}
```

在webpack中加入如下代码：

```diff
...
    // Adds support for CSS Modules, but using SASS
    // using the extension .module.scss or .module.sass
    {
        test: sassModuleRegex,
        use: getStyleLoaders(
            {
                importLoaders: 2,
                sourceMap: isEnvProduction && shouldUseSourceMap,
                modules: true,
                getLocalIdent: getCSSModuleLocalIdent,
            },
            'sass-loader'
        ),
    },

+   // Less
+   {
+       test: lessRegex,
+       exclude: lessModuleRegex,
+       use: getStyleLoaders({
+           importLoaders: 2,
+           sourceMap: isEnvProduction && shouldUseSourceMap,
+           // 改动
+           modules: true,   // 新增对css modules的支持
+           getLocalIdent: getCSSModuleLocalIdent,
+           // localIdentName: '[name]__[local]__[hash:base64:5]',
+       }, 'less-loader'),
+       // Don't consider CSS imports dead code even if the
+       // containing package claims to have no side effects.
+       // Remove this when webpack adds a warning or an error for this.
+       // See https://github.com/webpack/webpack/issues/6571
+       sideEffects: true,
+   },
+   // Adds support for CSS Modules (https://github.com/css-modules/css-modules)
+   // using the extension .module.css
+   {
+       test: lessModuleRegex,
+       use: getStyleLoaders({
+           importLoaders: 2,
+           sourceMap: isEnvProduction && shouldUseSourceMap,
+           modules: true,
+           getLocalIdent: getCSSModuleLocalIdent,
+       }, 'less-loader'),
+   },

    // "file" loader makes sure those assets get served by WebpackDevServer.
    // When you `import` an asset, you get its (virtual) filename.
    // In production, they would get copied to the `build` folder.
    // This loader doesn't use a "test" so it will catch all modules
    // that fall through the other loaders.
    {
        loader: require.resolve('file-loader'),
        // Exclude `js` files to keep "css" loader working as it injects
        // its runtime that would otherwise be processed through "file" loader.
        // Also exclude `html` and `json` extensions so they get processed
        // by webpacks internal loaders.
        exclude: [/\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/],
        options: {
            name: 'static/media/[name].[hash:8].[ext]',
        },
    },
...
```

## 配置 antd 按需加载

安装插件

```shell
npm i -D babel-plugin-import
```

修改 `/config/webpack.config.js`，加入如下代码

```diff
    // Process application JS with Babel.
    // The preset includes JSX, Flow, TypeScript, and some ESnext features.
    {
        test: /\.(js|mjs|jsx|ts|tsx)$/,
        include: paths.appSrc,
        loader: require.resolve('babel-loader'),
        options: {
            customize: require.resolve(
                'babel-preset-react-app/webpack-overrides'
            ),

            plugins: [
                [
                    require.resolve('babel-plugin-named-asset-import'),
                    {
                        loaderMap: {
                            svg: {
                                ReactComponent: '@svgr/webpack?-svgo,+ref![path]',
                            },
                        },
                    },
                ],
+               // antd 按需加载
+               [
+                   "import",
+                   {
+                       libraryName: 'antd',
+                       libraryDirectory: 'es',
+                       // `style: true` 会加载 less 文件
+                       style: 'css'
+                   }
+               ]
            ],
            // This is a feature of `babel-loader` for webpack (not Babel itself).
            // It enables caching results in ./node_modules/.cache/babel-loader/
            // directory for faster rebuilds.
            cacheDirectory: true,
            cacheCompression: isEnvProduction,
            compact: isEnvProduction,
        },
    },
```

## 配置热加载

安装

```shell
## 注意：可以安全地将 react-hot-loader 安装为常规依赖项而不是dev依赖项，因为它会自动确保它不会在生产中执行且占用空间很小。
npm install react-hot-loader
```

修改 package.json， 在 babel 中加入配置

```diff
	"babel": {
		"presets": [
			"react-app"
		],
+		"plugins": [
+			"react-hot-loader/babel"
+		]
	},
```

要求：
1. 确保根组件被标记为热加载

```js
// App.js
import { hot } from 'react-hot-loader/root';
const App = () => <div>Hello World!</div>;
export default hot(App);
```

2. 确保 react-hot-loader 在 react 和 react-dom 之前被 required

> 或者在入口文件中（在 react 之前） import 'react-hot-loader'
> 或者 webpack entry point 前面用 react-hot-loader/patch 拼接

3. 如果需要支持 hooks，需要使用 react-hot-dom

4. 具体使用方式可以参考官方的 [github](https://github.com/gaearon/react-hot-loader "github")

## 配置 babel

安装相关库

```shell
## babel 编译的核心库
npm install --save-dev @babel/core
## 预设环境
npm install --save-dev @babel/preset-env
## babel 钩子
npm install --save-dev @babel/register
## 将类和对象装饰器编译为ES5
npm install --save-dev @babel/plugin-proposal-decorators
## 用来解析 import
npm install --save-dev @babel/plugin-syntax-dynamic-import
## 此插件转换静态类属性以及使用属性初始化程序语法声明的属性
npm install --save-dev @babel/plugin-proposal-class-properties
## 允许解析 import 的 meta
npm install --save-dev @babel/plugin-syntax-import-meta
## 在JS字符串中转移 U+2028 LINE SEPARATOR 和 U+2029 PARAGRAPH SEPARATOR
npm install --save-dev @babel/plugin-proposal-json-strings
## 外部引用辅助函数和内置函数，自动填充代码而不会污染全局变量
npm install --save-dev @babel/plugin-transform-runtime
## 安装 core-js
npm install --save core-js@3
##
npm install --save-dev @babel/runtime-corejs3
```

配置

```diff
	"babel": {
		"presets": [
			"react-app",
+			[
+				"@babel/preset-env",
+				{
+					"useBuiltIns": "usage",
+					"corejs": "3"
+				}
+			]
+		],
+		"plugins": [
+			"react-hot-loader/babel",
+			"@babel/plugin-syntax-dynamic-import",
+			"@babel/plugin-syntax-import-meta",
+			[
+				"@babel/plugin-proposal-decorators",
+				{
+					"legacy": true
+				}
+			],
+			[
+				"@babel/plugin-proposal-class-properties",
+				{
+					"loose": false
+				}
+			],
+			"@babel/plugin-proposal-json-strings",
+			[
+				"@babel/plugin-transform-runtime",
+				{
+					"absoluteRuntime": false,
+					"corejs": "3",
+					"helpers": true,
+					"regenerator": true,
+					"useESModules": false
+				}
+			]
+		]
	},
```

配置支持的浏览器，browserlist 具体配置可参考官方 [github](https://github.com/browserslist/browserslist "github") ~ 根据具体情况自行修改


```json
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all",
      "IE >= 9"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
```

## 配置目录别名

添加 `@` 代表 `src` 目录

```diff
	alias: {
		// Support React Native Web
		// https://www.smashingmagazine.com/2016/08/a-glimpse-into-the-future-with-react-native-for-web/
		'react-native': 'react-native-web',
+		'@': paths.appSrc
	},
```

## 配置 mock 服务

安装

```shell
npm i -D @acme-top/express-mock-middleware mockjs express
```

在 `src` 目录下新建文件 `setupProxy.js`，内容如下

```js
const path = require("path");
const proxy = require('http-proxy-middleware');
const paths = require('../config/paths');

require('@babel/register');

module.exports = function (app) {

    const mockPaths = [
        path.join(paths.appPath, 'mock')
    ];

    app.use(require("@acme-top/express-mock-middleware").createMiddleware(mockPaths));
};
```

## 配置 eslint

安装

```shell
npm i -D  eslint-config-airbnb eslint-config-prettier eslint-plugin-compat eslint-plugin-import eslint-plugin-markdown eslint-plugin-react
```

在项目根目录下新建文件 `.eslintrc.js`，内容根据自己的规范自行修改

```js
module.exports = {
    parser: 'babel-eslint',
    extends: ['airbnb', 'prettier', 'plugin:compat/recommended'],
    env: {
        browser: true,
        node: true,
        es6: true,
        mocha: true,
        jest: true,
        jasmine: true,
    },
    globals: {
		// 项目中定义的全局变量 ~ 请自行修改
        REACT_APP_ANT_DESIGN_PRO_ONLY_DO_NOT_USE_IN_YOUR_PRODUCTION: true,
        page: true,
    },
    rules: {
        'react/jsx-filename-extension': [1, {extensions: ['.js']}],
        'react/jsx-wrap-multilines': 0,
        'react/prop-types': 0,
        'react/forbid-prop-types': 0,
        'react/jsx-one-expression-per-line': 0,
        'react/jsx-indent': 0,
        'react/jsx-indent-props': 0,
        'react/jsx-tag-spacing': 0,
        'react/button-has-type': 0,
        'react/jsx-no-undef': 0,
        'react/no-find-dom-node': 0,
        'import/prefer-default-export': 0,
        'react/jsx-first-prop-new-line': 0,
        'import/no-unresolved': [2, {ignore: ['^@/']}],
        'import/no-extraneous-dependencies': 0,
        // 'import/no-cycle': 0,
        'jsx-a11y/no-noninteractive-element-interactions': 0,
        'jsx-a11y/click-events-have-key-events': 0,
        'jsx-a11y/no-static-element-interactions': 0,
        'jsx-a11y/anchor-is-valid': 0,
        'no-unreachable': 0,
        'no-unused-vars': 0,
        'no-use-before-define': 0,
        'no-param-reassign': 0,
        'no-else-return': 0,
        'no-underscore-dangle': 0,
        'no-console': 0,
        'linebreak-style': 0,
        'prefer-destructuring': 0,
        'prefer-rest-params': 0,
        'global-require': 0,
        'compat/compat': 0
    },
    settings: {
        polyfills: ['fetch', 'promises', 'url', 'object-assign'],
    },
};
```

## 创建开发目录

目录推荐 Antd Pro 的结构，如下

```shell
├── config                   # 配置，包含 webpack、构建等配置
├── mock                     # 本地模拟数据
├── public
│   └── favicon.png          # Favicon
├── src
│   ├── assets               # 本地静态资源
│   ├── components           # 业务通用组件
│   ├── e2e                  # 集成测试用例
│   ├── layouts              # 通用布局
│   ├── models               # 全局 model
│   ├── pages                # 业务页面入口和常用模板
│   ├── services             # 后台接口服务
│   ├── utils                # 工具库
│   ├── locales              # 国际化资源
│   ├── global.less          # 全局样式
│   └── global.ts            # 全局 JS
├── tests                    # 测试工具
├── README.md
└── package.json
```






















