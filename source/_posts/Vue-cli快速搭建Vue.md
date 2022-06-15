---
title: Vue-cli快速搭建Vue
date: 2017-03-05
categories: JS
tags: 
    - Vue
    - Vue-cli
#description: 
---

Vue + webpack模式是目前使用很广泛的前端解决方案，而Vue-cli脚手架能快速帮你搭建Vue项目框架，让你更专注于业务需求开发。下面介绍vue-cli的安装步骤。

<!-- more -->

### 一、环境准备
nodejs安装  -  [nodejs官网](https://nodejs.org/en/download/ "nodejs官网下载安装包")

### 二、安装 vue-cli 脚手架
``` bash
npm install -g vue-cli
```
### 三、使用vue-cli初始化项目
``` bash
vue init webpack {安装目录}
```
** mark:此处可供选择的形式有5种 **
+ webpack - A full-featured Webpack + vue-loader setup with hot reload, linting, testing & css extraction
+ webpack-simple - A simple Webpack + vue-loader setup for quick prototyping
+ browserify - A full-featured Browserify + vueify setup with hot-reload, linting & unit testing
+ browserify-simple - A simple Browserify + vueify setup for quick prototyping
+ simple - The simplest possible Vue setup in a single HTML file

** init步骤： **
``` bash
- Project name vue-vli
- Project description tony's first vue pro
- Author tony
- Vue build standalone
- Install vue-router? Yes
- Use ESLint to lint your code? Yes
- Pick an ESLint preset Standard
- Setup unit tests with Karma + Mocha? Yes
- Setup e2e tests with Nightwatch? Yes
```

### 四、进入目录
``` bash
cd {安装目录}
```

### 五、安装依赖
``` bash
npm install
```
** mark: 此处安装依赖有几种办法： **
- 1.首先在package.json中配置好devDependencies，然后直接执行npm install（脚手架里用的这种方法）
- 2.逐个安装所需要的依赖，npm install xxx —save-dev，带上—dev参数，则自动会把依赖添加到package.json中

### 六、开始运行
``` bash
npm run dev
```
** mark:此处的 dev 命令见 package.json 中配置的 scripts 属性 **

OK，到此算完成简单的搭建了，访问localhost://8080即可访问，以下为学习笔记。
