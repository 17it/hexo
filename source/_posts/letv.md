---
title: letv
date: 2017-04-17
tags:
- dialry
---

### components里form和table新加的功能以及vue route
<!-- more -->

### 一、只url中的参数改变时，vue route的使用
+ 问题描述：vue单页应用中，点击左侧栏菜单，当仅url参数改变，路径不变时，默认不会走主组件（Logic）里面的create方法的，会导致页面不能刷新（或者获取新数据）。
+ 解决办法：可以在主组件（Logic）里面挂上vue route的钩子，在里面捕获参数的改变，然后改页面state里面的值，来触发对应的刷新（或者获取新数据）的方法。

#### 1.在页面里的主组件Logic里面挂上vue route的钩子函数 beforeRouteUpdate,当页面url中的参数变化时，会进入这里。在函数里面调用state里的方法 pageList_resetState。
<img src="http://onm9ileaw.bkt.clouddn.com/letv/11.png">
#### 2.在state.js里的方法pageList_resetState中改变面包屑 和 改变state的值（如pageNum,hasParamChange）.
<img src="http://onm9ileaw.bkt.clouddn.com/letv/22.png">
#### 3.在页面里对应的组件中监听state的值改变，再调用对应的ajax获取新数据。
<img src="http://onm9ileaw.bkt.clouddn.com/letv/33.png">

### 二、form组件扩展
#### 1.input后支持增加颜色选择器
+ 示例：
```js
{
    prop:’bgColor’,
    type:’colorPicker’,
    colorChange: function(){},
    colorPickerStyle: 'line-height: normal;position: absolute;font-size: 0;margin-left:10px;'
}
```
+ 效果图：
  <img src="http://onm9ileaw.bkt.clouddn.com/letv/1.png">

#### 2.form表单支持检测链接类型的输入框
+ 示例:
```js
var preObj = {
    key: 'movie',
    value: 'http://movie.le.com'
};

attributes:[
{
    prop: 'publishAddress',
    label: '发布地址',
    type: 'require_checkUrl',
    placeholder: '页面URL',
    style: 'width:370px;margin-right:10px;',
    preLabel: preObj
}]
```
+ 效果图：
  <img src="http://onm9ileaw.bkt.clouddn.com/letv/3.png">

### 三、table组件扩展
#### 1.table里按钮支持自定义样式
示例：
```js
attributes:[
{
    prop: '',
    title: '',
    value: 1,
    disabled: false,
    onclick: this.closeStatus,
    style: 'color:red'
}
]
```
#### 2.table里普通显示文字支持自定义样式
#### 3.table的custom类型支持传入disabled
示例：
```js
attributes:[{
    prop: 'modManage',
    label: '模块管理',
    type: 'ifelsetext',
    width: '100',
    options: [
        {
    	    prop: 'isEditing',
        	value: false,
            style: 'color:blue;cursor:pointer;text-align:center',
            onclick: this.manageModule
        },
        {
        	prop: 'isEditing',
            value: true,
            style: 'color:red;cursor:pointer;text-align:center',
            onclick: this.unlockPage
        }
    ]
}]
```
#### 4.table里支持列排序
示例：
```js
attributes: [
{
    prop: 'id',
    label: '页面ID',
    width: '100',
    sortable: true
}
]
```
#### 5.table新增ifelsetext类型
示例：
```js
attributes:[{
    prop: 'modManage',
    label: '模块管理',
    type: 'ifelsetext',
    width: '100',
    options: [
        {
    	    prop: 'isEditing',
        	value: false,
            style: 'color:blue;cursor:pointer;text-align:center',
            onclick: this.manageModule
        },
        {
        	prop: 'isEditing',
            value: true,
            style: 'color:red;cursor:pointer;text-align:center',
            onclick: this.unlockPage
        }
    ]
}]
```
+ 效果图：
  <img src="http://onm9ileaw.bkt.clouddn.com/letv/4.png">