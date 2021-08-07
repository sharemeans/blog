---
title: uni-app的环境变量
categories: uni-app
tags: [环境变量, uni-app]
date: 2021-8-1
---

> uni-app有较多环境变量，有些变量是没有体现在官方文档中的。以下是个人用过的环境变量。

### NODE_ENV
构建环境。该值可以影响到一些默认配置。如

值为production时：
- h5模式下，publicPath才能生效。
- 错误信息不会输出
- 启用terser代码压缩

值为development时：
- h5模式下，默认开启sourceMap

### UNI_PLATFORM
基准平台，即代码编译目标。可选值有：
- h5
- mp-weixin
- app-plus
- quickapp-native
- mp-baidu
- mp-toutiao
- mp-qq
它的值决定了编译的核心流程。

### UNI_INPUT_DIR
入口文件所在目录，默认是src。

不只是入口文件，也代表着和入口文件层级关系固定的资源路径。

因此不管入口目录是什么，该目录下的资源要符合uni-app的规则。

### UNI_OUTPUT_DIR

打包后代码的输出目录。默认为：
```
/dist/${process.env.NODE_ENV === 'production' ? 'build' : 'dev'}/${process.env.UNI_PLATFORM}
```

### UNI_MINIMIZE
是否开启代码压缩。
