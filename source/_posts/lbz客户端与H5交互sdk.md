---
title: lbz客户端与H5交互sdk
date: 2018-03-13
categories: H5
tags: 
    - H5
    - IOS
    - Android
    - sdk
#description: 
---

lbz客户端与H5交互sdk定义文档
<!-- more -->

### 一、APP相关
#### 1.是否是APP环境
H5调用示例：
```js
var env = LBZSdk.app.isAppEnv; // true | false
```
说明：此方法不需要客户端提供接口
#### 2.获取APP信息
接口名：
core.getAppInfo
传入参数：
(function(info){})
回调示例：
```js
{
  name: '{String}', // 应用名称,如:AiQiYi,Letv
  key: '{String}', // 应用对应的key，如：key123
}
```
H5调用示例：
```js
LBZSdk.app.getInfo(function(info){
  console.log(info.name);
})
```
说明：回调字段可能有修改

#### 3.是否安装某个应用
接口名：
core.hasInstalled
传入参数：
(opt, function(info){})
opt参数示例：
```js
{
  "name": "weixin" // weixin | alipay | ... 需要判断的应用名(1.ios支持判断哪些应用依赖于第三方宿主APP支持；2.安卓目前支持主流应用 - weixin | alipay | mqq | youku | taobao | weibo | mqzone | openapp.jdmobile | letvclient | tmall | qiyi-iphone)
}
```
回调示例：
```js
{
  status: {Boolean} // 是否已安装  true | false
}
```
H5调用示例：
```js
LBZSdk.app.hasInstalled({name: 'weixin'},function(res){
  console.log(res.status);
})
```

#### 4.监听app退出到后台
接口名：
core.onPause
传入参数：
(function(res){})
回调示例：
```js
{

}
```
H5调用示例：
```js
LBZSdk.gEvent.enable('onPause');// 取消注册监听把"enable"换成"disable"
LBZSdk.device.on('onPause', function(){
  console.log('onPause');
})
```

#### 5.监听app从后台回显
接口名：
core.onResume
传入参数：
(function(){})
回调示例：
```js
{

}
```
H5调用示例：
```js
LBZSdk.gEvent.enable('onResume');// 取消注册监听把"enable"换成"disable"
LBZSdk.device.on('onResume', function(){
  console.log('onResume');
})
```

### 二、获取设备信息，网络状态
#### 1.获取设备信息
接口名：
core.getDeviceInfo
传入参数：
(function(info){})
回调示例：
```js
{
  deviceId: '{String}', // 设备唯一标示符
  name: '{String}',  // 设备名称 XiaoMi、Iphone
  version: '{String}', // 系统版本 10.2
  screen: { // 屏幕分辨率
    width: '{Number}', // 宽
    height: '{Number}' // 高
  }
}
```
H5调用示例：
```js
LBZSdk.device.getInfo(function(info){
  console.log(info.deviceId);
})
```
#### 2.获取电量
接口名：
core.getPowerLevel
传入参数：
(function(power){})
回调示例：
```js
{
  level: {Number} // 当前电量0-100
}
```
H5调用示例：
```js
LBZSdk.device.getPower(function(power){
  console.log(power.level);
})
```
#### 3.获取设备存储空间
接口名：
core.getSpaceSize
传入参数：
(function(size){})
回调示例：
```js
{
  spaceSize: {Number}, // 字节 剩余存储空间大小
  totalSize: {Number} // 字节 总存储空间大小
}
```
H5调用示例：
```js
LBZSdk.device.getSpaceSize(function(size){
  console.log(size.spaceSize);
})
```
~~#### 4.监听屏幕旋转（不需要这个接口）~~
~~接口名：~~
~~core.onOrientationChange~~
~~传入参数：~~
~~(function(net){})~~
~~回调示例：~~
```js
{
  orientation: '{String}', // 当前屏幕方向 landscape | portrait
}
```
~~H5调用示例：~~
```js
LBZSdk.gEvent.enable('onOrientationChange');// 取消注册监听把"enable"换成"disable"
LBZSdk.device.on('onOrientationChange', function(res){
  console.log(res.orientation);
})
```
#### 5.设置屏幕方向
接口名：
core.setOrientation
传入参数：
(opt, function(net){})
opt参数示例：
```js
{
  "orientation": "landscape" // 屏幕方向 landscape | portrait
}
```
回调示例：
```js
{
  status: {Boolean}, // true | false 是否设置成功
}
```
H5调用示例：
```js
LBZSdk.device.setOrientation({"orientation": "landscape"}, function(res){
  console.log(res.status);
})
```

