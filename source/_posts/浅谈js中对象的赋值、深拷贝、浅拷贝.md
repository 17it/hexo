---
title: 浅谈js中对象的赋值、深拷贝、浅拷贝
date: 2018-10-08
categories: JS
tags: 
    - JS
#description: 
---

在js中，万物皆对象...
<!-- more -->

所以日常编码中会经常跟对象打交道，一个比较常用的案例就是对象的拷贝，下面就遇到的一些拷贝问题做简单解析，随时补充。

## 一、基本数据类型 的拷贝
基本数据类型（Number,String,Boolean,Undefined,Null）的拷贝就很简单了，直接用 = 就行，无论你什么姿势，都OK。
```js
// Number
a = 1.1;b = a;b = 2; console.log(a,b)
// String
a = 'hello';b = a;b = 3; console.log(a,b)
// Boolean
a = false;b = a;b = 'sss'; console.log(a,b)
// Undefined
a = undefined;b = a;b = false; console.log(a,b)
// Null
a = null;b = a;b = undefined; console.log(a,b)
```
经过实践我们会发现，无论如何修改b的值，a还是一动不动，保持初心。

## 二、复杂数据类型（object） 的拷贝
常用的复杂数据类型包括：{} 、[] 、function(){} 、Date 、RegExp 、null（这个比较特殊）等

#### 1、我们依然用一的简单赋值（=）来进行一遍操作（赋值）
```js
// {}
a = {name: 'abc'};b = a;b.name = 'sss'; console.log(a,b)
// []
a = ['a','b','c'];b = a;b[1] = 'd'; console.log(a,b)
// function
a = function(){ alert('aaa'); };b = a;b = function(){ alert('bbb'); }; console.log(a.toString(),b.toString())
// Date
a = new Date('2018-10-11 00:00:00');b = a;b = new Date('1970-01-01 00:00:00');console.log(a,b)
// RegExp
a = new RegExp('abc');b = a;b = new RegExp('aaa'); console.log(a,b)
```
经过实践我们会发现：
1、当类型为{}、[]的时候，改变b的值，a也会跟着一起变化。
2、当类型为Date、function、RegExp的时候，a保持不变。
总结：
我们发现{}或者[]时，简单的赋值操作并不能实现它们的拷贝，只是改了b的指向，使a和b都指向同一个引用，随意改变一个，都会影响另外一个的值。

#### 2、Object.assign 和 for in进行{}和[]的拷贝（浅拷贝）
```js
// Object.assign
a = {name: 'aaa'};b = Object.assign({}, a);b.name = 'bbb';  console.log(a,b)
a = [1,2,3];b = Object.assign([], a);b[1] = 4;  console.log(a,b)
// for in
var copy = function(a) {
 var res = a.constructor();
 for(var key in a) {
  if(a.hasOwnProperty(key)) {
    res[key] = a[key];
  }
 }
 return res;
}
a = {name: 'aaa'};b = copy(a);b.name = 'bbb';  console.log(a,b)
a = [1,2,3];b = copy(a);b[1] = 4;  console.log(a,b)
```
我们惊奇的发现，貌似这俩方法挺好使，不禁心中暗自惊喜。但是如果我们进一步测试，把a改的更复杂点。
a = {name:'aaa',people:{name: 'abc'}};b = Object.assign({}, a);b.people.name = 'def';  console.log(a,b)
a = {name:'aaa',people:{name: 'abc'}};b = copy(a);b.people.name = 'def';  console.log(a,b)
a = [1,2, {name: 'aaa'}];b = Object.assign([], a);b[2].name = 'bbb';  console.log(a,b)
a = [1,2, {name: 'aaa'}];b = copy(a);b[2].name = 'bbb';  console.log(a,b)
哇喔，这样就嗝屁了。

#### 3、使用JSON.parse（）与JSON.stringify（）对对象进行拷贝（深拷贝，不完美）
```js
var deepCopy = function(a) {
  return JSON.parse(JSON.stringify(a));
}
a = {name:'aaa',people:{name: 'abc'}};b = deepCopy(a);b.people.name = 'def';  console.log(a,b)
a = [1,2, {name: 'aaa'}];b = deepCopy(a);b[2].name = 'bbb';  console.log(a,b)
```
酱紫试验了下，貌似还不错，但仔细研究就会发现：
1.上述的方法会忽略值为function以及undefined的字段，而且对date类型的支持也不太友好
a = {name:'aaa',fun:function(){console.log('fun');},nn: undefined};b = deepCopy(a); console.log(a.fun,b.fun) // b.fun会输出undefined,并且b.nn也会不存在
2.上述方法只能克隆原始对象自身的值，不能克隆它继承的值
function Person (name) {
    this.name = name
}
var a = new Person('王二');
var b = deepCopy(a);
a.constructor == Person; // true
b.constructor == Object; // true

#### 4、for in实现比较完美的深拷贝方法（深拷贝，完美）
function realCopy(obj) {
  // 简单数据类型或者null直接return obj
  if (obj == null || typeof (obj) != 'object') { return obj; }

  var type = obj.constructor;
  // Date或RegExp也直接return对应值
  if(type == Date) { return new Date(obj); }
  if(type == RegExp) { return new RegExp(obj); }

  var temp = obj.constructor();
  for (var key in obj) {
    // 不遍历其原型链上的属性
    if (obj.hasOwnProperty(key)) {
      // 注意这里的递归调用
      temp[key] = realCopy(obj[key]);
    }
  }
  return temp;
}
至此，方法4算是比较完美的一个深拷贝方案了。