---
title: 谈谈NODE_ENV
date: 2018-10-28
categories: NODE JS
tags: 
    - NODE
    - JS
#description: 
---

在开发项目的时候，不可避免的就是各个环境（正式/开发/测试）对应不同的配置信息。我们选择在构建项目的时候根据不同的命令（NODE_ENV）让代码引用不同的配置，而不是建不同的分支或者在服务器上修改配置文件。
<!-- more -->
说说几种类型得项目在构建的时候使用NODE_ENV区分环境的场景。
####
一、vue项目（以vue-admin框架为例）
a.package.json文件配置不同打包命令
```bash
"build": "node build/build.js",
"dev:build": "NODE_ENV=development node build/build.js"
```
b.build.js最上方添加代码
```js
process.env.NODE_ENV = process.env.NODE_ENV || 'production'
```
c.webpack.prod.conf.js文件里根据env引用不同的配置文件
```js
const env = process.env.NODE_ENV == 'production' ? require('../config/prod.env') :  require('../config/dev.build.env')
```
二、pomelo项目
a.package.json文件配置不同打包命令
```bash
"build": "NODE_ENV=production pomelo start -e production --daemon",
"build:dev": "NODE_ENV=dev pomelo start --daemon",
"build:test": "NODE_ENV=test pomelo start --daemon",
```
b.config.js根据env输出不同的配置
```js
const env = process.env.NODE_ENV
const config = {
  production: { },
  dev: {},
  tst: {}
}
module.exports = {
  Config: config[env]
}
```
三、babel项目
场景：以上俩项目都是基于运行时获取的env，我们只要配置正确即可。但有些项目我们只需要编译成静态文件，然后用服务启动，如果只是用之前的配置的话，最终编译输出的会是process.env.NODE_ENV，启动服务就会报错。
拿babel编译项目举例，搜索好久才寻得结果，参照 https://github.com/Tencent/wepy/issues/835 。
a. 安装插件 babel-plugin-transform-node-env-inline
```bash
npm i -D babel-plugin-transform-node-env-inline
```
b. .babelrc 文件配置plugins transform-node-env-inline （关键）
```bash
  "plugins": [
    "transform-node-env-inline",
    "xxxx"
  ]
```
c. package.json 中配置不同的编译命令
```js
"build": "NODE_ENV=production babel src -d dist",
"build:dev": "NODE_ENV=dev babel src -d dist",
"build:test": "NODE_ENV=test babel src -d dist"
```
d. config.js里根据env加载不同配置
```js
const envType = process.env.NODE_ENV
const config = {
  production: { },
  dev: {},
  tst: {}
}
export let config = config[env]
```

** 总结一句话就是首先在package.json里用NODE_ENV配置不同的打包/编译命令；然后在配置里根据process.env.NODE_ENV获取环境，输出不同的配置。（如果项目不支持运行时动态获取，可以想办法让其支持，比如安装插件 babel-plugin-transform-node-env-inline）
** 很多框架可能自行封装过，比如nuxt框架，请参照 https://zh.nuxtjs.org/api/configuration-env
