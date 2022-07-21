---
title: eslint实践
date: 2022-07-08
categories: NODE JS
tags: 
    - NODE
    - JS
#description: 
---

eslint实践
<!-- more -->

#### 一、安装

```bash
npm i eslint --D
```

#### 二、配置文件

一个可参考的配置如下（注：由于项目是基于Uni-app的小程序项目，所以globals里需要加上uni、wx）

.eslintrc.js
```js
/* eslint-disable */
module.exports = {
    root: true,
    env: {
        browser: true,
        node: true,
        es6: true
    },
    globals: {
        uni: 'readonly',
        plus: 'readonly',
        wx: 'readonly',
    },
    extends: ['plugin:vue/recommended', 'standard'],
    parserOptions: {
        parser: '@babel/eslint-parser',
        sourceType: 'module'
    },
    //自定义规则，当与基础规则发生冲突时，覆盖基础规则
    //"off"或0-关闭规则;
    //"warn"或1-开启规则，使用警告级别的错误;
    //"error"或2-开启规则，使用错误级别的错误
    rules: {
        // 每行最大属性个数，如果是多行，每行最多一个属性
        'vue/max-attributes-per-line': [
            'error',
            {
                singleline: 10,
                multiline: {
                    max: 1
                }
            }
        ],
        // 忽略html标签自闭合 vue中html自终止标签是否写‘/’
        'vue/html-self-closing': 'off',
        'no-undef': 'off',
        // 关闭禁用混合操作符
        'no-mixed-operators': 'off',
        // 单行html元素内容在新的一行
        'vue/singleline-html-element-content-newline': 'off',
        // 多行html元素内容在新的一行
        'vue/multiline-html-element-content-newline': 'off',
        // 关闭v-html的校验，因为我们可能会用到富文本编辑器
        'vue/no-v-html': 'off',
        'vue/multi-word-component-names': 'off',
        'vue/component-definition-name-casing': 'off',
        'vue/html-closing-bracket-newline': [
            'error',
            {
                singleline: 'never',
                multiline: 'always'
            }
        ],
        // 一行内容最长有多少字符
        'max-len': ['error', { code: 400 }],
        //函数名称或function关键字与开始参数之间允许有空格:关 默认为开 与prettier冲突
        'space-before-function-paren': ['error', 'never'],
        //语句末尾不允许有';'：开
        semi: ['error', 'never'],
        //不允许使用var
        'no-var': 2,
        // 禁用tab
        'no-tabs': 'off',
        //是否允许promise中reject()内无内容
        'prefer-promise-reject-errors': ['error', { allowEmptyReject: true }]
        // 不允许对象嵌套对象的大括号之间有空格 例如：{ query: { id: row.id }[这里]}
        // 某些框架下会使用下边的配置，比如elementUI-admin
        // 'object-curly-spacing': [
        //   2,
        //   'always',
        //   {
        //     objectsInObjects: false
        //   }
        // ]
    }
}
```

#### 三、使用

package.json里加上命令
```bash
"scripts": {
    "lint": "eslint --fix --ext .js,.vue ./src"
}
```

#### 四、配置规则

除了最基础的eslint包，我们还需要如下库

```bash
npm i @babel/eslint-parser -D
npm i eslint-config-standard -D
npm i eslint-plugin-standard -D
npm i eslint-plugin-vue -D
npm i eslint-plugin-import -D
npm i eslint-plugin-node -D
npm i eslint-plugin-promise -D
```

以下版本可以作为参考：
```bash
    "@babel/eslint-parser": "^7.18.9",
    "eslint": "^8.20.0",
    "eslint-config-standard": "^16.0.3",
    "eslint-plugin-import": "^2.26.0",
    "eslint-plugin-node": "^11.1.0",
    "eslint-plugin-promise": "^6.0.0",
    "eslint-plugin-standard": "^5.0.0",
    "eslint-plugin-vue": "^9.2.0",
```

#### 五、忽略lint文件

.eslintignore
```bash
# 忽略打包的文件
dist
# 忽略uni-app官方的组件库错误和警告，官方的竟然通不过...
src/components/uni-lechart
src/components/uni-echarts
src/uni_modules
src/utils/aliyun-upload-sdk-1.0.0.min.js
src/utils/aliplayercomponents-1.0.8.min.js
src/utils/aliyun-webrtc-miniapp-sdk.js
src/utils/html-parser.js
/src/common/permission.js
src/wxcomponents
src/miniprogram_npm
src/utils/crypto
```

#### 六、自动格式化

##### webstorm
+ 1、在Preferences -> Languages & Frameworks -> JavaScript -> Code Quality Tools -> ESLint里启用
+ 2、在代码里右键，Fix Eslint Problems（还可以自己设置快捷键进行格式化）

#### 七、答疑解惑
+ 1、执行命令 npm run lint时报错"ESLint couldn't find the plugin "eslint-plugin-n"."?
  答：npm i eslint-config-standard -D的时候，安装的版本是17.x，降级成16.x即可（可以参照上面给出的配置）
+ 2、可能在很多现成的脚手架里面会看到同时使用了eslint和prettier，是否有必要？
  答：通过试验，可以不要prettier。至于为什么要同事使用两者，原因如下：（参考：https://www.jianshu.com/p/dd07cca0a48e）
  <img src="/eslint实践/eslint_prettier.png" width="50%">