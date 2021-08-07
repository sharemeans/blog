---
title: 修改uni-app的默认打包路径
categories: uni-app
tags: [打包路径, uni-app]
date: 2021-6-8
---  

## 背景

uni-app默认只有开发和生产环境，打包生成的路径默认下是如下映射关系：

NODE_ENV | 目录
---|---
production | dist/build/${UNI_PLATFORM}
其它 | dist/dev/${UNI_PLATFORM}

## 需求

如果新增自定义环境'test'和'preproduction'，则默认都是按照'dev'环境去映射打包路径。新增的环境要怎么做？官方文档并没有明确说明。

我们希望让每个环境都有对应的打包路径：

NODE_ENV | 目录
---|---
production | dist/build/${UNI_PLATFORM}
preproduction | dist/prev/${UNI_PLATFORM}
test | dist/test/${UNI_PLATFORM}
dev | dist/dev/${UNI_PLATFORM}

通过查找源码中打包相关配置，发现有个`UNI_OUTPUT_DIR`参数可以配置打包路径，覆盖掉默认行为。


```    
"test": "cross-env UNI_OUTPUT_DIR=dist/test/mp-weixin NODE_ENV=test UNI_PLATFORM=mp-weixin vue-cli-service uni-build"
```