#### 6.获取网络
接口名：
core.getNetwork
传入参数：
(function(net){})
回调示例：
```js
{
  type: '{String}',   // 2G|3G|4G|wifi|unknown|none 当前网络类型
  operator: '{String}'  // CUCC（中国联通）|CMCC（中国移动）|CTCC（中国电信） 通信运营商
}
```
H5调用示例：
```js
LBZSdk.network.getInfo(function(net){
  console.log(net.type);
})
```
#### 7.监听网络
接口名：
onNetworkChange
传入参数：
(function(net){})
回调示例：
```js
{
  now: '{String}' //2G|3G|4G|wifi|unknown|none 当前网络类型
}
```
H5调用示例：
```js
LBZSdk.gEvent.enable('onNetworkChange');// 取消注册监听把"enable"换成"disable"
LBZSdk.network.on('onNetworkChange', function(net){
  console.log(net.now);
})
```


### 三、登陆和用户
#### 1.获取登陆信息
接口名：
fun.isLogin
传入参数：
(function(flag){})
回调示例：
```js
{
  login: {Boolean}, // 是否登陆 true | false
}
```
H5调用示例：
```js
LBZSdk.user.isLogin(function(flag){
  console.log(flag);
})
```
~~#### 2.获取用户信息（废弃，H5页面上直接调用接口获取用户信息）~~
~~接口名：~~
~~fun.getUserInfo~~
~~传入参数：~~
~~(function(info){})~~
~~回调示例：~~
```js
{
  name: '{String}', // 用户名
  nickname: '{String}', // 用户昵称
  email: '{String}', // 邮箱
  uid: '{String}', // uid
  img: '{String}', // 用户头像
  status: {Number} // 用户状态：0:正常 1:已禁用 ...
}
```
~~H5调用示例：~~
```js
LBZSdk.user.getInfo(function(info){
  console.log(info.name);
})
```
~~说明：如果获取不到用户信息则返回为空{}~~

#### 3.登陆
接口名：
fun.userLogin
传入参数：
(function(info){})
回调示例：
```js
{
  res: {Boolean}, // true | false 是否登陆成功
  errCode: {Number} // 客户端拿token去乐必中后端换ticket时返回的错误码，如10108：用户被封禁  注：res为true的时候errCode为200
}
```
H5调用示例：
```js
LBZSdk.user.login(function(info){
  console.log(info.res);
})
```

#### 4.重新授权（H5用参数必传，且为web）
接口名：
fun.reGrant
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  source: 'web' // 固定为 web，为了区分来自H5还是APP里面的
}
```
回调示例：
```js
{
  status: {Boolean} // true | false 是否成功
}
```
调用示例：
```js
LBZSdk.user.reGrant(function(res){
  console.log(res.status);
})
```
说明：1.在入口处如果授权失败，会进入授权失败页面（APP内置），点击页面空白处会调用该方法去重新走授权流程。当重新授权成功后，跳转到目标页。
     2.用户未登陆状况下进入页面，然后页面上调起用户登陆接口，如果此时登陆授权失败，则需要跳转到H5的授权失败页面。该方法在H5授权失败页调用。


~~#### 5.登出（废弃）~~
~~接口名：~~
~~fun.userLogout~~
~~传入参数：~~
~~(function(res){})~~
~~回调示例：~~
```js
{
  status: '{Boolean}' // 是否登出成功 true | false
}
```
~~H5调用示例：~~
```js
LBZSdk.user.logout(function(res){
  console.log(res.status);
})
```
~~#### 6.监听登陆状态改变（废弃）~~
~~接口名：~~
~~fun.onLoginChange~~
~~传入参数：~~
~~(function(res){})~~
~~回调示例：~~
```js
{
  action: '{String}', // 动作名称（login | logout）
  name: '{String}', // 用户名
  nickname: '{String}', // 用户昵称
  email: '{String}', // 邮箱
  uid: '{String}', // uid
  img: '{String}', // 用户头像
  status: {Number} // 用户状态：0:正常 1:已禁用 ...
}
```
~~H5调用示例：~~
```js
LBZSdk.user.on('onLoginChange', function(res){
  console.log(res.action);
})
```
~~说明：当action是logout时，其他字段为空~~


### 四、客户端存取数据
#### 1.存储数据
接口名：
fun.setData
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  key: '{String}', // 存储的key
  value: '{String}}' // 存储的内容
}
```
回调示例：
```js
{
  status: {Boolean}, // 是否存储成功 true | false
}
```
H5调用示例：
```js
LBZSdk.data.setData({key: 'abc', value: '123'}, function(res){
  if(res.status === false){
    console.log('存储失败');
  }
})
```
#### 2.读取数据
接口名：
fun.getData
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  key: ["abc", "123"] // 获取的key数组
}
```
回调示例：
```js
{
  "abc": "ds",
  "123": "qwqwq"  // 数据，单个元素类型可以是String,Number,JSONString
}
```
H5调用示例：
```js
LBZSdk.data.getData({key: ["abc", "123"]}}, function(res){
  console.log('获取的数据为：', res);
})
```
说明：如果获取不到数据则返回结果里data为空

#### 3.复制到系统剪切板
接口名：
fun.copyToClipboard
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  data: 'abc123'
}
```
回调示例：
```js
{
  status: {Boolean}, // 是否复制成功  true | false
}
```
H5调用示例：
```js
LBZSdk.data.copyToClipboard({data: 'abc123'}, function(res){
  console.log(res.status);
})
```

