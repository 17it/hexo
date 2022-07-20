---
title: uni-app项目往cli模式迁移
date: 2022-07-12
categories: UniApp 小程序
tags: 
    - 小程序
    - UniApp
#description: 
---

uni-app支持两种模式创建项目，HBuilderX和vue-cli脚手架。
<!-- more -->
因为初次用uni-app，所以一开始使用的是第一种方式创建。用着用着，慢慢发现第一种的不好之处了~

+ 由于HBuilderX开发工具的摆烂，所以开发的时候，都需要开HBuilderX、微信开发者工具、webstorm(或者vscode)三个IDE
+ 打包时只能用HBuilderX的build菜单操作，没办法注入环境变量，多个环境打包时，只能手动改代码，然后打包
+ 没办法做到打包、上传一条龙

自己欠的债，不管何时都得还~

#### 一、用vue-cli创建一个新项目

参照官方文档：https://uniapp.dcloud.io/quickstart-cli.html#install-vue-cli

##### 1、安装vue-cli
```bash
npm install -g @vue/cli@4
```

##### 2、新建项目
```bash
vue create -p dcloudio/uni-preset-vue my-uni
```

创建的时候模板选择『Hello uni-app』即可，其他模板应该也可以~

#### 二、迁移

+ 删除新项目下src目录下所有的目录
+ 把原先项目的目录都拷贝到新项目的src目录下（需要排除目录"node_modules"、"unpackage"）
+ 使用原项目下的同名文件替换新项目src下面的同名文件内容

#### 三、加eslint、stylelint、lint-staged

如果需要可以添加，分别参考《husky+lint-staged实现代码提交前校验.md》《eslint实践.md》《stylelint实践.md》

#### 四、环境区分

##### 1、修改打包命令

使用vue-cli新建的项目，我们可以在package.json里看到自带了很多的scripts命令，包括dev:xxx和build:xxx。由于只用到了小程序的开发、打包，所以我们可以删掉其他平台的相关命令，精简如下：

```bash
"scripts": {
  "build": "cross-env NODE_ENV=production UNI_PLATFORM=mp-weixin VUE_APP_ENV=production vue-cli-service uni-build && sh ./build/build.sh",
  "build:test": "cross-env NODE_ENV=production UNI_PLATFORM=mp-weixin VUE_APP_ENV=test vue-cli-service uni-build && sh ./build/build.sh",

  "dev": "cross-env NODE_ENV=development UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch",
  "test": "cross-env NODE_ENV=test UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch"
}
```

##### 2、加.env.xxx

新建 .env.development 文件
```bash
NODE_ENV=development
VUE_APP_BASE_API = https://dev.xxx.com
```

新建 .env.test 文件
```bash
NODE_ENV=test
VUE_APP_BASE_API = https://test.xxx.com
```

新建 .env.production 文件
```bash
NODE_ENV=production
VUE_APP_BASE_API = https://xxx.com
VUE_APP_BASE_API_TEST = https://test.xxx.com
```

##### 3、在项目中获取 VUE_APP_BASE_API

```bash
process.env.VUE_APP_ENV === 'test' ? process.env.VUE_APP_BASE_API_TEST : process.env.VUE_APP_BASE_API
```

+ 这一步可能会有疑惑，为什么要在.env.production里同时加正式和测试环境的BASE_API？
+ 是因为 uni-build命令只能区分NODE_ENV=production和其他值，也就是想要发布版本，只能在packages.json的打包命令里加上NODE_ENV=production。这样以来有个弊端，如果我想打包test环境，发布到预览版，我还需要手动改.env.production里的配置VUE_APP_BASE_API。
+ 所以，我们可以在打包命令里再加上一个字段VUE_APP_ENV，build命令里指定"VUE_APP_ENV=production"，build:test命令里指定"VUE_APP_ENV=test"，然后在项目取的时候判断下VUE_APP_ENV，如："process.env.VUE_APP_ENV === 'test' ? process.env.VUE_APP_BASE_API_TEST : process.env.VUE_APP_BASE_API"。
+ 这样就可以通过 npm run build 和 npm run build:test 分别来打包正式环境和test环境用来发布预览版和提审了。

#### 五、解疑答惑

+ 1.cli模式run dev比run build后的包大？
答：run dev的时候，没有对代码进行压缩
+ 2.开发的时候怎么使用手机进行预览？
问题描述：直接run dev出来的包很大，超过2M，不能预览。官方有说可以在dev命令里加参数--minimize，加上之后，发现每次热更新巨慢4分钟左右，也就是说：我修改了一个文案用了2秒，想在手机上预览看效果，得等上4分钟~
答：不加参数--minimize，在微信开发者工具里勾选『预览及真机调试时主包、分包体积上限调整为4M』（此处有个疑问：为啥hbuilder打包时，加上压缩后打包挺快？）
+ 3.项目里使用的uni三方插件"zb-table"和"lime-charts"报错？
答：zb-table报错"egenerator-runtime"的解决方案：https://article.itxueyuan.com/Owm5jb
+ 4.pages目录分包注意事项？
答：分包目录不能跟tarBar配置里的任意一项的目录重合，比如tarBar里有 pages/mine/mine，此时不能把pages/mine作为分包（解决办法就是把pages/mine/mine文件换个目录）
