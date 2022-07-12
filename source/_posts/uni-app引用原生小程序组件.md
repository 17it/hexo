---
title: uni-app引用原生小程序组件
date: 2022-07-12
categories: UniApp 小程序
tags: 
    - 小程序
    - UniApp
#description: 
---

uni-app引用原生小程序组件
<!-- more -->

uni-app 支持在 App 和 小程序 中使用小程序自定义组件，从HBuilderX2.4.7起，H5端也可以运行微信小程序组件。

#### 零、需求背景

在做小程序直播时，需要用到阿里云提供的直播sdk，阿里云官方有给一个小程序的demo，但用的是小程序原生写法，需要集成到Uni-app项目里。

#### 一、引入

+ 在pages同级新建目录wxcomponents（此目录名字固定不能变），把原生小程序代码放到该目录下
+ 在pages.json里对应页面的style->usingComponents引入组件
```bash
"style": {
    "usingComponents": {
        "meet": "/wxcomponents/meeting/meeting"
    }
}
```
+ 在页面中进行引用
```bash
<view class="">
  <meet></meet>
</view>
```

到此，感觉岁月静好，收拾电脑都准备下班了~ 然而，仍然用不了~

#### 二、完善

翻阅了微信小程序文档发现，页面和组件的写法不太一样，由于阿里云官方提供的demo是页面形式，而这里是以组件的形式引用，所以需要做对应修改：

+ Page -> Component
+ 原先基于页面的生命周期钩子改成对应组件的生命周期钩子

** 微信小程序页面生命周期：https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/page.html
** 微信小程序组件生命周期：https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/lifetimes.html
