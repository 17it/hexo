---
title: Micro-App
date: 2025-12-17
categories: web
tags: 
    - web
    - 微前端
#description: 
---

当项目过多时，往往需要把一个项目内嵌到另外一个项目中，也就会用到微前端。之前一般会用qiankun，但现在有了另一种选择-Micro-App。
<!-- more -->

微前端的概念是由ThoughtWorks在2016年提出的，它借鉴了微服务的架构理念，核心在于将一个庞大的前端应用拆分成多个独立灵活的小型应用，每个应用都可以独立开发、独立运行、独立部署，再将这些小型应用融合为一个完整的应用，或者将原本运行已久、没有关联的几个应用融合为一个应用。微前端既可以将多个项目融合为一，又可以减少项目之间的耦合，提升项目扩展性，相比一整块的前端仓库，微前端架构下的前端仓库倾向于更小更灵活。

京东的微前端框架 MicroApp，借鉴了WebComponent的思想，通过js沙箱、样式隔离、元素隔离、路由隔离模拟实现了ShadowDom的隔离特性，并结合CustomElement将微前端封装成一个类WebComponent组件，从而实现微前端的组件化渲染，旨在降低上手难度、提升工作效率。

MicroApp和技术栈无关，也不和业务绑定，可以用于任何前端框架。

#### 优势：

+ 1、使用简单
我们将所有功能都封装到一个类WebComponent组件中，从而实现在基座应用中嵌入一行代码即可渲染一个微前端应用。

+ 2、功能强大
MicroApp提供了js沙箱、样式隔离、元素隔离、路由隔离、预加载、数据通信等一系列完善的功能。

+ 3、兼容所有框架
为了保证各个业务之间独立开发、独立部署的能力，micro-app做了诸多兼容，在任何前端框架中都可以正常运行。

### 一、快速开始

**主应用：**

1、安装依赖
```bash
npm i @micro-zoe/micro-app --save
```
2、初始化micro-app
```bash
// main.js
import microApp from '@micro-zoe/micro-app'
microApp.start()
```
3、加载子应用(name必传且不能重复)
```bash
<micro-app name='my-app' url='http://localhost:3000/'></micro-app>
```

**子应用：**

micro-app从主应用通过fetch加载子应用的静态资源，由于主应用与子应用的域名不一定相同，所以子应用需要支持跨域。
```bash
location / {
add_header 'Access-Control-Allow-Origin' '*';
  # 其它配置...
}
```

到此，最简单的micro-app就集成了。

### 二、配置项

开发一个Hello World浏览器插件很简单，只需要一个manifest.json配置文件，和一个popup.html文件即可。开发使用html.css.js，跟书写页面一样简单。

```bash
name：应用名称（不能重复，name变化时会重新渲染）
url：应用地址（必须指向子应用的index.html）
iframe：开启iframe沙箱（为false时候默认为with沙箱）
inline：使用内联script（开启inline模式后，script元素会被保留，有性能损耗，建议仅开发环境使用）
clear-data：卸载时清空数据通讯中的缓存数据（默认false，子应用被卸载后数据通讯中的缓存数据会被保留）
destroy：卸载时删除缓存资源（默认false，子应用被卸载后不会删除缓存的静态资源和沙箱数据）
disable-scopecss：关闭样式隔离（关闭样式隔离可以提升页面渲染速度，但可能会相互污染）
disable-sandbox：关闭js沙箱（关闭沙箱可能会导致一些不可预料的问题）
ssr：开启ssr模式（当子应用是ssr应用时，需要设置ssr属性）
keep-alive：开启keep-alive模式（开启keep-alive后，应用卸载时会进入缓存；优先级小于destroy）
default-page：指定默认渲染的页面（默认子应用渲染后会展示首页，可以指定子应用渲染的页面）
router-mode：路由模式（search、native、native-scope、pure、state）
baseroute：设置子应用的基础路径（只有路由模式是native或native-scope才需要设置）
keep-router-state：保留路由状态（为true则子应用卸载后重新渲染时将恢复卸载前的页面（页面中的状态不保留）优先级小余default-page）
disable-memory-router：关闭虚拟路由系统（默认false，子应用将运行在虚拟路由系统中，和主应用的路由系统进行隔离）
disable-patch-request：关闭子应用请求的自动补全功能（默认false，MicroApp对子应用的fetch、XMLHttpRequest、EventSource进行自动补全子域名）
fiber：开启fiber模式（默认false，子应用js是同步执行的，这会阻塞主应用的渲染线程。开启后会异步执行，但会降低渲染速度）

其它配置:   
globalAssets：用于设置全局共享资源，和预加载一样，会在浏览器空闲时加载资环，提高效率
exclude：过滤元素（当子应用不需要加载某个js或css，可以通过在link、script、style设置exclude属性）
ignore：忽略元素（当link、script、style元素具有ignore属性，micro-app不会处理它，元素将原封不动进行渲染）

```

