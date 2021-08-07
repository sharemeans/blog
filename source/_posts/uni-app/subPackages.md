---
title: "uni-app 分包优化"
categories: uni-app
tags: [分包优化, uni-app]
date: 2021-6-29
--- 

manifest.json文件配置，可以开启分包优化。
```
"mp-weixin":{
  // ...
   "optimization": {
     "subPackages": true
   }
  // ...
}
```

该配置优化的具体具体逻辑：

- 静态文件：分包下支持 static 等静态资源拷贝，即分包目录内放置的静态资源不会被打包到主包中，也不可在主包中使用
- js文件：当某个 js 仅被一个分包引用时，该 js 会被打包到该分包内，否则仍打到主包（即被主包引用，或被超过 1 个分包引用）
- 自定义组件：若某个自定义组件仅被一个分包引用时，且未放入到分包内，编译时会输出提示信息

划重点：
1. 自定义组件不会自动被优化，只是会在命令行显示提示信息，并不会自动移动组件
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628306356264-image.png)

2. 只会自动移动可移入分包的js文件。
3. 分析分包建议结果，发现其建议存在部分误导，开发者需要主动鉴别：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628306408144-image.png)
4. 分析分包建议结果，发现其建议存在遗漏。
5. npm中的组件也会参与分包优化建议，但是使用easycom的情况下是不会被提示的。