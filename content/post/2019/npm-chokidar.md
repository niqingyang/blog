---
title: "NPM - 使用 chokidar 监听文件变化"
date: 2019-05-13T23:06:44+08:00
draft: false
categories:
    - web
tags:
    - chokidar
    - nodejs
    - npm
    - js
coverColor: "#942b2bf7"
coverImage: https://cdn.jsdelivr.net/gh/acme-top/static@master/images/2021/04/20210410225005-npm-libs.png
---

## 前言

使用 Node 开发时，一些像 webpack 等编译打包的工具都会监听文件的变化，然后自动执行编译打包。如果是自己写一些类似的自动化脚本、工具之类的也会遇到同样的需求，而使用 chokidar 就是其中一个不错的选择。

## 标准库

Node.js 的标准库提供了 `fs.watch` 和 `fs.watchFile`，但是都存在一些问题...

Node.js fs.watch：

- 不报告MacOS上的文件名。
- 在MacOS上使用Sublime等编辑器时，根本不报告事件。
- 经常报告两次事件。
- 发布大多数更改为rename。
- 不提供递归查看文件树的简单方法。

Node.js fs.watchFile：

- 事件处理几乎一样糟糕。
- 也不提供任何递归观看。
- 导致高CPU利用率。
- Chokidar解决了这些问题。

Chokidar解决了这些问题。而且具官网声称微软的 Visual Studio Code, gulp, karma, PM2, browserify, webpack, BrowserSync 等等都在用这个库。它已在生产环境中得到证明。

## 安装

```js
npm install chokidar
```

## 用法

```js
const chokidar = require('chokidar');

// 监听当前目录, ignores .dotfiles
chokidar.watch('.', {ignored: /(^|[\/\\])\../}).on('all', (event, path) => {
  console.log(event, path);
});

// 一些典型的示例：

// 初始化一个监听器
const watcher = chokidar.watch('file, dir, glob, or array', {
  ignored: /(^|[\/\\])\../,
  persistent: true
});

const log = console.log.bind(console);

// 添加监听事件
watcher
  .on('add', path => log(`File ${path} has been added`))
  .on('change', path => log(`File ${path} has been changed`))
  .on('unlink', path => log(`File ${path} has been removed`));

// 其他可能用到的事件
watcher
  .on('addDir', path => log(`Directory ${path} has been added`))
  .on('unlinkDir', path => log(`Directory ${path} has been removed`))
  .on('error', error => log(`Watcher error: ${error}`))
  .on('ready', () => log('Initial scan complete. Ready for changes'))
  .on('raw', (event, path, details) => {
    log('Raw event info:', event, path, details);
  });

// 'add', 'addDir' and 'change' 事件也可以接收到一个统计结果信息作为第二个参数
// http://nodejs.org/api/fs.html#fs_class_fs_stats
watcher.on('change', (path, stats) => {
  if (stats) console.log(`File ${path} changed size to ${stats.size}`);
});

// 监听新的文件.
watcher.add('new-file');
watcher.add(['new-file-2', 'new-file-3', '**/other-file*']);

// 获取文件系统中实际上正在被监听的路径列表
var watchedPaths = watcher.getWatched();

// 禁止监听指定的文件
watcher.unwatch('new-file*');

// 停止监听器
watcher.close();

// 所有的参数列表，详情可见 https://github.com/paulmillr/chokidar#api
chokidar.watch('file', {
  persistent: true,

  ignored: '*.txt',
  ignoreInitial: false,
  followSymlinks: true,
  cwd: '.',
  disableGlobbing: false,

  usePolling: true,
  interval: 100,
  binaryInterval: 300,
  alwaysStat: false,
  depth: 99,
  awaitWriteFinish: {
    stabilityThreshold: 2000,
    pollInterval: 100
  },

  ignorePermissionErrors: false,
  atomic: true // or a custom 'atomicity delay', in milliseconds (default 100)
});
```

