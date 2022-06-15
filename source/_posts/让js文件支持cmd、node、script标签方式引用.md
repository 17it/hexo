---
title: 让js文件支持cmd、node、script标签方式引用
date: 2018-10-08
categories: JS
tags: 
    - JS
#description: 
---

让js文件支持cmd、node、script标签方式引用
<!-- more -->
我们纯手生写的js文件，怎样同时支持cmd、node、script标签方式引用，只需要给他简单的套用一层外壳
####
```js
(function (global, factory) {
  "use strict";
  if (typeof module === "object" && typeof module.exports === "object") {
    module.exports = global.document ?
      factory(global, true) :
      function (w) {
        if (!w.document) {
          throw new Error("requires a window with a document");
        }
        return factory(w);
      };
  } else {
    factory(global);
  }
})(typeof window !== "undefined" ? window : this, function (window, noGlobal) {
  "use strict";

  if (typeof define === "function" && define.amd) {
    define("xxx", [], function () {
      return u;
    });
  }

  if (!noGlobal) {
    window.xxx = u;
  }
  return u;
});
```

实践出真知
```js
var util = {
  VERSION: '1.0.01',
  fun: function(){
    alert('fun');
  }
};

(function (global, factory) {
  "use strict";
  if (typeof module === "object" && typeof module.exports === "object") {
    module.exports = global.document ?
      factory(global, true) :
      function (w) {
        if (!w.document) {
          throw new Error("requires a window with a document");
        }
        return factory(w);
      };
  } else {
    factory(global);
  }
})(typeof window !== "undefined" ? window : this, function (window, noGlobal) {
  "use strict";

  if (typeof define === "function" && define.amd) {
    define("util", [], function () {
      return u;
    });
  }

  if (!noGlobal) {
    window.util = u;
  }
  return u;
});
```
现在试着用 require 、import、和直接script标签方式引用吧！！！
