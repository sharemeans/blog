---
title: 项目静态资源CDN服务无响应情况的回源
categories: 工程化
tags: [webpack, cdn]
date: 2021-5-27
---

## 背景
某日，华为云上部署的cdn服务故障导致部分区域静态资源（js和css）响应超时。这种情况是我们没有预料到的，幸亏用户不多，否则需要紧急发版将静态资源指向源服务器。

我们的项目采用uni-app框架开发，内部基于vue-cli。生产模式会使用publicPath配置将静态资源指向cdn服务器，即，代码经过webpack编译后资源引用地址就已经固定了。

## 任务拆解
为了应对以上情况，我们需要实现一种机制，能够识别cdn异常，并切换资源为源服务器。

我们的任务分为以下：
1. publicPath运行时全局变更
2. cdn资源请求异常识别方法
3. 资源开关切换机制

## 方案分析与实施
### publicPath运行时全局变更
经过分析打包后的模块代码：
```
/******/  function jsonpScriptSrc(chunkId) {
/******/    return __webpack_require__.p + "static/js/" + ({"project-qlymain-pages-404":"project-qlymain-pages-404"}[chunkId]||chunkId) + "." + {"chunk-0826ee9b":"964d9183","chunk-69a5a22c":"9e1c8209","project-qlymain-pages-404":"330de720"}[chunkId] + ".js"
/******/  }
/******/  ....此处省略一些中间代码
/******/  // __webpack_public_path__
/******/  __webpack_require__.p = "/";
```
以及webpack源码：
```
this.hooks.requireExtensions.tap("MainTemplate", (source, chunk, hash) => {
  const buf = [];
  const chunkMaps = chunk.getChunkMaps();
  // ...
  // ...
  buf.push("// __webpack_public_path__");
  buf.push(`${this.requireFn}.p = ${JSON.stringify(publicPath)};`);
  return Template.asString(buf);
});
```
可以发现，publicPath被写入模块属性`__webpack_require__.p`上，我只需要修改这个属性值就可以了。

一开始我写了个插件，通过拦截compiler钩子修改这个模块属性：
```
function pushBuff(valName, defaultVal, source) {
  var buf = [];
  buf.push(source);
  buf.push('');
  buf.push('// Dynamic assets path override (/build/plugins/runtime-publicpath-plugin)');
  buf.push(`__webpack_require__.p = ${valName} || '${defaultVal}'`);
  return buf.join('\n');
}
// ...
if (compiler.hooks && compiler.hooks.thisCompilation) {
    compiler.hooks.thisCompilation.tap('RuntimePublicpathPlugin', function (compilation) {
        compilation.mainTemplate.hooks.requireExtensions.tap('RuntimePublicpathPlugin', function (source, chunk, hash) {
            return pushBuff(globalVariable, defaultPublicPath, source)
        })
    });
}
```

但是，这个插件的目的是覆盖`__webpack_require__.p`的值为runtime的运行时全局变量（window[globalVariable]），但是我们的开关状态可能要存储到localstorage中（因为要实现状态缓存），而且存进去的字段名由业务代码决定，这样就导致插件和业务过渡耦合。

