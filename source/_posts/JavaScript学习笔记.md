---
title: JavaScript学习笔记
date: 2017-03-05
categories: JS
tags: 
    - JS
#description: 
---

JS的一些学习笔记
<!-- more -->

### 一、ECMAScript 类型：
+ 原始类型：Undefined,Null,Bollean,Number,String
+ 引用类型：Object

### 二、函数：
```js
function a(){} //定义,不自动执行
```
```js
var a = function(){} //定义,不自动执行
```
```js
var a = function(){}(); //立即执行
注意：function a(){}(); //错误的
```
```js
(function a(){}）(); //立即执行
```
```js
(function(){})(); //同上一样,立即执行
```
```js
(function(){}()); //上边的”美观“写法,立即执行
```
```js
!function(){}(); //立即执行,装B写法
```
```js
+function(){}(); //立即执行,装B写法
```
```js
true && function(){}(); //立即执行,装B写法
```

### 三、函数闭包：
```js
var Circle = function(r){
	this.r = r;
}
Circle.PI = 3.1415926;
Circle.prototype.area = funtion(){
	return this.PI * this.r * this.r;
}
调用：
var c = new Circle(1.0); 
alert(c.area()); 
或者:
	(new Circle()).area(1.0);
```
```js
var Circle = function(){ 
	var obj = {};
	obj.PI=3.1415926;
	obj.area=function(r){
		return this.PI * r * r;
	}
	return obj;
}
或者：
var Circle = function(){ 
	return{ 
		PI:3.1415926,
		area:function(r){
			return this.PI * r * r;
		}
	}
}
调用：
var c = new Circle(1.0); 
alert(c.area());
或者:
	 (new Circle()).area(1.0);
改进版：
var Circle = (function(){ 
	var obj = {};
	obj.PI=3.1415926;
	obj.area=function(r){
		return this.PI * r * r;}return obj;
	}());
调用： 
Circle.area(1);
```
```js
var Circle = new Object();  
Circle.PI = 3.14159;  
Circle.Area = function(r) {  
	return this.PI * r * r;  
} 
调用：
alert( Circle.Area( 1.0 ) ); 
```
```js
var Circle={
	"PI":3.1415926,
	"area":function(r){
		return this.PI * r * r
	}
}
调用：
alert(Circle.Area(1.0));
```
```js
var Circle = new Function("this.PI=3.1415926;this.area=function(r){return r * r * this.PI}");
调用：
alert((new Circle()).area(1.0));
```

### 四、函数的prototype和直接赋值的方法的区别：
```js
var ff = function(){ 
	this.PI = 3.1415926; 
}
ff.showD = function(){ 
	alert(this.PI); 
}
ff.prototype.showE = function (){ 
	alert(this.PI); 
}
调用：
ff.showD(); //undefined
(new ff()).showE(); //3.1415926
错误写法 :
ff.showE(); //报错
(new ff()).showD(); //报错
```
解释：
1、不使用prototype属性定义的对象方法，是静态方法，只能直接用类名进行调用！另外，此静态方法中无法使用this变量来调用对象其他的属性！
2、使用prototype属性定义的对象方法，是非静态方法，只有在实例化后才能使用！其方法内部可以this来引用对象自身中的其他属性！

### 五、直接用in判断对象是否有该属性
```js
var dd = {"aa":"1","bb":"2"}
"aa" in dd //true;
"cc" in dd //false;
```

### 六、js方法里面，带空格的处理
动态拼接html的时候，如以下方法会报错
```HTML 
<a id="test" onclick=fun(" 你好 有空格","en这个没空格")></a>
```
+ escape 函数 和unescape函数
+ 不把参数拼接在html中的函数中，拼为该元素的属性，如：

```HTML
<a id="test" data-name=" 你好 有空格" onclick="fun(this)"></a>
```

### 七、for in方法既能循环数组也能循环对象,同 jQuery 的 $.each  
```js
var a = {"a":"a","b":"b","c":"c"},b = ["a","b","c"];
for(var t in a){console.log(t);} //输出 0,1,2
for(var t in b){console.log(t);} //输出 a,b,c
for(var t in a){console.log(a[t]);} //输出 a,b,c
for(var t in b){console.log(a[t]);} //输出 a,b,c

$.each(a,function(k,v){console.log(k+","+v);})//输出 a,a ， b,b ，c,c
$.each(b,function(k,v){console.log(k+","+v);})//输出 0,a ， 1,b ，2,c 
```
*备注：但是直接用下标取值的时候对象不行；能 b[0],不能 a[0]*

** 拓展：**

1.原生JS循环数组还能用 forEach (注意：输出参数和$.each的参数输出相反)

```js
a = ["a","b","c"],
b = ["a", "b", "c"];
a.forEach(function(k,v){
	console.log(k+","+v)
})
// 输出 a,0 ， b,1 ，c,2 
```
2.$.each后面还能加参数，如果加入参数，则输出的是 自定义的参数，用this输出当前循环的对象
```js
$.each(b,function(k,v){console.log(k+","+v);}) //输出 0,a ， 1,b ，2,c
$.each(b,function(k,v){console.log(k+","+v);},["1","2"]) //输出 1,2 ，1,2 ，1,2
$.each(b,function(k,v){console.log(this +","+ k+","+v);}) //输出 a,0,a ， b,1,b ，c,2,c
```
具体参考：http://www.cnblogs.com/mabelstyle/archive/2013/02/19/2917260.html

### 八、js四种取整方法的比较 
Math.floor VS Math.round VS parseInt VS |0 VS >>0 VS ~~ VS >>>0
parseInt最慢，其他差不多（Math.round 是四舍五入，其他方法都是直接取整）
详情参照：<http://jsperf.com/math-floor-vs-math-round-vs-parseint/42>
