---
title: "插件Pipeline: Groovy"
categories: 工具
date: 2021-4-30
--- 

Jenkins 自动化构建基于一系列插件，[常用的插件](https://www.jianshu.com/p/e0b7d377132a?from=timeline)

最基础的是流水线插件。公司内的流水线插件配置使用Pipeline: Groovy。该插件的基本配置如下。

可以直接在文本框写入简单的脚本（Pipeline script），也可以读取Jenkinsfile内的流水线脚本（Pipeline script from scm）。scm全称是Source Control Management，如gitlab。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343189420-image.png)

选择了SCM之后需要配置git的地址和账号密码：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343216758-image.png)

选择构建分支：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343247362-image.png)

构建分支BRANCH的值来源于参数，需要配置参数化构建，Jenkins可配置的参数类型有：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343271570-image.png)

其中，Git Parameter类型需要安装插件：Git Parameter。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343386941-image.png)

该插件获取以下类型的值，并赋值给我们定义的变量名BRANCH（可以是其它名字）：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343409839-image.png)

分支选择时可以配置过滤器以及分支显示框的高度：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343429756-image.png)

另外可以配置其它参数，如构建环境：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343450153-image.png)
checkbox类型的值传入Jenkinsfile中的格式为字符串，分隔符和Jenkins上的配置一致。

Jenkinsfile中可以通过params 获取到参数化构建中传入的参数：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343471298-image.png)
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343490451-image.png)


选择构建分支时，可以使用参数化构建传入的参数：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343513092-image.png)

pipeline:Groovy可以配置构建执行脚本文件路径：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-30/1622343529779-image.png)