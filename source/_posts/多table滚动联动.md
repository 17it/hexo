---
title: 多table滚动联动
date: 2021-12-10
categories: JS
tags: 
    - JS
#description: 
---

在做一个需求，左右两个table需要滚动联动，你侬我侬（you scroll, i scroll）。
<!-- more -->
### 初实现
* 对于这个需求，第一想法就是分别监听两个table的scroll事件，然后在scroll事件里分别给对方设置scrollTop值。
```js
this.dom = this.$refs.tbl.$el.querySelector('.fw-scroll-body')
this.affixDom = this.$refs.affix_tbl.$el.querySelector('.ant-table-body')
this.dom.addEventListener('scroll', () => {
  this.affixDom.scrollTop = this.dom.scrollTop
}, true)
this.affixDom.addEventListener('scroll', () => {
  this.dom.scrollTop = this.affixDom.scrollTop
}, true)
```
一顿操作猛如虎，mac运行流畅，都准备开酒庆祝了，然而，用windows上的chrome一测，滚动卡的一批~
### 问题分析
* 最先想到的是不是windows版的chrome在频繁处理scroll事件，遂加上debounce 100ms，发现有所改善，但是两个table会出现不太同步滚动的现象。
* 作为一个精益求精的码农(其实是没能说服产品)，开始思考还有没有更流畅的体验。几经折腾，开始怀疑是不是频繁处理scroll导致的卡顿，试着注掉一个table的scroll事件，发现顿时流畅了。在两个scroll事件里加上console，滚动表格，发现两个事件都在执行。综合上述两种现象，发觉导致卡顿的根本原因是滚动的时候两个table来回循环给对方赋值scrollTop导致scrollTop一直不会大变。
### 解决方案
判断鼠标当前所在位置属于哪个table，不是当前table的时候不触发当前table的滚动事件，两个思路。

1、实时获取鼠标位置，判断鼠标在哪个table区域内（舍弃，每次鼠标移动，都涉及到位置计算，不太优）

2、监听两个table的mouseover和mouseout事件，来改变标志位
```js
this.whereIsMouse = '' // 鼠标所在位置
this.dom = this.$refs.tbl.$el.querySelector('.fw-scroll-body')
this.affixDom = this.$refs.affix_tbl.$el.querySelector('.ant-table-body')

this.dom.addEventListener('scroll', () => {
  // 判断鼠标是否在dom表格内，如果在，则触发该滚动事件
  if (this.whereIsMouse === 'dom') {
    this.affixDom.scrollTop = this.dom.scrollTop
  }
}, true)
this.affixDom.addEventListener('scroll', () => {
  if (this.whereIsMouse === 'affixDom') {
    this.dom.scrollTop = this.affixDom.scrollTop
  }
}, true)

this.dom.addEventListener('mouseover', () => { this.whereIsMouse = 'dom' }, true)
this.dom.addEventListener('mouseout', () => { this.whereIsMouse = '' }, true)
this.affixDom.addEventListener('mouseover', () => { this.whereIsMouse = 'affixDom' }, true)
this.affixDom.addEventListener('mouseout', () => { this.whereIsMouse = '' }, true)
```

+ 方向不对，努力白费~
