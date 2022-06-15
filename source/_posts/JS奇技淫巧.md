---
title: JS奇技淫巧
date: 2017-03-05
categories: JS
tags: 
    - JS
#description: 
---

一些js的常用技巧
<!-- more -->

#### if(){}else if(){}else... 太复杂？switch代码太多？试试 && 和 object
```js
var add_level = (add_step==5 && 1) 
		|| (add_step==10 && 2) 
		|| (add_step==12 && 3) 
		|| (add_step==15 && 4) 
		|| 0; //通过&&运算符来取值
var add_level={'5':1,'10':2,'12':3,'15':4}[add_step] || 0; //通过对象的属性来取值
```

** && 符号拓展 ：**
1.参与方法执行   
```js
flag == true && function(){ 
	//do something ; 
}(); 
// 相当于 
if(flag == true){ 
	function(){
	// do something;
	} 
}
```
2.取值：
```js
var a = {};
a.name = "n"; 
var b = a && a.name ; // 相当于 if(a && a.name){ b = a.name ;}
var b = a.name && a ; // 相当于 if(a && a.name){ b = a;}
var b = a && a.age ; // b = undefined,相当于 if(a && a.age){ b = a.age; }
```

#### 转成相应进制，如14进制（注意两者的区别）
1.parseInt(string,radix)
```js
parseInt("14sad",14) = 18; // 基数为14进制，结果即 1*14的一次方 + 4*14的0次方 = 18
```
2.parseInt().toString(radix)
```js
parseInt("14das").toString(14) = “10”; // 转成14进制后的字符串
```

#### JS中0、""、null、false、undefined、NaN 都会判为false，其他都为true
+ 运用 || 或 && 的时候特别注意0，如：

```js
a = 0；b = a || 3；结果 b = 3; 
// 因为会认为 a 是 false ；
```
+ 再比如 ：

```js
c = a && alert("有值"); // 结果不会弹框
```

** 拓展：**
如果要判断是否有值，需要排除值为0的情况，可以  
```js
if(!(a === null || typeof a == "undefined" || a == "")){ alert("有值"); }
// 注意是全等 === null
```
