---
title: H5 ReplaceState详情页到列表页的无刷新
date: 2017-03-05
categories: H5
tags: 
    - H5
#description: 
---

最近遇到一个用户体验问题，即从详情页回到列表页的时候，需要记住用户上次的位置，并且不刷新。
最开始返回使用的是 history.Go(-1)，发现并不好使，也许在有些浏览器下有效果。折腾半天，突然看到HTML5里新加的 window.history.replaceState的方法，以下是亲自实践。
<!-- more -->
### 假设前提
#### 列表页位list.html
+ 获取列表的方法 getList()
+ 支持分页
+ 构建列表的方法dealWithResult()
+ 到详情页的方法 viewDetail()
+ 一个全局变量data，用来累加存储列表数据

#### 详情页是 detail.html ，我们不需要在detail页面做任何动作，只需要修改list页面

### 方法实现
1.在viewDetail方法里面处理，记住当前页码、当前滚动条的位置、当前列表显示的所有数据
```js
var statedata = {}; 
statedata.curPage = curPage; //当前页码
statedata.data = data; //列表所有的数据  
statedata.sh = function () { //当前滚动条位置
  var scrollTop = 0;
  if (document.documentElement && document.documentElement.scrollTop) {
    scrollTop = document.documentElement.scrollTop;
  }else if (document.body) {
    scrollTop = document.body.scrollTop;
  }             
  return scrollTop;
}();
```
2.替换state，不推荐使用 history.pushState方法，因为pushState每次都会加入一条数据，导致在列表页返回的时候会一直返回上一个state,不能退出列表页
```js
  history.replaceState(statedata , "", "list.html");
  // 跳转到详情页
  window.location.href = 'detail.html?id=';
```
3.在list.html的onload里面处理
```js
// 判断如果有history.state.data，说明是从详情页返回的
if(!!(window.history.state && window.history.state.data)) {
  $("#loading").hide(); 
  $("#nomore").hide(); $("#loadmore").show(); //隐藏loading，显示加载更多（为了分页）
  dealWithResult(window.history.state); //根据记录的数据显示列表
  curPage = window.history.state.curPage; 
  statedata  = window.history.state; //把页面和data赋值给全局变量
  window.history.replaceState({}, "", "list.html"); //清空state，防止列表页点返回的时候会回到上一个state
}else{
  getList(); //没有window.history.state.data，说明不是从详情页来的，则调用ajax从服务器获取数据
}
```
4.在dealWithResult方法里处理：
```js
function dealWithResult(listdata){
  // 构建列表  略
  // 判断如果是详情页回来，获取上次的滚动条位置，等列表构建完成后，用js设置当前滚动条位置为上次的位置
  if (!!(window.history.state && window.history.state.data)) {
    // 延迟 0.5秒滚动，防止页面中列表还没构建完成，保险起见（其实在执行这里之前，列表页已经构建完成）
    setTimeout(window.scroll(0, window.history.state.sh || 0), 500);
    // 此处用window.scroll方法，不用window.scrollBy，因为window.scrollBy会乱跳
  }
}
```
5.在getList方法里需要给全局变量statedata赋值
```js
if (curPage == 1) { 
  statedata = data; 
} else { // data为ajax调用接口返回的数据
  var s = statedata.data;
  //如果当前页码不是1，则把返回的结果累加到statedata里面
  for (var t in data.data) {
    s.push(data.data[t]);
  }
  statedata.data = s;
}
```
