---
title: 从tapable学习Function构造函数的用法
categories: 工程化
tags: [tapable, webpack]
date: 2021-8-2
---

> tapable是webpack实现插件机制的核心。插件通过tapable注册到webpack的执行过程中的各个事件节点。当执行过程到达对应节点时，tapable上注册的插件就会以该事件节点要求的方式执行。

tapable执行插件队列的实现，是通过在运行时根据调用方法，使用Function构造函数动态构造新的函数并执行。

如：
```
// 初始化钩子列表
const hook = new AsyncSeriesHook()
// 挂载钩子
hook.tapAsync('tap1', (callback) => {
  setTimeout(() => {
    callback('tap1')
  }, 0)
})
hook.tapPromise('tap2', () => {
  return Promise.resolve('tap2')
})
// 执行钩子
hook.callAsync((res) => {
  console.log(res)
})
```
callAsync方法执行过程中，使用new Function拼接新函数，生成结果如下：
```
function(_callback) {
  "use strict";
  var _context;
  var _x = this._x;
  function _next0() {
    var _fn1 = _x[1];
    var _hasResult1 = false;
    var _promise1 = _fn1();
    if (!_promise1 || !_promise1.then)
      throw new Error('Tap function (tapPromise) did not return promise (returned ' + _promise1 + ')');
    _promise1.then((function (_result1) {
      _hasResult1 = true;
      _callback();
    }), function (_err1) {
      if (_hasResult1) throw _err1;
      _callback(_err1);
    });
  }
  var _fn0 = _x[0];
  _fn0((function (_err0) {
    if (_err0) {
      _callback(_err0);
    } else {
      _next0();
    }
  }));
}
```

### 使用方法


```
new Function ([arg1[, arg2[, ...argN]],] functionBody)
```

**functionBody**
   
    一个字符串，表示函数体的内容。


以下代码：

```
const adder = new Function("a", "b", `
  if (isNaN(a) || isNaN(b)) return
  
  return a + b
`);
```
生成的函数就如下：
```
(function anonymous(a,b
) {

  if (isNaN(a) || isNaN(b)) return
  
  return a + b

})
```
### 作用域

MDN表示，Function动态生成的函数，作用域为全局作用域，而不是创建函数时所在的作用域。

tapabel就想到一个办法，将动态的生成的函数，动态绑定到类实例方法上，在动态函数中直接访问'this'上的属性，即实例属性。

### 安全问题
**new Function**由于是动态生成代码，因此会存在安全问题。在配置了CSP安全策略的浏览器页面中执行会有以下报错：

```
Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'report-sample' 'self' *.speedcurve.com 'sha256-q7cJjDqNO2e1L5UltvJ1LhvnYN7yJXgGO7b6h9xkL1o=' www.google-analytics.com/analytics.js 'sha256-JEt9Nmc3BP88wxuTZm9aKNu87vEgGmKW1zzy/vb1KPs=' polyfill.io/v3/polyfill.min.js assets.codepen.io production-assets.codepen.io".

    at new Function (<anonymous>)
    at <anonymous>:1:15
(anonymous) @ VM54:1
```

因此，Function构造函数一般不适用于在浏览器端使用。