### 三、生命周期

**生命周期**

+ created: 标签初始化后，加载资源前触发。
+ beforemount: 加载资源完成后，开始渲染之前触发。
+ mounted: 子应用渲染结束后触发。
+ unmount: 子应用卸载时触发。
+ error: 子应用加载出错时触发，只有会导致渲染终止的错误才会触发此生命周期。

**监听生命周期**

```bash
<micro-app name='xx' url='xx' @created='created' @beforemount='beforemount' @mounted='mounted' @unmount='unmount' @error='error' />
```

**全局事件**
在子应用的加载过程中，micro-app会向子应用发送一系列事件，包括渲染、卸载等事件。

+ 渲染事件:
```bash
window.onmount = (data) => {
  console.log('子应用已经渲染', data)
}
```

+ 卸载事件:
```bash
window.onunmount = () => {
  console.log('子应用已经卸载') // 执行卸载相关操作
}

window.addEventListener('onmount|unmount', function () {
  console.log('子应用已经卸载') // 执行卸载相关操作
})
```

### 四、环境变量

+ **__MICRO_APP_ENVIRONMENT__**

在子应用中通过 window.__MICRO_APP_ENVIRONMENT__ 判断是否在微前端环境中

+ **__MICRO_APP_NAME__**

在子应用中通过 window.__MICRO_APP_NAME__ 获取应用的name值

+ **__MICRO_APP_PUBLIC_PATH__**

用于设置webpack动态public-path，将子应用的静态资源补全为 http 开头的绝对地址

1. 在子应用src目录下创建名称为public-path.js
```bash
if (window.__MICRO_APP_ENVIRONMENT__) {
  __webpack_public_path__ = window.__MICRO_APP_PUBLIC_PATH__
}
```

2. 在子应用的入口文件的最顶部引入public-path.js
```bash
import './public-path'
```

+ **__MICRO_APP_BASE_ROUTE__**

子应用的基础路径

+ **__MICRO_APP_BASE_APPLICATION__**

判断当前应用是否是主应用 (在执行microApp.start()后此值才会生效)

+ **rawWindow**

子应用里获取真实window（即主应用window）

+ **rawDocument**

子应用里获取真实document（即主应用document）

### 五、JS沙箱

JS沙箱通过自定义的window、document拦截子应用的JS操作，实现一个相对独立的运行空间，避免全局变量污染，让每个子应用都拥有一个相对纯净的运行环境。

micro-app有两种沙箱模式：with沙箱和iframe沙箱，它们覆盖不同的使用场景且可以随意切换，默认情况下使用with沙箱，如果无法正常运行可以切换到iframe沙箱。

在沙箱环境中，顶层变量不会泄漏为全局变量。

例如：
在正常情况下，通过 var name 或 function name () {} 定义的顶层变量会泄漏为全局变量，通过window.name或name就可以全局访问，但是在沙箱环境下这些顶层变量无法泄漏为全局变量，window.name或name的值为undefined，导致出现问题。

