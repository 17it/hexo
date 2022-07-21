---
title: husky+lint-staged实现代码提交前校验
date: 2022-07-08
categories: GIT
tags: 
    - CI/CD
    - GIT
#description: 
---

husky+lint-staged实现代码提交前校验
<!-- more -->
    
依赖：此文依赖于《eslint实践》和《stylelint实践》，假设项目里已经加上了eslint和stylelint的功能~

#### 一、概念

 + husky: husky是一个git hook工具，可以帮助我们触发git提交的各个阶段：pre-commit、commitmsg、pre-push等
 + lint-staged: husky可以实现在pre-commit时校验代码，但整个项目校验会很慢，lint-staged能够让lint只检测暂存区的文件，所以速度很快

#### 二、husky

##### 1.安装husky
```bash
npm install husky -D
```

##### 2.下面命令第一次执行一次就行
```bash
npm set-script prepare "husky install" // npm set-script命令需要npm版本为7.x，如果npm版本不够，直接在package.json的script里加命令 "prepare": "husky install"
npm run prepare
```

##### 3.在package.json里面添加lint命令（下例中添加了eslint和stylelint）
```bash
"scripts": {
  "lint": "eslint --fix --ext .js,.vue ./src",
  "lintcss": "stylelint src/**/*.{scss,vue} --fix --custom-syntax postcss-html"
}
```

##### 4.初始化husky
```bash
npx husky add .husky/pre-commit "npm test"
```

执行完这步，我们会发现在项目根目录下会生存一个新的目录 .husky，并且目录下会生成一个文件 pre-commit，文件里有一条命令 npm test。
其实到此，我们已经完成了husky的安装和初始化，也就是现在我们就可以验证husky了。

```bash
git add .
git commit -m 'some message'
```
执行上面git命令，紧接着，我们在终端会看到在commit之前会先执行"npm test"。

##### 5.修改 .husky/pre-commit 里命令"npm test"为如下lint命令
```bash
npm run lint
npm run lintcss
```
再次执行git commit -m 'xxx'，此时会先npm run lint校验js，然后 npm run lintcss校验scss\css\sass等，如果有书写不规范，会提示失败，终止commit操作。

#### 三、lint-staged

上面已经实现了git commit之前对css和js进行书写规范校验，但是有个弊端：每次commit之前都会校验整个项目下的文件（当然排除.eslitignore和.stylelintignore里排除掉的），所以我们就需要借助 lint-staged，让其只校验本次有更改的文件，也就是本次git stage的文件。

##### 1.安装 lint-staged
```bash
npm install lint-staged --save-dev
```

_** 这里可能会出现错误提示『Cannot use import statement outside a module』，原因是因为默认安装的 lint-staged是 13.x版本，此版本需要nodejs版本为 12.20.x以上，而本地node版本为 12.14.1 **_
_** 对lint-staged降级即可，最后实验出node 12.14.1环境下可以用的版本为 11.x，参照：https://github.com/okonet/lint-staged/issues/1084 **_
```bash
npm install lint-staged@11 --save-dev
```

##### 2.package.json里配置 lint-staged
```bash
"lint-staged": {
  "*.{js,vue}": [
    "eslint --fix",
    "git add"
  ],
  "*.{scss,vue}": [
    "stylelint --custom-syntax postcss-html",
    "git add"
  ]
}
```

##### 3.修改 .husky/pre-commit 为如下命令
```bash
npx lint-staged
```

至此，再次git commit的时候，eslit和stylelint就只会校验本次修改的文件，大大提升校验速度。
