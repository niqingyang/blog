---
title: "使用 mobx-react-stores 开发 react 应用"
date: 2019-06-19T01:11:29+08:00
draft: false
categories:
    - web
tags:
    - react
    - antd
    - mobx
themeColor: "#d75813"
coverColor: "#d75c14f7"
coverImage: https://static.acme.top/wp-content/uploads/2019/06/mobx-react-stores.png
---

## 前言

mobx-react-stores 是为了方便将 antd pro 改为使用 mobx 而开发的，里面的思路和代码大量的参考了 dva 和 umi 的实现

## 安装

```shell
npm i mobx-react-stores
```

## 使用

获取 `stores` 实例对象

```js
import {stores} from 'mobx-react-stores';
```

### 状态管理

mobx-react-stores 可以用作状态管理，也是其最核心的目的，接口设计的比较简单，主要就两个：

1. **stores.add(model: object | Promise, namespaceAlias: null | string)** - 用来向 `stores` 注入待管理的 `model` 对象

- 参数 `model` 如果为对象则必须包含一个 string 类型的 `namespace` 属性，用作其在 `stores` 中的唯一标识，也就是 `stores` 中的属性名称
- 参数 `model` 也可以为一个 `Promise` 对象，`Promise` 必须返回一个包含 string 类型的 `namespace` 属性
- 参数 `namespaceAlias` 可选，用来为 `model` 起别名，如果存在则 `model` 可以不包含 `namespace` 属性

2. **dispatch({type, payload})** - 类似 `dva` 的 `dispatch`，用来通过 `namespace/action` 这种形式快速访问 `model` 对象中的函数

- 参数 `type` 为字符串类型，格式为 `namespace/action`
- 参数 `payload` 为任意类型，可选，用于向所调用的函数中传递参数，`namespace/action` 函数接收到的参数就是 `payload`
- `dispatch` 一般直接被注入到了上下文中，方便组件调用

### 监听加载状态

`mobx-loading` 能实现类似 dva-loading 的效果，方便监听 model 及其 action 的执行状态，减少开发人员对组件 `show` `hide` 的操作

- 获取 loadingStore 对象的方法

```js
import {loadingStore} from 'mobx-loading'
```

mobx-react-stores 默认集成了 mobx-loading，作为 stores 的内置属性，namespace 为 `loading`，所以可以直接从 stores 中获取 loadingStore 对象

```js
const {loading} = stores;
```

- 使用方式

```js
// 通过 actions 获取指定 namespace/action 的当前加载状态
loading.actions['goods/fetchList']

// 通过 models 获取指定 namespace 的当前加载状态
// namespace 下任何 action 加载状态为 true，则此 namespace 的加载状态即为 true
loading.models.goods

// 通过 global 获取全局的加载状态
// 只要任何一个 model 的状态为 true，则 global 的状态即为 true
loading.global

// 从 mobx-react-stores 和 mobx-loading 中都可以获取 namespace 装饰器
// 用作 class，方便向 class 中注入 namespace 属性
import {namespace} from 'mobx-loading';
import {namespace} from 'mobx-react-stores';
```

- 使用示例

```js
// 获取 namespace 和 loading 装饰器
import {namespace, loading} from 'mobx-react-stores';
import {fetchRandomUser} from "@/services/demo";

// 默认会为 class 添加 namespace 属性，值为将 class 类名转换为首字母小写的字符串
// 此处 namespace 值为 randomUser
@namespace
class RandomUser {

    user;

    message;

	// 通过 loading 装饰器，向 loadingStore 中注入当前 action 的加载状态
    @loading
    fetchUser = async () => {

        this.user = null;
        this.message = null;

        const response = await fetchRandomUser().catch(this.onRejected);

        if (response.results) {
            const user = response.results[0];

            this.change({
                user: {
                    name: `${user.name.first} ${user.name.last}`,
                    email: user.email,
                    picture: user.picture.large
                },
                message: null
            });
        }
    }

    change = ({user, message}) => {
        this.user = user;
        this.message = message;
    }

    onRejected = (e) => {
        return {
            status: 500,
            message: e.message
        }
    }
}

export default new RandomUser();

// -----------------------RandomUser.js--------------------------------

import React from 'react';
import {inject} from 'mobx-react'

@inject(({stores: {dispatch, loading, randomUser}}) => {
    return {
        dispatch,
        randomUser,
        loading: loading.models.randomUser
    }
})
class RandomUser extends React.PureComponent {

    onFetchUser = () => {
        const {dispatch} = this.props;
        dispatch({
            type: 'randomUser/fetchUser'
        })
    }

    render() {
        const {loading, randomUser} = this.props;

        if (loading) {
			// do someting
		}else{
			const {user, message} = randomUser;
			// do someting
		}
	}
}

export default RandomUser;

```

-  **@loading(names: string | array | null)** - loading 装饰器作用于 model 内的函数，可以接收字符串和数组的参数

- 例如： @loading('randomUser/fetchList')、@namespace(['randomUser', 'fetchList']) 与上面的示例代码结果一样

- `loading` 装饰器优先使用当前 class 的 namespace 属性和 函数的名称（首字母小写）生成此 action 的唯一标识，无 namespace 属性则使用 class 的名称（首字母小写）。上门示例中 loading 无参则其函数的加载状态可由 `loadingStore.actions['randomUser/fetchList']` 取得

