---
title: stylelint实践
date: 2022-07-08
categories: NODE CSS
tags: 
    - NODE
    - CSS
#description: 
---

stylelint是一个强大的，现代的代码检查工具，可以帮助你在团队合作中强制执行样式约定.

<!-- more -->

#### 一、安装

```bash
npm i stylelint --D
```

#### 二、配置文件

可以有以下三种方式配置stylelint规则(一旦发现它们中的任何一个，将不再继续进行查找，进行解析，将使用解析后的对象)，下面以第三种举例：

+ package.json 中的stylelint 属性
+ .stylelintrc.js 文件
+ stylelint.config.js 文件

一个可参考的配置如下，另附常用配置释义：https://blog.csdn.net/BigChicken3/article/details/107151209/

.stylelintrc.js
```js
module.exports = {
  root: true,
  extends: [
    'stylelint-config-standard',
    'stylelint-config-standard-scss',
    'stylelint-config-recommended-vue',
    'stylelint-config-rational-order',
    'stylelint-config-prettier'
  ],
  plugins: ['stylelint-order', 'stylelint-scss'],
  // https://stylelint.docschina.org/user-guide/rules/
  rules: {
    indentation: [2, { severity: 'warning' }],
    'number-leading-zero': null,
    'declaration-colon-space-after': 'always-single-line',
    'declaration-block-no-redundant-longhand-properties': null,
    'no-descending-specificity': null,
    'no-empty-source': null,
    'no-duplicate-selectors': null,
    'selector-type-no-unknown': null,
    'selector-descendant-combinator-no-non-space': null,
    // 允许嵌套的深度最多 10 层
    'max-nesting-depth': 10,
    // 防止::deep报错
    'selector-pseudo-element-no-unknown': null,
    // 防止类似@mixin报错
    'at-rule-no-unknown': null,
    // 支持rpx
    'unit-no-unknown': [true, { ignoreUnits: ['rpx'] }],
    // 支持import引入scss
    'scss/at-import-partial-extension': ['always', { except: ['scss'] }],
    'selector-class-pattern': null,
    'font-family-no-missing-generic-family-keyword': null
  }
}
```

#### 三、使用

package.json里加上命令
```bash
"scripts": {
    "lintcss": "stylelint src/**/*.{scss,vue} --fix"
}
```

#### 四、scss

##### 1、支持scss
~~上面的命令只能支持校验css文件，由于项目中使用到的是scss，所以需要另外安装postcss-scss，并使用customSyntax，如下：~~

```bash
npm i postcss-scss -D
```

~~package.json里命令修改为~~
```bash
"scripts": {
    "lintcss": "stylelint src/**/*.{scss,vue} --fix --custom-syntax postcss-scss"
}
```

##### 2、避坑

** 划重点划重点划重点 **

一番实验下来，发现这样配置后发现：vue文件里除了scss和template代码块，还有script代码块，如果此时用 stylelint 校验，会同时校验script代码，嗷~~~
所以果断舍弃 postcss-scss，换成 postcss-html，修改后配置如下：

```bash
npm i postcss-html -D
```
package.json里命令修改为
```bash
"scripts": {
    "lintcss": "stylelint src/**/*.{scss,vue} --fix --custom-syntax postcss-html"
}
```

#### 五、配置规则

除了最基础的stylelint包，我们还需要如下库

```bash
npm i postcss-scss -D // 不能省略
npm i postcss -D
npm i stylelint-config-standard -D // stylelint的推荐配置
npm i stylelint-order -D // 格式化css文件时对代码的属性进行排序
npm i stylelint-config-standard-scss -D // stylelint的推荐scss配置
npm i stylelint-config-recommended-vue -D
npm i stylelint-config-prettier -D
npm i stylelint-config-rational-order -D
npm i stylelint-scss -D
```

可供参考的配置

```bash
    "postcss-html": "^1.5.0",
    "postcss-scss": "^4.0.4",
    "postcss": "^8.4.14",
    "stylelint": "^14.9.1",
    "stylelint-config-prettier": "^9.0.3",
    "stylelint-config-rational-order": "^0.1.2",
    "stylelint-config-recommended-vue": "^1.4.0",
    "stylelint-config-standard": "^23.0.0",
    "stylelint-config-standard-scss": "^4.0.0",
    "stylelint-order": "^5.0.0",
    "stylelint-scss": "^4.3.0",
    "stylelint-webpack-plugin": "^3.3.0",
```

#### 六、忽略lint文件

同样有三种方式可以配置忽略文件，如果是忽略目录或者文件，推荐方法2

+ .stylelintrc.js中配置ignoreFiles
+ .stylelintignore 文件
+ 在代码块上加 /* stylelint-disable */

.stylelintignore
```bash
src/uni_*
src/components/uni-*
src/wxcomponent
```

#### 七、自动格式化

据说vscode里有stylelint的插件，安装后可以实现快捷键快速格式化~，然而对于webstorm用户，也有办法：

##### 1、可以在webpack里配置，热更新的时候自动stylelint格式化

此时就需要用到库 stylelint-webpack-plugin

```bash
npm i stylelint-webpack-plugin -D
```

vue.config.js
```bash
module.exports = {
  configureWebpack: {
    plugins: [
      new StyleLintPlugin({
        files: ['src/**/*.{vue,css,sass,scss}'],
        fix: false, // 是否自动修复，自动修复并不能修复全部问题，所以前期建议设置成false，自己手动解决
        cache: true
      })
    ]
  }
}
```

##### 2、在webstorm里配置 External tools

请参照：https://stackoverflow.com/questions/54304313/stylelint-fix-in-webstorm

#### 八、答疑解惑
+ 1、npm run lintcss并不能递归格式化src目录下所有的vue和css文件？
  答：看了官方文档，递归stylelint src目录下所有的vue和css文件的时候，这样配置stylelint src/**/*.{scss,vue}，但实践发现，只能找到 src/xxx/xxx.{vue,css}，并不能按照我们预期递归src目录下所有的vue和css文件。
     好在步骤7.1里，可以配置stylelint-webpack-plugin，这样在npm run dev的时候就会先让stylelint通过之后再继续执行。
