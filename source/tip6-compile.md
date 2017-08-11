---
layout: default
id: tip6-compile
title: 代码编译
prev: tip5-debug.html
---


### client
前端的bundle build使用webpack来构建，使用cli命令创建项目，会自动生成[webpack配置](https://github.com/hcnode/koa-cola/blob/master/template/webpack.config.js)
ts文件的loader使用了[awesome-typescript-loader](https://github.com/s-panferov/awesome-typescript-loader)，并配置了使用babel，加入babel-polyfill到bundle，可以兼容ie9+。

webpack的入口tsx文件在项目里面的`view/app.tsx`:
```javascript
import * as React from 'react';
import { render } from 'react-dom';
import IndexController from '../api/controllers/IndexController';
import index from './pages/index';
import officialDemo from './pages/officialDemo';
import colastyleDemo from './pages/colastyleDemo';

var { createProvider } = require('koa-cola');
// 使用koa-cola提供的createProvider会自动建立路由，如果手动使用官方的Provider，则需要开发者手动写router
var Provider = createProvider([IndexController], {
  index,
  officialDemo,
  colastyleDemo
});

render(<Provider />, document.getElementById('app'));
```

wepack build 新建默认的项目得到的bundle的大小有400K，依赖的库组成如下图：
<img src="https://github.com/hcnode/koa-cola/raw/master/screenshots/bundle.png" alt="Drawing" width="800"/>

webpack的配置文件默认加了四个IgnorePlugin插件，因为有些文件是前后端都会使用，所以需要忽略服务器端的require。

```javascript
// 以下两个是给服务器端使用，不能打包到webpack
new webpack.IgnorePlugin(/\.\/src\/app/),
new webpack.IgnorePlugin(/\.\/src\/util\/injectGlobal/),
// 以下两个是controller引用的，也是服务器端使用，也不能打包到webpack，如果你的controller也有服务器端使用的库，也必须要加IgnorePlugin插件
new webpack.IgnorePlugin(/koa$/),
new webpack.IgnorePlugin(/koa-body$/),
```


### server
koa-cola本身框架只编译了部分代码，比如es6的module import和export，ts类型相关的语法，对es6或者es7（比如async/await）没有进行编译，尽量用到node.js原生的es高级语法（所以会不支持低版本的node），如果你想希望你的应用在低版本node下使用，则需要你手动build出你所希望的代码，并包括所依赖的koa-cola库。

如果在node.js 8.0的环境下运行，则可以不需要任何编译，可以直接使用ts-node运行（cli运行命令都是使用ts-node），甚至可以直接[线上使用](https://github.com/TypeStrong/ts-node/issues/104)