#### 4.读取系统剪切板内容
接口名：
fun.getClipboard
传入参数：
(function(res){})
回调示例：
```js
{
  data: '{String}', // 剪切板数据，类型可以是String,Number,JSONString
}
```
H5调用示例：
```js
LBZSdk.data.getClipboard(function(res){
  console.log('获取的数据为：', res.data);
})
```
说明：如果获取不到数据则返回结果里data为空

### 五、webView相关
#### 1.打开新窗口
接口名：
fun.openWebview
传入参数：
(opt)
opt示例如下：
```js
{
  url: '{String}', // 打开的链接
  title: '{String}', // 设置窗口顶部title
  type: '{String}' // 新webview打开还是当前
}
```
说明：type 可选"new"和"cur"，指定是否需要新开webview，默认"cur"
回调示例：
```js
无回调
```
H5调用示例：
```js
LBZSdk.win.open({url: 'http://www.le.com', type: 'new', title: '乐视视频'});
```
#### 2.窗口前进一页
接口名：
fun.WebviewForward
传入参数：
(function(res){})
回调示例：
```js
{
  status: '{Boolean}', // 前进结果 true | false
}
```
H5调用示例：
```js
LBZSdk.win.forward(function(res){
  console.log(res.status);
})
```
说明：当webview里没有下一页的时候status返回false

#### 3.窗口后退一页
接口名：
fun.WebviewBack
传入参数：
(function(res){})
回调示例：
```js
{
  status: '{Boolean}', // 后退结果 true | false
}
```
H5调用示例：
```js
LBZSdk.win.back(function(res){
  console.log(res.status);
})
```
说明：当webview里没有上一页的时候status返回false

#### 4.后退到指定url
接口名：
fun.backToUrl
传入参数：
(opt, function(res){})
opt示例
```js
{
  url: '{String}'  // url
}
```
回调示例：
```js
{
  result: '{Boolean}', // true | false
}
```
H5调用示例：
```js
LBZSdk.win.backToUrl({"url": "http://m.le.com/"}, function(res){
  console.log(res.result);
})
```
说明：
1.如果历史纪录里没有该url，则不做处理，回调函数返回false;
2.如果历史纪录里有且只有一个，则回退到指定url;
3.如果历史纪录里有多个指定的url，则回退到最外层的那个，即最开始打开的那个指定url

