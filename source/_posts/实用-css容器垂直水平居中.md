---
title: css容器垂直水平居中
date: 2017-07-18
categories: 前端实用
tags: 
    - css
    - 垂直居中
#description: 
---

css容器垂直水平居中常用方法，持续更新中...
<!-- more -->

### 一、借助伪类after

+ 1、html
```html
<div class="out">
    <div>
        <div>头部</div>
        <div>中部</div>
        <div>尾部</div>
    </div>
</div>
```
+ 2、css
```CSS
.out {
    z-index: 2011;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    text-align: center;
    background: rgba(0, 0, 0, 0.5);
}
.out>div {
    text-align: left;
    display: inline-block;
    vertical-align: middle;
    background-color: #fff;
    width: 90%;
    border-radius: 3px;
    font-size: 16px;
    overflow: hidden;
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
}
.out:after {
    /* 最重要的是这步 */
    content: "";
    display: inline-block;
    height: 100%;
    width: 0;
    vertical-align: middle;
}
```

### 二、display:table

+ 1、html
```html
<div class="out">
    <div class="cell">
        <div class="content">
        	<div>头部</div>
            <div>中部</div>
            <div>尾部</div>
        </div>
    </div>
</div>
```
+ 2、css
```CSS
.out {
    display: table;
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    z-index: 100;
    background: rgba(0, 0, 0, 0.5);
}
.cell {
     /* 关键步骤 */
     display: table-cell;
     vertical-align: middle;
 }
 .content {
     background: #fff;
     width: 60%;
     margin: 0 auto;
 }
```

### 三、借助js

+ 1、html
```html
<div class="out">
    <div class="cell">
        <div>头部</div>
        <div>中部</div>
        <div>尾部</div>
    </div>
</div>
```
+ 2、css
```CSS
.out {
    z-index: 2017;
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.5);
    text-align: center;
}
.cell {
    position: relative;
    background: #fff;
    width: 60%;
    margin: 0 auto;
}
```
+ 3、js
```js
  window.onload = function () {
      var out = document.querySelector('.out');
      var cell = document.querySelector('.cell');
      var oh = out.offsetHeight;
      var ch = cell.offsetHeight;

      cell.style.top = (oh - ch) / 2 + 'px';
  }
```

### 四、flex
+ 1、html
```html
<div class="box">
     <section class="inner">我是内容</section>
</div>
```
+ 2、css
```css
.box {
    display: flex;
    justify-content: center;
    align-items: center;
}
.inner {
    margin: 50px 0;
}
```


持续更新中。。。



