---
title: 原生JS的debounce和throttle
date: 2017-03-05
categories: JS
tags: 
    - JS
#description: 
---

说说原生JS中的debounce和throttle

<!-- more -->

### 适用场景举例
- input输入框自动补全
- windowresize事件
- 跟踪鼠标
- DOM 元素动态定位

### 方法封装
```JS
/*
* 频率控制 返回函数连续调用时，fn 执行频率限定为每多少时间执行一次
* @param fn {function}  需要调用的函数
* @param delay  {number}    延迟时间，单位毫秒
* @param immediate  {bool} 给 immediate参数传递false 绑定的函数先执行，而不是delay后后执行。
* @return {function}实际调用函数
*/
var throttle = function (fn,delay, immediate, debounce) {
  var curr = +new Date(),//当前事件
  last_call = 0,
  last_exec = 0,
  timer = null,
  diff, //时间差
  context,//上下文
  args,
  exec = function () {
    last_exec = curr;
    fn.apply(context, args);
  };
  return function () {
    curr= +new Date();
    context = this,
    args = arguments,
    diff = curr - (debounce ? last_call : last_exec) - delay;
    clearTimeout(timer);
    if (debounce) {
      if (immediate) {
        timer = setTimeout(exec, delay);
      } else if (diff >= 0) {
        exec();
      }
    } else {
      if (diff >= 0) {
        exec();
      } else if (immediate) {
        timer = setTimeout(exec, -diff);
      }
    }
    last_call = curr;
  }
};
     
/*
* 空闲控制 返回函数连续调用时，空闲时间必须大于或等于 delay，fn 才会执行
* @param fn {function}  要调用的函数
* @param delay   {number}    空闲时间
* @param immediate  {bool} 给 immediate参数传递false 绑定的函数先执行，而不是delay后后执行。
* @return {function}实际调用函数
*/
var debounce = function (fn, delay, immediate) {
  return throttle(fn, delay, immediate, true);
};
```

### 举个例子
html代码：
```HTML
<input id="ipt"/>
```
js代码：
```JS
var ipt = document.getElementById('ipt');
var ipt_input = throttle(function(){
  console.log(this.value + ':' + new Date());
},2000, true, true);
window.onload = function(){
  addLister(ipt,'input',ipt_input);
}
function addLister(el,e,h){
  if(!el) return false;
  return el.addEventListener 
  && el.addEventListener(e,h,true) 
  || el.attachEvent && el.attachEvent(e,h);
}
function rmListener(el,e,h){
  if(!el) return false;
  return el.addEventListener 
  && el.removeEventListener(e,h,true) 
  || el.detachEvent && el.detachEvent(e,h);
}
```

### ** 拓展知识 **
```JS
var resizeTimer=null;
$(window).on('resize',function(){
  if(resizeTimer){
    clearTimeout(resizeTimer)
  }
  resizeTimer=setTimeout(function(){
    console.log("window resize");
    },400);
  }
);
```