#### 5.设置窗口属性
接口名：
fun.setWebview
传入参数：
(opt, function(res){})
opt示例
```js
{
  "title": "乐必中", // 窗口title
  "bg": "150B3B", // head背景颜色,不需要#
  "textColor": "ffffff", // 字体颜色,不需要#
  "themes": "light", // 当前主题：1.用于改变状态栏 2.如果没传backIcon，便于客户端选择本地默认的backIcon 3.如果没传closeIcon，便于客户端选择本地默认的closeIcon
  "backIcon": "http://i2.letvimg.com/lc07_img/201812/13/14/46/back_light@3x.png", // 返回按钮图片地址 http://i2.letvimg.com/lc07_img/201812/13/14/45/back_dark@3x.png
  "closeIcon": "http://i0.letvimg.com/lc07_img/201812/13/14/46/close_light@3x.png" // 关闭按钮图片地址 http://i1.letvimg.com/lc07_img/201812/13/14/46/close_dark@3x.png
}
```
回调示例：
```js
{
  status: '{Boolean}', // 设置结果 true | false
}
```
H5调用示例：
```js
LBZSdk.win.set({"title": "乐必中", "bg": "150B3B", "textColor": "ffffff", "themes": "light", "backIcon": "http://i2.letvimg.com/lc07_img/201812/13/14/46/back_light@3x.png", "closeIcon": "http://i0.letvimg.com/lc07_img/201812/13/14/46/close_light@3x.png"}, function(res){
  console.log(res.status);
})
```

#### 6.注册返回按钮阻止事件
-- 需要在url中加参数 backInterrupt=1
-- 如果想定制化提示文案或者按钮文案，需要在url里加参数：interruptTitle, interruptCo, interruptYesText,interruptNoText
-- 默认提示文案： "确认返回" "取消" "确定"
-- url示例：http://lebz.le.com/?backInterrupt=1&interruptTitle=大爷,不再玩会儿了&interruptCo=以后常来啊&interruptYesText=不玩了&interruptNoText=滚
接口名：
onBackInterrupt
传入参数：
(function(net){})
回调示例：
```js
{
  result: 'ok|cancel'  // 点击了哪个按钮
}
```
H5调用示例：
```js
LBZSdk.gEvent.enable('onBackInterrupt');// 取消注册监听把"enable"换成"disable"
LBZSdk.network.on('onBackInterrupt', function(res){
  console.log(res.result);
})
```

~~#### 5.调起APP原生页面（不需要这个接口）~~
~~接口名：~~
~~fun.openNative~~
~~传入参数：~~
~~(opt)~~
~~opt示例如下：~~
```js
{
  url: '{String}' // APP内页面地址
}
```
~~回调示例：~~
```js
无回调
```
~~H5调用示例：~~
```js
LBZSdk.win.openNative({url: 'user_center'});
```
~~说明：传入的url需要提前知道~~


