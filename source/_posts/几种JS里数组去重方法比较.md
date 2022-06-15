---
title: 几种JS里数组去重方法比较
date: 2019-09-20
categories: 前端
tags: 
    - 前端
    - JS
#description: 
---

昨天面试被问到JS里数组去重的方法，于是把最常用的for + object方法说了（当时本来准备说es6的Set去重的，结果突然忘记Set了）。
<!-- more -->
完事在网上搜了下去重的方法，发现远不止当时想到的两种，找了几种常用的方法，做了下执行效率对比，发现在处理大数据量的时候，竟然出现了将近一千倍的差距。为之震惊...

#### 一、常用8种方法执行效率对比（总15w条数据）
1.method.js
```js
﻿let obj = {
	// filter + indexOf
	distinct1: (a, b) => {
		console.log('filter + indexOf')
		let arr = a.concat(b);
		return arr.filter((item, index) => {
			return arr.indexOf(item) === index;
		});
	},
	// filter + object
	distinct2: (a, b) => {
		console.log('filter + object')
		let temp = {};
		let arr = a.concat(b);
		return arr.filter(item => {
			temp[item] = temp[item] || 0;
			temp[item] ++;
			return temp[item] === 1;
		});
	},
	// for + object
	distinct3: (a, b) => {
		console.log('for + object')
		let arr = a.concat(b);
		let temp = {};
		let res = [];
		for(let i = 0; i < arr.length; i++) {
			if(!temp[arr[i]]) {
				res.push(arr[i]);
				temp[arr[i]] = true;
			}
		}
		return res;
	},
	// for + indexOf
	distinct4: (a, b) => {
		console.log('for + indexOf')
		let arr = a.concat(b);
		let res = [];
		for(let i = 0; i < arr.length; i++) {
			if(res.indexOf(arr[i]) < 0) {
				res.push(arr[i]);
			}
		}
		return res;
	},
	// for of + includes
	distinct5: (a, b) => {
		console.log('for of + includes')
		let arr = a.concat(b);
		let res = [];
		for(let i of arr){
			!res.includes(i) && res.push(i);
		}
		return res;
	},
	// for of + object
	distinct6: (a, b) => {
		console.log('for of + object')
		let arr = a.concat(b);
		let res = [];
		let obj = {};

		for(let i of arr) {
			if(!obj[i]) {
				res.push(i);
				obj[i] = true;
			}
		}
		return res;
	},
	// Array.sort
	distinct7: (a, b) => {
		console.log('Array.sort')
		let arr = a.concat(b);
		arr.sort((a, b) => { return a > b ? 1 : -1; });
		let res = [];

		for(let i = 0; i < arr.length; i++) {
			arr[i] != arr[i - 1] && res.push(arr[i]);
		}

		return res;
	},
	// Set
	distinct8: (a, b) => {
		console.log('Set')
		return Array.from(new Set([...a, ...b]));
	}
};

module.exports = obj;
```
2.main.js（len1+len2为数组总长度）
```js
let len1 = 100000;
let len2 = 50000;
let dist = require('./method.js');

let arr = Array.from(new Array(len1), (x, index) => {
	return index;
});
let dis = Array.from(new Array(len2), (x, index) => {
	return index + index;
});
let times = Object.keys(dist); // 方法数量

for(let i of times) {
	let start = new Date().getTime();
	let res = dist[i](arr, dis);
	console.log('----------end---------------', res.length);
	let end = new Date().getTime();
	console.log('总耗时：', end - start);
}
```
3.执行结果分析
```js
filter + indexOf
----------end--------------- 100000
总耗时： 10608

filter + object
----------end--------------- 100000
总耗时： 14

for + object
----------end--------------- 100000
总耗时： 14

for + indexOf
----------end--------------- 100000
总耗时： 10471

for of + includes
----------end--------------- 100000
总耗时： 10501

for of + object
----------end--------------- 100000
总耗时： 12

Array.sort
----------end--------------- 100000
总耗时： 91

Set
----------end--------------- 100000
总耗时： 46
```

惊讶的发现，执行时间最短的依次是：for...of + object <  for + object = filter + object < Set < Array.sort < for + indexOf < for...of + includes < filter + indexOf。
并且最长的执行时间 10608ms 竟然比最短的 12ms 多了将近1000倍（仅仅15w条数据的情况下）。
粗略的总结下规律：
    1.循环中用对象属性来判断要比数组查找快的多（filter+indexOf和filter+object对比可以看出；以及for+object和for+indexOf对比可以看出）
    2.数组查找indexOf和includes执行效率差不多（for+indexOf和for...of+includes可以看出）
    3.数组循环for...of和for差不多（for+object和for...of+object可以看出）
    4.Array.sort和Set比基本2倍快

为了更明显的看出差别，我们把测试数据量扩大一个数量级（150w）再测一次，测试结果如下：
```js
filter + object
----------end--------------- 1000000
总耗时： 129

for + object
----------end--------------- 1000000
总耗时： 109

for of + object
----------end--------------- 1000000
总耗时： 121

Array.sort
----------end--------------- 1000000
总耗时： 909

Set
----------end--------------- 1000000
总耗时： 718
```
你可能发现了，测试结果里没有filter+indexOf、for+indexOf、for...of+includes，是因为150w条数据的时候，这几个方法直接卡死不动了（跟15w条的结果比，不止是10倍的关系），尴尬...
再次对比结果发现：
    1.再次印证了上面中的规律1（indexOf要比对象属性慢得多）
    2.for+object要比for...of+object稍快，filter+object次之
    3.Set比Array.sort快，但到不了第一种情况下的2倍差距

#### 二、结果分析
从两次测试不难发现，在实际项目中，尽量用for+object（或者for...of+object和filter+object）；如果为了代码简洁，可以用es6的Set，在实际运用中，数据量不大的情况，差别还是可以接受的；千万不能用indexOf或者includes，简直就不是一个数量级。

** 突然想到刚毕业时做.net，一次老大开会指出『千万不能把数据库数据都读到内存中操作，而应先从数据库中筛选出符合条件的数据，然后放到内存中操作，否则大数据量的时候可能会把服务器搞崩』。当时还不以为然，也没做实际对比。今天通过上面的实际测量，才发现差距之大，不由得汗颜，之前写代码的随性...仅以此文告诫自己吧！！！
