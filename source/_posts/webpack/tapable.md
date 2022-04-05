---
title: tapable学习笔记
categories: 工程化
tags: [tapable]
date: 2021-7-24
---  

## tapable内部关系结构图
tapable官方文档写的过于简洁，如果想要搞清楚tap类型、钩子类型、call类型之间的关系，还是得看源码才行。以下是我整理出来的关系结构图：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-7-24/1627102887500-tapable%E5%85%B3%E7%B3%BB%E7%BB%93%E6%9E%84%E5%9B%BE.png)

## 简单示例
以AsyncSeriesHook的callAsync为例，分别插入一个promise和async类型的插件：
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

#### 插件挂载
tapAsync和tapPromise方法会将插件插入tap数组中。
#### 插件执行
AsyncSeriesHook为异步串行执行插件队列：执行完每个插件后都会接着执行下一个插件。

插件执行的代码是经过包裹封装的。以上代码封装后的结构如下：
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
以上是callAsync最终执行的函数。
- _callback就是callAsync传入的回调
- this._x指的是tap数组
- _next0内部封装了插件tap2的执行语句
- 当tap1插件，即_fn0方法的callback调用时无传参，则表示插件正常结束，可以执行_next0。否则不继续执行剩余的插件，直接结束队列并调用_callback

## 总结
1. tap方法类型决定了**插件类型**
2. 钩子类型其实是**插件的执行顺序**和**插件传参/终止条件**的组合，和插件类型无关
3. 钩子执行方式中，同步钩子只能用同步方法调用。异步钩子只能用异步方法调用。
4. 插件类型取决于插件本身的需求，无需迁就钩子类型。只不过，异步插件就不要挂载到同步钩子类型上。
5.promise和callAsync方法的不同体现在，插件队列结束时通过何种方式执行钩子回调。call方法无需回调。