### 六、虚拟路由系统

MicroApp通过拦截浏览器路由事件以及自定义的location、history，实现了一套虚拟路由系统，子应用运行在这套虚拟路由系统中，和主应用的路由进行隔离，避免相互影响。

**路由模式**

有5种模式，search、native、native-scope、pure、state；可以通过配置disable-memory-router:true 关闭；keep-router-state可保留子应用路由状态；
+ search：默认模式，通常不需要特意设置，search模式下子应用的路由信息会作为query参数同步到浏览器地址上
+ native：模式是指放开路由隔离，子应用和主应用共同基于浏览器路由进行渲染，它拥有更加直观和友好的路由体验
+ native-scope：模式的功能和用法和native模式一样，唯一不同点在于native-scope模式下子应用的域名指向自身而非主应用
+ pure：模式是指子应用独立于浏览器路由系统进行渲染，即不修改浏览器地址，也不增加路由堆栈，pure模式下的子应用更像是一个组件。
+ state：模式是指基于浏览器history.state进行渲染的路由模式，在不修改浏览器地址的情况下模拟路由行为，相比其它路由模式更加简洁优雅，表现和iframe路由系统类似。

**导航**

主应用控制子应用跳转
```bash
microApp.router.push|replace({name: 'my-app', path: 'http://localhost:3000/page1?id=9527'})
```

子应用控制主应用跳转(子应用无法直接控制主应用的跳转)
```bash
1、主应用注册：microApp.router.setBaseAppRouter(主应用的路由对象)
2、子应用调用：const baseRouter = window.microApp.router.getBaseAppRouter();  baseRouter.主应用路由的方法(...) ;
```

子应用控制其它子应用跳转
```bash
window.microApp.router.push|replace({name: 'my-app2', path: 'http://localhost:3000/page1'})
```

**设置默认页面**

子应用默认渲染首页，但可以通过设置defaultPage渲染指定的默认页面。

+ 方式一：通过default-page属性设置
```bash
<micro-app default-page='页面地址'></micro-app>
```

+ 方式二：通过router API设置
```bash
microApp.router.setDefaultPage({ name: '子应用名称', path: '页面地址' })
```

**导航守卫**

导航守卫用于监听子应用的路由变化，类似于vue-router的全局守卫，不同点是MicroApp的导航守卫无法取消跳转。

```bash
microApp.router.beforeEach((to, from, appName) => {
  console.log('全局前置守卫 beforeEach: ', to, from, appName)
})

microApp.router.afterEach((to, from, appName) => {
  console.log('全局后置守卫 afterEach: ', to, from, appName)
})
```

**获取路由信息**

主应用获取子应用的路由信息：
```bash
microApp.router.current.get('my-app')
```

子应用获取子应用的路由信息：
```bash
window.microApp.router.current.get('my-app2')
```

**同步路由信息**

在一些特殊情况下，主应用的跳转会导致浏览器地址上子应用信息丢失，此时可以主动调用方法，将子应用的路由信息同步到浏览器地址上。

指定子应用：
```bash
microApp.router.attachToURL('my-app')
```

所有正在运行的子应用：
microApp.router.attachAllToURL()
```bash
```

所有正在运行的子应用，包含处于隐藏状态的keep-alive应用：
```bash
microApp.router.attachAllToURL({ includeHiddenApp: true })
```

所有正在运行的子应用，包含预渲染应用：
```bash
microApp.router.attachAllToURL({ includePreRender: true })
```

### 七、样式隔离、元素隔离

#### 样式隔离

MicroApp的样式隔离是默认开启的，开启后会以<micro-app>标签作为样式作用域，利用标签的name属性为每个样式添加前缀，将子应用的样式影响禁锢在当前标签区域。

+ 1、在所有子应用中禁用
```bash
microApp.start({
  disableScopecss: true, // 默认值false
})
```

