---
title: 前端错误捕获学习与实践
categories: uni-app
tags: [错误捕获, uni-app]
date: 2021-6-4
---

小程序和上线后遇到部分页面出现白屏的情况，小程序还好说，有些错误是可以在微信后台浏览的，H5就不一样了，经历了找遍测试机都无法复现的痛苦之后，迎来的便是缠着用户配合调试的无奈。自己动手，解决一部分开发者的痛苦。先把能捕获的错误上报先。

## H5
### window.onerror = function (event) {}

该回调可以捕获js运行时错误。如果函数返回true，就会阻止默认事件处理函数（如consoe输出）。

### window.addEventListener('error')

```
window.addEventListener('error', function (msg, url, lineNo, columnNo, error) {
    // 获取出错的脚本路径，行列信息，以及报错信息
}, true)
```

该回调可以捕获js语法错误，或者运行时错误，或者脚本加载错误。比window.onerror先触发。无法阻止默认事件处理函数。

如果要和window.onerror一起使用，需要过滤要重叠的部分，该方法可以只负责监听脚本加载错误：

```
window.addEventListener('error', event => (){ 
  // 过滤js error
  let target = event.target || event.srcElement;
  let isElementTarget = target instanceof HTMLScriptElement || target instanceof HTMLLinkElement || target instanceof HTMLImageElement;
  if (!isElementTarget) return false;
  // 上报资源地址
  let url = target.src || target.href;
  console.log(url);
}, true);
```


### window.addEventListener('unhandledrejection')

```
window.addEventListener('unhandledrejection', function (event) {
    const error = event.reason
}, true)
```

该回调可以捕获promise链中未被catch的错误。可以通过event.preventDefault阻止默认事件处理函数（如：console输出）。

### 跨域资源脚本错误捕获
script标签不受浏览器同源策略影响。但是，H5默认跨域js无法获取脚本错误的具体信息。除非script标签增加跨域限制，且资源返回`Access-Control-Allow-Origin`头部信息。

由于我们的项目，生产的静态JS资源可能使用了cdn，这种情况下，脚本报错是无法获取完整信息的。只能得到“script error.”信息。

解决办法：
1. 为cdn资源的返回头添加`Access-Control-Allow-Origin`头部即可。（目前采用该方法）
2. script标签添加： crossorigin="anonymous"。该步骤是匿名获取目标脚本。

以上2步都要做。针对webpack的htmlWebpackPlugin，生成的动态script标签默认是没有crossorigin属性，我们可以借用以下插件帮忙完成添加属性的工作：
- [webpack-subresource-integrity](https://www.npmjs.com/package/webpack-subresource-integrity)
- [html-webpack-inject-attributes-plugin](https://www.npmjs.com/package/html-webpack-inject-attributes-plugin)

参考资料：
- https://www.cnblogs.com/vivotech/p/11162672.html

### 现有成熟的可解决方案

- [trackjs](https://trackjs.com/how/)
- [sentry](https://sentry.io/)

## 小程序

### App.onError或wx.onError
官方表示，二者的触发时机一致，但是经过实际试验，发现后者在自定义组件的生命周期钩子中并没有触发。

相关社区文章：
https://developers.weixin.qq.com/community/develop/doc/000c8cf5794770272709f38a756000

### 官方后台
官方后台可查看客户端捕获的代码报错，无需业务端重新上报。

### 成熟方案

#### sentry

sentry提供小程序平台方案：https://github.com/lizhiyao/sentry-miniapp


## 参考资料
- [前端异常埋点系统初探](https://mp.weixin.qq.com/s/nvI_6e_DC0p1ukY9oXStWg)