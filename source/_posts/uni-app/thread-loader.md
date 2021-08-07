
---
title: uni-app无法使用thread-loader
categories: uni-app
tags: [thread-loader, uni-app]
date: 2021-6-27
---

拦截config.module.rules，给每个rule插入thread-loader。
按照文档的说法，thread-loader有使用限制：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628307624782-image.png)

翻译一下，有以下限制：
1. 无法在具有输出文件功能的loader上使用
2. 不能在plugin生成的loader上使）
3. 不能在访问了webpack配置的loader上使用

暂时先不理会这些限制，先让它跑起来。

 安装最新版（3.0.3），vue.config.js文件添加以下代码：
 ```
config.module.rules[19].use.unshift({loader: 'thread-loader'})
```

实验需要，随便找了一个/\.vue$/文件添加thread-loader，运行一下有报错：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628307666164-image.png)

去github上查到类似的issue ：https://github.com/dcloudio/uni-app/issues/2198
根据别人的解决方案，尝试了升级版本，更换npm源，都没有解决。

无奈只能自己定位错误。
代码定位到`vue-li-shared/lib/platform`文件：
```
const uniPluginOptions = global.uniPlugin.options || {}
```

`global.uniPlugin`值为undefined，我找下uniPlugin的来源，发现全局只赋值了一次，而thread-loader之间的global变量是不共享的。

后来跑去看了源码，看到在设置默认vue.config.js配置时的一段注释：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628307706484-image.png)

看来果真如此，自定义compiler不是在每个线程都执行一遍的，可能只有第一个线程能拿到这个自定义的compiler。

不能用thread-loader的另外一个原因是，uni-app的loader使用了不能序列化的配置，否则会出现loadder内部报错属性undefined之类的bug，正如vue-cli官网解释的那样：