+ 2、单个子应用中禁用
```bash
<micro-app name='xx' url='xx' disableScopecss='false'></micro-app>
```

+ 3、在某一个css文件中禁用(可禁用单行 或 整个文件)
```bash
/*! scopecss-disable */
.test { color: red }
/*! scopecss-enable */
```

#### 元素隔离

元素隔离的概念来自ShadowDom，即ShadowDom中的元素可以和外部的元素重复但不会冲突，micro-app模拟实现了类似ShadowDom的功能，元素不会逃离<micro-app>元素边界，子应用只能对自身的元素进行增、删、改、查的操作。

子应用中不能获取主应用中的元素，但主应用可以获取子应用中的。

### 八、数据通信

micro-app提供了一套灵活的数据通信机制，方便主应用和子应用之间的数据传输。

主应用和子应用之间的通信是绑定的，主应用只能向指定的子应用发送数据，子应用只能向主应用发送数据，这种方式可以有效的避免数据污染，防止多个子应用之间相互影响。

同时我们也提供了全局通信，方便跨应用之间的数据通信。

#### 一、子应用获取来自主应用的数据

直接获取：
```bash
window.microApp.getData()
```

监听变化：
```bash
window.microApp.addDataListener(dataListener: (data: Object) => any, autoTrigger?: boolean)
```

#### 二、子应用向主应用发送数据

**子：**
```bash
window.microApp.dispatch({type: '子应用发送给主应用的数据'}, (data) => {})
```

**主：**
```bash
microApp.addDataListener('my-app', (data) => {
    console.log('来自子应用my-app的数据', data)
    return '返回值1' // 会作为参数传给子应用dispatch的回调函数
})
```

forceDispatch: dispatch方法会缓存每次发送的值，然后合并发给主应用，如果相同则不发送，强制发送可以使用forceDispatch

#### 三、主应用向子应用发送数据

+ 方式1: 通过data属性发送数据（data只接受对象类型，数据变化时会重新发送）
```bash
<micro-app name='my-app' url='xx' :data='dataForChild' />
```

+ 方式2: 手动发送数据
```bash
microApp.setData('my-app', {type: '新的数据'}) // 强制发送用forceSetData
```

#### 四、主应用获取来自子应用的数据

+ 方式1：直接获取数据
```bash
microApp.getData(appName) // 返回子应用的data数据
```

+ 方式2: 监听自定义事件 (datachange) // 数据在事件对象的detail.data字段中，子应用每次发送数据都会触发datachange
```bash
<micro-app name='my-app' url='xx' @datachange='handleDataChange' />
```

+ 方式3: 绑定监听函数
```bash
microApp.addDataListener(appName: string, dataListener: (data: Object) => any, autoTrigger?: boolean)
```

#### 五、清空数据

由于通信的数据会被缓存，即便子应用被卸载也不会清空，这可能会导致一些困扰，此时可以主动清空缓存数据来解决。

**主应用：**
+ 方式一：配置项 - clear-data（子应用卸载时会同时清空主应用发送给当前子应用，和当前子应用发送给主应用的数据）
```bash
<micro-app clear-data></micro-app>
```

+ 方式二：手动清空 - clearData
```bash
microApp.clearData('my-app')
```

**子应用：**
```bash
window.microApp.clearData()
```

#### 全局数据通信

全局数据通信会向主应用和所有子应用发送数据，在跨应用通信的场景中适用。

**主应用:**
```bash
microApp.setGlobalData({type: '全局数据'}, () => {}) // 发送全局数据
microApp.clearGlobalData() // 清空全局数据
microApp.getGlobalData() // 获取全局数据方法一
microApp.addGlobalDataListener((data) => {
  console.log('全局数据', data)
  return '返回值1'  // 会作为参数传给子应用dispatch的回调函数
})
```

**子应用：** 跟主应用方法一样，只是把microApp改成 window.microApp

### 九、资源系统

#### 一、资源路径自动补全