<!--begin.warning-->

> 上面示例中的 loading 是装饰器，与 stores.loading 不同。stores.loading 是 loadingStore 注入 stores 后的属性

<!--end.warning-->

### 路由管理

为了方便管理路由以及向 stores 中注入 model，mobx-react-stores 提供了类似 umi 管理路由的方法，不过没有像 umi 那样去生成临时文件，而是更直接的方式

<!--begin.tip-->

> 建议在 `src` 目录下新建 `default.router.js` 文件来存放路由，如果模块多而复杂，可以将各个模块的路由分散开来然后聚合到这个文件中

<!--end.tip-->

**路由示例**

```js
export default [
    // user
    {
        path: '/user',
		// 布局文件
        component: () => import('./layouts/UserLayout'),
		LoadingComponent = require('./components/PageLoading').default，
		Routes: [require('./pages/Authorized').default],
        authority: ['admin', 'user'],
        routes: [
			// 从 /user 重定向到 /user/login
            {path: '/user', redirect: '/user/login'},
            {path: '/user/login', name: 'login', component: () => import('./pages/User/Login')},
            {
                name: 'register',
                path: '/user/register',
                models: () => [import('./pages/User/models/register')],
                component: () => import('./pages/User/Register')
            },
            {
                name: 'register.result',
                path: '/user/register-result',
                component: () => import('./pages/User/RegisterResult'),
            },
        ],
    }
]
```

- **name** - 路由名称，用于生成菜单，name 和 path 都存在的才会生成菜单 ~ Antd Pro 这么定义的
- **path** - 路由的路径
- **models** - model 对象集合，支持按需加载，会与 component 一同加载，支持：`() => [import('xxx/model1'), import('xxx/model2')]`、`() => import('xxx/model1')`、`[() => import('xxx/model1'), () => import('xxx/model2')]` 等形式。主要方便于子路由的 models 和上级路由 models 进行数组合并，一些公共的 models 可以放在上级路由中，方便管理。
- **component** - 组件，如果有下级路由，则是下级路由的布局文件，支持异步按需加载（需提供一个函数，类似：`()=>import('xxxx')）`
- **LoadingComponent** - 异步按需加载组件时的加载效果组件
- **Routes** - 方便自定义权限校验路由，可参考 Antd Pro 去实现
- **authority** - 访问路由所需的权限，用于权限校验，可参考 Antd Pro 去实现
- **routes** - 子路由集合

**使用路由**

```js
import {formatterRoutes, renderRoutes} from 'mobx-react-stores';
import router from './default.router.js';

// ... do someting

// 格式化
const routes = formatterRoutes(this.router, this.stores);
// 渲染路由
let children = renderRoutes(routes);

return (
	<Router history={history}>
		{children}
    </Router>
);

// ... do someting
```

### 国际化

mobx-react-stores 推荐使用 react-intl 来管理国际化

注入 Locale

```js
import {stores, Locale} from 'mobx-react-stores';

// ... do someting

// 可参考 Antd Pro
const translations = {
    'en-US': {
        messages: {
            ...require('../locales/en-US.js').default,
        },
        locale: 'en-US',
        antd: require('antd/lib/locale-provider/en_US'),
        data: require('react-intl/locale-data/en'),
        momentLocale: '',
    },
    'zh-CN': {
        messages: {
            ...require('../locales/zh-CN.js').default,
        },
        locale: 'zh-CN',
        antd: require('antd/lib/locale-provider/zh_CN'),
        data: require('react-intl/locale-data/zh'),
        momentLocale: 'zh-cn',
    },
	// ... 其他语言包
};

// 注入
stores.add(new Locale('zh-CN', translations));

// ... do someting
```

**切换语言**

```js
stores.locale.change('zh-CN');

// 或者

dispatch({
	type: 'locale/change',
	payload: 'zh-CN'
});
```

## 集成

为了简化开发，方便使用 国际化、history 等，mobx-react-stores 提供了集成好的接口

**使用示例**

在 `src` 目录下新建 `index.js` 文件作为入口文件

```js
// 兼容 IE9、IE10 ~ 如果需要的话请解开封印
// import 'react-app-polyfill/ie9';
// import 'react-app-polyfill/stable';
import React from 'react';
import {LocaleProvider} from 'antd';
import {app} from 'mobx-react-stores';
import get from 'lodash/get';
import './utils/axios'; // 初始化拦截器
import './global.less';
import * as serviceWorker from './serviceWorker';

app.stores = require('./models').default;
app.router = require('./default.router').default;

app.defaultLoadingComponent = require('./components/PageLoading').default;

// 渲染时回调，方便一些额外的操作，比如注入 antd 的国际化
app.renderCallback = (children) => {

    const {locale} = app.stores;

    const antd = get(app.stores, 'locale.translation.antd', undefined);

    if (antd) {
        return (
            <LocaleProvider locale={antd.default || antd}>
                {children}
            </LocaleProvider>
        );
    }

    return children;
};

app.render(document.getElementById('root'));

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

**集成后**

- **stores** 中将获得 `intl`、`routing` 两个对象，可用于国际化和操作路由
- **intl** 是 react-intl 注入后的实例对象，可使用 `intl.formatMessage()` 等函数
- **routing** 可以实现 `routing.push()`、`routing.replace()` 等路由操作