### 六、弹框
#### 1.打开弹框
接口名：
fun.Alert
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  title: '{String}', // 标题
  content: '{String}}', // 内容
  buttons: '{Array}' // 显示按钮的文字，最多两个，如：['确定'] 或 ['Yes'] 或 ['是','否']
}
```
回调示例：
```js
{
  btn: '{String}' // 用户点击的按钮  ok | cancel
}
```
H5调用示例：
```js
LBZSdk.alert.open({title: '显示标题', content: '显示的内容', buttons: ['好的','取消']}, function(res){
  if(res.btn === 'cancel'){
    console.log('用户点击了取消按钮');
  } else {
    console.log('用户点击了确定按钮');
  }
});
```
说明：1.如果传入的参数中buttons只有一个，则默认显示"确认"按钮
     2.如果之前已有一个弹出框了，则覆盖之前的
#### 2.toast
接口名：
fun.Toast
传入参数：
(opt)
opt示例如下：
```js
{
  dur: '{Number}', // 显示时长,单位：毫秒,默认2000 如：2500
  pos: {Number},  // 显示位置， 0 | 1 （0:center,1:bottom）
  content: {String} // 显示的内容，其中 \n 为换行符
}
```
回调示例：
```js
无回调
```
H5调用示例：
```js
LBZSdk.alert.toast({dur: 3000, pos: 0, content: '分享成功\n获得20金币'})
```


### 七、分享
#### 1.打开分享弹框
接口名：
fun.openShare
传入参数：
(opt, function(){})
opt示例如下：
```js
{
  title: '', // 自定义分享标题
  desc: '', // 自定义分享内容
  link: '', // 自定义分享链接
  imgUrl: '' // 自定义分享图标
}
```
回调示例：
```js
{
  code: {Number}, // 分享状态码（200表示成功，400表示用户取消等，具体定义待确定）
  channel: {String} // 用户点击的分享渠道 （一期共三种 wxTimeline | wxFriend | weibo）
}
```
H5调用示例：
```js
LBZSdk.share.open({
   title: '', // 自定义分享标题
   desc: '', // 自定义分享内容
   link: '', // 自定义分享链接
   url: '', // 自定义分享链接（乐视android版分享字段是url，ios版是link）
   imgUrl: '', // 自定义分享图标
   bubbleTxt: '', // 气泡文案，如果不传或者传空，则不显示气泡
   shareIcon: '', // 分享图标地址，如果不传用默认的
   themes: 'dark | light' // 当前主题，便于客户端选择默认分享图标
}, function(res){
  console.log('分享渠道：' + res.channel + ';分享结果：' + res.code);
});
```
说明：1.一期只支持三种渠道分享：微信朋友圈，微信好友，新浪微博
     2.此分享是页面里用户点击按钮后手动触发的分享，与 showMenuShare 不同

#### 2.显示右上角分享按钮
接口名：
fun.showMenuShare
传入参数：
(opt, function(){})
opt示例如下：
```js
{
   title: '', // 自定义分享标题
   desc: '', // 自定义分享内容
   link: '', // 自定义分享链接
   url: '', // 自定义分享链接（乐视android版分享字段是url，ios版是link）
   imgUrl: '', // 自定义分享图标
   bubbleTxt: '', // 气泡文案，如果不传或者传空，则不显示气泡
   shareIcon: '', // 分享图标地址，如果不传用默认的
   themes: 'dark | light' // 当前主题，便于客户端选择默认分享图标
   ...
}
```
回调示例：
```js
{
  status: {Boolean} // 调用结果 true | false
}
```
H5调用示例：
```js
LBZSdk.share.showMenuShare({
  "title": "您还有1000金币待领取~手慢无！",
  "desc": "玩小游戏做任务，就能赚金币，赶紧戳☞",
  "link": encodeURIComponent("https://lebz.le.com/fruitShare.html"),
  "url": encodeURIComponent("https://lebz.le.com/fruitShare.html"),
  "imgUrl": encodeURIComponent("http://i0.letvimg.com/lc06_img/201804/26/18/26/wechatShare.png"),
  "bubbleTxt": '+20金币',
  "shareIcon": 'http://i0.letvimg.com/lc07_img/201812/13/14/47/share_light@3x.png',
  "themes": 'light'
}, function(res){
  console.log(res);
});
```

#### 3.关闭右上角分享按钮
接口名：
fun.hideMenuShare
传入参数：
(function(){})
回调示例：
```js
{
  status: {Boolean} // 调用结果 true | false
}
```
H5调用示例：
```js
LBZSdk.share.hideMenuShare(function(res){
    console.log(res);
});
```

#### 4.注册右上角分享监听
接口名：
onMenuShare
传入参数：
(function(net){})
回调示例：
```js
{
  code: {Number}, // 分享状态码（200表示成功，400表示用户取消等，具体定义根据第三方APP给的错误码确定）
  channel: {String} // 用户点击的分享渠道 （如 wxTimeline | wxFriend | weibo | wxShare）
}
```
H5调用示例：
```js
LBZSdk.gEvent.enable('onMenuShare');// 取消注册监听把"enable"换成"disable"
LBZSdk.share.on('onMenuShare', function(res){
  console.log(res);
})
```

~~#### 2.分享（废弃）~~
~~接口名：~~
~~fun.callShare~~
~~传入参数：~~
~~(opt, function(res){})~~
~~opt示例如下：~~
```js
{
  channelName: 'wxFriend', //分享渠道：weibo|wxFriend|wxTimeline
  title: '', // 自定义分享标题
  desc: '', // 自定义分享内容
  link: '', // 自定义分享链接
  imgUrl: '', // 自定义分享图标
}
```
~~回调示例：~~
```js
{
 code: {Number} // 分享结果code，需具体定义，如：200表示分享成功，400表示用户取消分享等...
}
```
~~H5调用示例：~~
```js
LBZSdk.share.callShare({
  channelName: 'wxFriend',
  title: '自定义分享标题',
  desc: '自定义分享内容',
  link: '自定义分享链接',
  imgUrl: '自定义分享图标'
}, function(res){
  if(res.code === 200){
    console.log('分享成功了，恭喜');
  }
})
```
~~说明：传入的参数可能需要根据不同的渠道差异化~~


### 八、支付
#### 1.调用支付
接口名：
fun.pay
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  goodsId: '{String}', // 商品id
  channel: '{String}' // 支付渠道 weixin | alipay | thirdWeixin | thirdAlipay        applepay(for ios)
}
```
回调示例：
```js
{
  code: {Number} // 支付结果code,需具体定义，如：200表示支付成功，400表示用户取消支付，401表示传入的金额与订单实际金额不符等...
}
```
H5调用示例：
```js
LBZSdk.pay.payOrder({goodsId: 'abcd1234', channel: 'weixin'}, function(res){
  if(res.code === 200){
    console.log('支付成功');
  } else if(res.code === 400){
    console.log('用户取消支付');
  } else {
    其他情况需具体定义
  }
});
```