针对资源link、script、img、background-image、font-face； 如：子应用中引用图片/myapp/test.png，最终渲染时会补全为 ${子应用域名}/myapp/test.png

publicPath：如果自动补全失败，可以采用运行时publicPath方案解决（见第四章环境变量里 __MICRO_APP_PUBLIC_PATH__ ）

#### 二、资源共享

当多个子应用拥有相同的js或css资源，可以指定这些资源在多个子应用之间共享，在子应用加载时直接从缓存中提取数据，从而提高渲染效率和性能。

+ 方式一：globalAssets
```bash
microApp.start({
  globalAssets: {
    js: ['js地址1', 'js地址2', ...], // js地址
    css: ['css地址1', 'css地址2', ...], // css地址
  }
})
```

+ 方式二：global 属性
```bash
<link rel="stylesheet" href="xx.css" global>
<script src="xx.js" global></script>
```

#### 三、资源过滤

+ 方式一：excludeAssetFilter   
```bash
microApp.start({
  excludeAssetFilter (assetUrl) {
    if (assetUrl === 'xxx') {
      return true // 返回true则micro-app不会劫持处理当前文件
    }
    return false
  }
})
```

+ 方式二：配置 exclude 属性
```bash
<link rel="stylesheet" href="xx.css" exclude>
<script src="xx.js" exclude></script>
<style exclude></style>
```

### 十、预加载

预加载是指在子应用尚未渲染时提前加载静态资源，从而提升子应用的首次渲染速度。

为了不影响主应用的性能，预加载会在浏览器空闲时间执行。

```bash
microApp.preFetch(apps: app[] | () => app[], delay?: number)
```

<img src="/Micro-App/preload.png" width="50%">

+ 方式一：设置数组
```bash
microApp.preFetch([
  { name: 'my-app4', url: 'xxx', level: 1, 'default-page': '/page2' }
])
```

+ 方式二：设置一个返回数组的函数
```bash
microApp.preFetch(() => [
  { name: 'my-app4', url: 'xxx', level: 1, 'default-page': '/page2' }
])
```

+ 方式三：在start中设置预加载数组
```bash
microApp.start({
  preFetchApps: [
    { name: 'my-app4', url: 'xxx', level: 1, 'default-page': '/page2' }
  ]
})
```

+ 方式四：在start中设置一个返回预加载数组的函数
```bash
microApp.start({
  preFetchApps: () => [
    { name: 'my-app4', url: 'xxx', level: 1, 'default-page': '/page2' }
  ]
})
```

此外还可以全局设置预加载的延迟和level：
```bash
microApp.start({
  prefetchDelay: 5000, // 修改delay默认值为5000ms
  prefetchLevel: 1 // 修改level默认值为1  
})
```

### 十一、umd模式

MicroApp支持两种渲染微前端的模式，默认模式和umd模式，推荐使用umd模式。

+ 默认模式：子应用在初次渲染和后续渲染时会顺序执行所有js，以保证多次渲染的一致性。
+ umd模式：子应用暴露出mount、unmount方法，此时只在初次渲染时执行所有js，后续渲染只会执行这两个方法，在多次渲染时具有更好的性能和内存表现。

**开启umd方式**（修改子应用的main.js）

```bash
let app = null
// 将渲染操作放入 mount 函数，子应用初始化时会自动执行
window.mount = () => {
  app = new Vue({
    router,
    render: h => h(App),
  }).$mount('#app')
}

// 将卸载操作放入 unmount 函数，就是上面步骤2中的卸载函数
window.unmount = () => {
  app.$destroy()
  app.$el.innerHTML = ''
  app = null
}

// 如果不在微前端环境，则直接执行mount渲染
if (!window.__MICRO_APP_ENVIRONMENT__) {
  window.mount()
}
```

### 十二、keep-alive

在应用之间切换时，我们有时会想保留这些应用的状态，以便恢复用户的操作行为和提升重复渲染的性能，此时开启keep-alive模式可以达到这样的效果。

