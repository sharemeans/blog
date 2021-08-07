---
title: webpack学习笔记
categories: 其它
tags: [sourcemap, webpack]
date: 2021-6-20
---

有这样一个模块：

```
// src/base/b.js
function foo() {
  console.log('function foo')
}
module.exports = foo
```

## development 'eval-source-map'
b模块的编译结果为：

```
/***/ "./src/base/b.js":
/*!***********************!*\
  !*** ./src/base/b.js ***!
  \***********************/
/*! no static exports found */
/***/ (function(module, exports) {

eval("function foo() {\n  console.log('function foo');\n}\n\nmodule.exports = foo;//# sourceURL=[module]\n//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9zcmMvYmFzZS9iLmpzP2Q5YjYiXSwibmFtZXMiOlsiZm9vIiwiY29uc29sZSIsImxvZyIsIm1vZHVsZSIsImV4cG9ydHMiXSwibWFwcGluZ3MiOiJBQUFBLFNBQVNBLEdBQVQsR0FBZTtBQUNiQyxTQUFPLENBQUNDLEdBQVIsQ0FBWSxjQUFaO0FBQ0Q7O0FBQ0RDLE1BQU0sQ0FBQ0MsT0FBUCxHQUFpQkosR0FBakIiLCJmaWxlIjoiLi9zcmMvYmFzZS9iLmpzLmpzIiwic291cmNlc0NvbnRlbnQiOlsiZnVuY3Rpb24gZm9vKCkge1xuICBjb25zb2xlLmxvZygnZnVuY3Rpb24gZm9vJylcbn1cbm1vZHVsZS5leHBvcnRzID0gZm9vIl0sInNvdXJjZVJvb3QiOiIifQ==\n//# sourceURL=webpack-internal:///./src/base/b.js\n");

/***/ })
```

`eval`方法的注释部分便是sourcemap。包含
1. sourceURL。

2. sourceMappingURL
该部分是base64编码。解码后如下：

```
{
  // sourcemap版本号是3
  "version": 3,
  // 源码地址，可能由多个模块合并，因此为数组格式。后面的hash用来刷新浏览器缓存。
  "sources": [
    "webpack:///./src/base/b.js?d9b6"
  ],
  // 转换前的所有变量名、属性名、方法名
  "names": [
    "foo",
    "console",
    "log",
    "module",
    "exports"
  ],
  // 记录位置信息的字符串
  "mappings": "AAAA,SAASA,GAAT,GAAe;AACbC,SAAO,CAACC,GAAR,CAAY,cAAZ;AACD;;AACDC,MAAM,CAACC,OAAP,GAAiBJ,GAAjB",
  // 转换后的文件名，这里不清楚为什么后缀名重复
  "file": "./src/base/b.js.js",
  // 转换后的代码，目测分行存储在数组中
  "sourcesContent": [
    "function foo() {\n  console.log('function foo')\n}\nmodule.exports = foo"
  ],
  // 转换前的文件所在的目录。如果与转换前的文件在同一目录，该项为空
  "sourceRoot": ""
}
```

## mappings

- 每个分号对应转换后源码的一行；
- 每个逗号对应转换后源码的一个位置；
- 每个位置通常是5位；

5位的位置说明：

- 第一位，表示这个位置在【转换后代码】的第几列。
- 第二位，表示这个位置属于【sources属性】中的哪一个文件。
- 第三位，表示这个位置属于【转换前代码】的第几行。
- 第四位，表示这个位置属于【转换前代码】的第几列。
- 第五位，表示这个位置属于【names属性】的哪一个变量。该位置非必须。如果不是属性或变量则为空。
- 不需要保存转换后的行号，因为mappings中的分号就是行分割符。
- 每一行的位置数据，第一、三、四位都属于相对位置，相对mappings中前一个元素的位置，第三、四位的相对位置要看转换前的代码。

参考资料：[source-map 的原理](https://mp.weixin.qq.com/s/cuAiTfri0Ju0CD6a6MPRUg)

```
// 转换前
function foo() {
  console.log('function foo')
}
module.exports = foo
// 转换后
function foo() {\n  console.log('function foo');\n}\n\nmodule.exports = foo;
```
使用[base64-vlq库](https://www.npmjs.com/package/@lib/base64-vlq)或者[站长工具](https://www.murzwin.com/base64vlq.html)将mappings解码：

```
// 源码
"mappings": "AAAA,SAASA,GAAT,GAAe;AACbC,SAAO,CAACC,GAAR,CAAY,cAAZ;AACD;;AACDC,MAAM,CAACC,OAAP,GAAiBJ,GAAjB"
// 解码后：
0) [0,0,0,0], [9,0,0,9,0], [3,0,0,-9], [3,0,0,15]
1) [0,0,1,-13,1], [9,0,0,7], [1,0,0,1,1], [3,0,0,-8], [1,0,0,12], [14,0,0,-12]
2) [0,0,1,-1]
4) [0,0,1,-1,1], [6,0,0,6], [1,0,0,1,1], [7,0,0,-7], [3,0,0,17,-4], [3,0,0,-17]
// 绝对位置
(([from_position](source_index)=>[to_position]))

([0,0](#0)=>[0,0]) | ([0,9](#0)=>[0,9]) | ([0,0](#0)=>[0,12]) | ([0,15](#0)=>[0,15])
([1,-13](#0)=>[1,0]) | ([1,-6](#0)=>[1,9]) | ([1,-5](#0)=>[1,10]) | ([1,-13](#0)=>[1,13]) | ([1,-1](#0)=>[1,14]) | ([1,-13](#0)=>[1,28])
([2,-1](#0)=>[2,0])
([3,-1](#0)=>[4,0]) | ([3,5](#0)=>[4,6]) | ([3,6](#0)=>[4,7]) | ([3,-1](#0)=>[4,14]) | ([3,16](#0)=>[4,17]) | ([3,-1](#0)=>[4,20])
```
解码是确实解码了，位置看起来有点对不上，估计是算法没理解对，先不管了。


参考资料：
- [An Introduction to Source Maps](https://blog.teamtreehouse.com/introduction-source-maps)
- 

## 浏览器加载sourcemap

以chrome为例，preferences面板可以开启sourcemap支持：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-29/1624937369621-image.png)

浏览器自动识别代码中的sourceURL字段并加载对应的代码。

有了sourcemap，浏览器自动会解析源代码的位置。

这里有一点细节。
* sourceMappingURL中的sources字段，路径格式为`webpack:///`。sourceURL的路径格式为`webpack-internal:///`。
* sourceURL是用来给eval方法内的代码字符串命名的。webpack-dev-server编译后的代码是通过eval执行的。浏览器直接以该文件名建立新文件，调试时可以直接打开这个新文件，而不用在定位到eval方法中。
* sourceMappingURL中的sources内的文件为转化前的代码路径，用来调试时报错定位。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-29/1624971697729-image.png)