后来发现业务代码中可以使用webpack提供的变量[__webpack_public_path__](https://webpack.js.org/api/module-variables/#__webpack_public_path__-webpack-specific)，webpack会在编译时将这些变量替换成__webpack_require__.p。

至此，我们找到了publicPath运行时的覆盖方法：
```
__webpack_public_path__ = localstorage.getItem('__webpack_public_path__')
```
#### 异步资源加载
打包后按需加载的代码：
```
// script path function
function jsonpScriptSrc(chunkId) {
    return __webpack_require__.p + "" + ({
    "project-qlymall-views-order-logisticsDetail":"project-qlymall-views-order-logisticsDetail",
    "project-qlymall-views-order-refundDetail":"project-qlymall-views-order-refundDetail"
    // ......
    }[chunkId]||chunkId)+'.js'
}
// ......
// 创建script标签
/******/        var script = document.createElement('script');
/******/        // ......
/******/        script.src = jsonpScriptSrc(chunkId);
/******/
```
为了使我们的开关机制生效，需要保证jsonpScriptSrc方法在publicPath变更后执行。实施发现只要将以下语句放在入口文件最顶部执行即可。
```
__webpack_public_path__ = localstorage.getItem('__webpack_public_path__')
```


#### 插入index.html的cdn标签需要重写
处理完异步加载的资源，我们还要处理入口index.html的资源。vue-cli内部使用了html-webpack-plugin将所有的assets插入了index.html中，包括pulicPath：
```
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, viewport-fit=cover">
        <link rel="stylesheet" href="https://mycdn.mycompany.com/static/index.css" />
    </head>
    <body>
        <div id="app"></div>
        <script src=https://mycdn.mycompany.com/static/js/index.bd9cfc6f.js></script>
    </body>
</html>
```
如何处理这种情况呢？有2个思路。
1. 本着解决问题不如绕开问题的原则，考虑入口文件代码直接内联
2. 入口文件也使用异步加载的方法

针对方案1，看起来很简单粗暴，但是会产生一个问题：index.html使用的是源站资源，而非cdn。index.js一般代码比较大，如果内联是不是本身就违背了cdn的初衷呢？所以最终我们采用了方案2。

##### 入口文件异步加载
让入口文件异步加载的方法，一开始想到的方案是，写插件，覆盖vue-cli使用html-webpack-plugin的默认行为，替代为动态插入script标签的逻辑，这段代码由于不经过babel，所以语法需要谨慎。想想这样做有点大费周章了。

后来发现可以直接覆盖掉入口文件，我们称之为main2.js，该文件关键代码：
```
/* 加载主入口文件 */
import('@/main.js').catch(err => {
  handleError(err)
})
```
这样就巧妙实现了入口文件异步加载。那么问题来了，main2.js又会被html-webpack-plugin插入index.html。由于main2.js的代码量较少，我选择内联，使用HtmlWebpackInlineSourcePlugin将main.js内联到index.html中。
#### index.css的处理
在调试HtmlWebpackInlineSourcePlugin的过程中发现，index.css并不在assets列表中。我们看看index.css的生成代码：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-5-30/1622347411128-image.png)

这段代码做了2件事：
1. 通过检查html中有没有VUE_APP_INDEX_CSS_HASH来判断生辰的index.css文件名要不要带上hash
2. 将@dcloudio/uni-h5/dist/index.css文件中的内容取出来再加工，用copyWebpackPlugin插件将文件粘贴到dist目录中。

鉴于该文件不大，可以选择内联或者在main2.js中动态插入link标签。我选择了后者。

既然选择了动态插入link标签，就要知道，index.[hash].css的文件名，如何在运行时知晓呢？很简单，使用definePlugin将VUE_APP_INDEX_CSS_HASH注入运行时即可。

### cdn加载异常识别

一开始想到的是使用window.addEventListener('error')，但是尝试后发现无法捕获promise中的异常，所以改用window.addEventListener('unhandledrejection')。

在回调中，需要做判断区分是否是资源加载超时。通过分析webpack模块源码可以发现它有做超时异常处理：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-5-30/1622347667614-image.png)

且超时时间可以通过output.chunkLoadTimeout配置修改，默认值是120s，觉得太长的可以考虑覆盖。对于现在的用户来说，12s都算很长了，所以我选择缩短到12s。此外，超时异常格式可以通过上图的type和message匹配。

## cdn开关切换
针对开关切换，一开始想到的方案是通过接口变更开关状态，但是这样存在以下问题：
1. 简单的开关可以通过人为控制apollo字段实现，但是无法感知客户端异常。开发者需要实现一个客户端告警机制触发开关变更，实现成本较高。
2. 开关属于全局性的，一旦切换到源服务器，将会对源服务器造成较大压力。

后来选择客户端自行切换开关的方案：

**识别到cdn加载超时之后，我们业务的处理逻辑是：切换开关，将开关缓存5min，缓存的原因是，我们的业务属于APP内嵌h5，每个窗口都会重新初始化，所以需要窗口之间开关状态共享。**


## 难点清单
* 找到动态资源拼接的代码，找到__webpack_require__.p，如何重写它，使得运行时可修改
index.css处理
* 内联？还是动态插入的选择
* index.css文件名有hash怎么办