开启keep-alive后，应用卸载时不会销毁，而是推入后台运行。

```bash
<micro-app name='xx' url='xx' keep-alive></micro-app>
```

**生命周期**（created、beforemount、mounted、error、afterhidden、beforeshow、aftershow）

+ 1、不会触发unmount
+ 2、beforemount、mounted只在初始化的时候执行一次
+ 3、多beforeshow、aftershow、afterhidden三个周期

**子应用**

keep-alive模式下，在子应用卸载、重新渲染时，micro-app都会向子应用发送名为appstate-change的自定义事件，子应用可以通过监听该事件获取当前状态，状态值可以通过事件对象属性e.detail.appState获取。

e.detail.appState的值有三个：afterhidden、beforeshow、aftershow，分别对应卸载、即将渲染、已经渲染。

应用初始化时不会触发appstate-change事件。

```bash
window.addEventListener('appstate-change', function (e) {
  if (e.detail.appState === 'afterhidden') {
    console.log('已卸载')
  } else if (e.detail.appState === 'beforeshow') {
    console.log('即将重新渲染')
  } else if (e.detail.appState === 'aftershow') {
    console.log('已经重新渲染')
  }
})
```

### 十三、多层嵌套、插件系统

#### 多层嵌套

micro-app支持多层嵌套，即子应用可以嵌入其它子应用，但需要做一些修改（如A套B，B套C，则需要改B）。

+ 1、修改tagName
```bash
microApp.start({
  // 必须是以`micro-app-`开头的小写字母，例如：micro-app-b、micro-app-b-c
  tagName: 'micro-app-xxx'
})

// 将micro-app 换成 micro-app-xxx
<micro-app-xxx name='...' url='...'></micro-app-xxx>
```

+ 2、将B改成umd模式

参照章节十一

**注：**

+ 1、无论嵌套多少层，name都要保证全局唯一

+ 2、确保micro-app的版本一致，不同版本可能会导致冲突

#### 插件系统

插件系统的主要作用就是对js进行修改，每一个js文件都会经过插件系统，我们可以对这些js进行拦截和处理，它通常用于修复js中的错误或向子应用注入一些全局变量。

可参照：https://jd-opensource.github.io/micro-app/docs.html#/zh-cn/plugins

### 十四、高级功能

+ 1、自定义fetch

通过自定义fetch替换框架自带的fetch，可以修改fetch配置(添加cookie或header信息等等)，或拦截HTML、JS、CSS等静态资源

+ 2、excludeRunScriptFilter: 自定义屏蔽JS加载异常

可选择性屏蔽JS加载异常

+ 3、inheritBaseBody: 子应用body标签是否采用基座标签，默认不采用

+ 4、aHrefResolver: 自定义处理所有子应用 a 标签的 href 拼接方式

+ 5、escapeIframeWindowEvents : iframe 模式 逃逸沙盒的window事件

+ 6、disableIframeRootDocument : iframe模式禁用沙箱Document 默认为false

+ 7、excludeRewriteIframeConstructor : iframe模式下排除对指定构造函数的Symbol.hasInstance属性重写

示例：
```bash
microApp.start({
  fetch (url, options, appName) { return Promise }, // 需要return一个Promise
  excludeRunScriptFilter (address, error, appName, appUrl) { return true|false }, // 可以加上报等
  inheritBaseBody: true, // true: 采用基座标签 作为子应用的标签   
  aHrefResolver: (hrefValue, appName, appUrl) => { return 'xxx' },
  escapeIframeWindowEvents: ['message'], // 配置所有iframe子应用 逃逸沙盒的window.message事件
  disableIframeRootDocument: true, // iframe模式禁用沙箱Document，避免一些ui组件库Modal 或tooltip 偏移
  excludeRewriteIframeConstructor: ['EventTarget'] // iframe模式下，不对事件对象的原型判断进行代理
})
```
