---
title: uni-app H5模式下treeShaking的坑
categories: uni-app
tags: [treeShaking, uni-app]
date: 2021-5-10
---  

uni-apph5打包模式，使用[treeShaking](https://uniapp.dcloud.io/collocation/manifest?id=treeshaking)时，该配置只在生产模式下有效，即使部署到测试和预生产都没有问题，发布到生产之前一定要先本地打包生产模式确保无异常。

我们的项目发布到生产之后运行h5出现以下报错：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622344617219-image.png)

经过一番全局搜索，看到了一个插件（目录：@dcloudio/vue-cli-plugin-uni-optimize/packages/babel-plugin-uni-api/index.js）：

```
process.UNI_APIS = new Set()
module.exports = function ({
  types: t
}) {
  return {
    visitor: {
      MemberExpression (path, state) {
        if (
          t.isIdentifier(path.node.object) &&
          (
            path.node.object.name === 'uni' ||
            path.node.object.name === 'wx'
          )
        ) {
          process.UNI_APIS.add(path.node.property.name || path.node.property.value)
        }
      }
    }
  }
}
```
该插件在获取了AST语法树之后，将遇到uni.xxx语法的对象属性引用对应的api名称添加到process.UNI_APIS中。

然而，通过调试发现代码并没有运行到此处，此路不通。

后来终于发现了一个关键线索。

1. 读取optimization.treeShaking开关并标记

@dcloudio/vue-cli-plugin-uni/lib/env.js文件中读取到optimization.treeShaking配置为true时，就会执行以下代码：
```
process.env.UNI_OPT_TREESHAKINGNG = true
```

2. vue-cli-optimize插件

vue-cli-plugin-uni-optimize插件（@dcloudio/vue-cli-plugin-uni-optimize/index.js）针对uni-api做了以下事情：
* 将process.UNI_APIS写入@dcloudio\vue-cli-plugin-uni-optimize\.tmp\api.js路径下的文件
* 将uni api的模块引用路径替换成@dcloudio\vue-cli-plugin-uni-optimize\.tmp\api.js路径下的文件

但是我们从前文可知，唯一一处更新process.UNI_APIS的代码并未执行（猜测是uni-app代码的bug），导致uni-api update了个寂寞:
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622345030913-image.png)

以上关键代码并未更新任何api，只是引入了一些默认的核心api（@dcloudio/vue-cli-plugin-uni-optimize/packages/webpack-optimize-plugin/api.js）:
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622345080481-image.png)

以上代码标记部分，manifest[name][2]的值为true，即为核心api时才会被默认引入。

由于不知道什么原因没有使用babel-plugin-uni-api，而且，考虑到该插件的tree-shaking比较粗暴，增加了业务代码的开发复杂度。

而我负责的项目，业务中存在对uni的直接使用扩展运算符来实现uni-api和其它自定义api的整合：
```
sdk = {
    ...uni,
    someCustomApi: function(){...}
}
```
即使babel-plugin-uni-api插件正常工作，根据该插件的shaking原理，我们需要将扩展运算符改为按需赋值的结构：
```
sdk = {
    api1: uni.api1,
    api2: uni.api2,
    someCustomApi: function(){...}
}
```
每增加一个新的uni-api都要去修改sdk文件，这是开发不希望发生的事情。因此考虑简单粗暴从manifest.json文件入手，让每一个api都成为核心api。在compiler的beforeCompile钩子回调中修改manifest文件的标记：
```
const manifest = require('@dcloudio/uni-h5/manifest.json')
Object.keys(manifest).forEach(name => {
  if (manifest[name].length < 3) {
    manifest[name].push(true)
  }
})
const newManifest = JSON.stringify(manifest, null, 2);
try {
  fs.writeFileSync(path.resolve(process.cwd(), 'node_modules', '@dcloudio/uni-h5/manifest.json'), newManifest)
  console.log('@dcloudio/uni-h5/manifest.json 替换成功...')
} catch (err) {
  console.log('@dcloudio/uni-h5/manifest.json 替换失败...')
}
```
> writeFileSync将json写回文件时，需要转化为字符串，为了让字符串能够换行，保持json格式，需要传入JSON.stringify的第三个参数值为2，表示保留换行和空格，以及每次缩进使用2个空格。

## 结果

将“未开启tree-shaking”，“开启tree-shaking打包异常”，“开启tree-shaking并修复api打包异常”3个版本打包结果对比：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622345486280-image.png)

可以看到，uni-app的tree-shaking效果一般，修复tree-shaking的打包结果依旧比开启前少了0.1M。

[uni-app 2.2 大幅优化H5端性能体验，只开发H5，也要用uni-app](https://ask.dcloud.net.cn/article/36279)这篇文章提到，manifest内部的的tree-shaking其实主要做了2件事，内置组件的treeShaking和api的treeShaking，而且，文中数据显示，shaking效果拔群。但是本人尝试的结果发现效果实在一般。