### 九、缓存
#### 1.更新缓存
接口名：
fun.updateCache
传入参数：
(function(res){})
回调示例：
```js
{
  status: {Boolean} // 是否更新成功 true | false
}
```
H5调用示例：
```js
LBZSdk.cache.updateCache(function(res){
  console.log(res.status);
});
```

### 十、日期、时间选择
#### 1.日历选择
接口名：
fun.selectDate
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  "minDate": "2016-01-03", // 结束日期，当type为"time"时不需要
  "maxDate": "2019-11-12", // 开始日期，当type为"time"时不需要
  "type": "date" // 可选 date | time | ym  ==>  日期 | 时间 | 年月
}
```
回调示例：
```js
{
  value: {String} // 选择的时间（如"2018-02-23"、"2018-02"、"17:23"），如果用户点取消不会进入该回调
}
```
H5调用示例：
```js
LBZSdk.calendar.select({
  "minDate": "2016-01-03",
  "maxDate": "2019-11-12",
  "type": "date"
}, function(res){
  console.log(res.value);
});
```

### 十一、第三方原生APP相关
#### 1.打开商品详情页
接口名：
fun.openDetail
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  "channel": "tmall", // tmall | jd | taobao
  "id": 212123   // 商品id
}
```
回调示例：
```js
{
  status: {Boolean}, // 是否成功 true | false
}
```
H5调用示例：
```js
LBZSdk.native.openDetail({"channel": "taobao", "id": 1213121}, function(info) {
    alert(info);
});
```

### 十二、pomelo（仅供安卓4.4以下）
#### 1.init
接口名：
fun.initPomelo
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  "host": 'x.x.x.x',
  "port": 'xxxx',
  // 其他根据不同业务调用
}
```
回调示例：
```js
{
  status: {Boolean} // true | false
}
```
H5调用示例：
```js
LBZSdk.pomelo.init({
    "host":"lebzs.le.com",
    "port":9014,
    "user":{"uid":"10002","nickname":"乐必中_10002","picture":"https://i3.letvimg.com/lc06_user/201605/09/15/04/1923203311462777446_50_50.jpg","balance":"23800"}
  }, function(info) {
    alert(info);
});
```

#### 2.request
接口名：
fun.requestPomelo
传入参数：
(opt, function(res){})
opt示例如下：
```js
{
  // 其他根据不同业务调用
}
```
回调示例：
```js
{
  // 其他根据不同业务调用
}
```
H5调用示例：
```js
LBZSdk.pomelo.request("fruit.fruitHandler.select", {
      "fruit":{"id":4,"key":"banana4","image":"//i3.letvimg.com/lc03_img/201803/01/10/23/1022/fruits04.png","title":"雪梨","order":3}
   }, function(res) {
    alert(res);
});
```

#### 3.disconnect
接口名：
fun.disconnectPomelo
传入参数：
()
H5调用示例：
```js
LBZSdk.pomelo.disconnect();
```

#### 4.监听pomelo
接口名：
core.onPomeloEvent
传入参数：
(function(res){})
回调示例：
```js
{
  type: '{String}', // 自定义的监听类型，如： onResult , onSelect ...
  data: {}  // 回调的数据
}
```
H5调用示例：
```js
LBZSdk.gEvent.enable('onPomeloEvent');// 取消注册监听把"enable"换成"disable"
LBZSdk.pomelo.on('onPomeloEvent', function(res) {
    if('onResult' === res.type){
        console.log('onResult', res.data);
    } else if('onSelect' === res.type){
        console.log('onSelect', res.data);
    } else if(){
        ...
    }
});
```

+ 目前能想到的接口文档大致如此，做的过程中会有删改