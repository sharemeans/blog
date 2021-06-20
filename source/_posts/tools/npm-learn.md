---
title: npm学习总结
categories: 工具
tags: [npm]
date: 2021-5-20
--- 

## dependencies

如果是工具库之类的项目，发布之后，第三方使用时会安装dependencides，优先安装在第三方的目录下。

项目依赖和工具库依赖不冲突，则依赖安装到项目下，否则就会在这个工具库的目录下再安装一份。

## devDependencies

普通业务项目开发时，dependencies和devDependencies区别感知不大。但是如果我们开发的是一个工具库，publish之后生成的package.json中是没有devDependencies的。

## peerDependencies

同等依赖。这个同等的意思是，你想引用我这个库，你最好也一起安装这些依赖，要和我平级安装在你的目录下，而不是像dependencies一样有可能安装在我的库目录下面。

## 命令

## npm install

npm install的依赖安装处理逻辑如下图：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-8/1623119747450-image.png)

### 项目依赖和工具库依赖冲突

项目依赖和工具库依赖不冲突，则依赖安装到项目下，否则就会在这个工具库的目录下再安装一份。

项目有如下依赖：
```
{
  "dependencies": {
    "fetch": "^1.1.0",
    "biskviit": "1.0.1"
  }
}
```
fetch@^1.1.0的依赖如下：
```
{
  "dependencies": {
    "biskviit": "2.0.1"
  }
}
```

由于项目顶级依赖和fetch对biskviit的版本不一致，存在冲突，所以biskviit在项目的node_modules目录下安装一份1.0.1的版本，然后在node_modules/fetch/node_modules目录下安装2.0.1版本。


### 工具库之间依赖冲突

假如项目的依赖如下：
```
{
  "dependencies": {
    "fetch": "^1.1.0",
    "biskviit": "2.0.0"
  }
}
```
fetch@^1.1.0的依赖如下：
```
{
  "dependencies": {
    "biskviit": "1.0.1"
  }
}
```
安装fetch时，遇到了biskviit@1.0.1，会先检查项目依赖有没有biskviit。找到了，但是版本不一致，npm会选择较高的版本安装在顶级node_modules目录下，其余版本安装在各自工具库的目录下。

### npm init
初始化npm管理的项目，结果是一个package.json文件。

也可以通过config命令修改单个字段：
```
npm config set init.author.name "Lucas"
npm config set init.author.email "lucasXXXXXX@gmail.com"
npm config set init.author.url "lucasXXXXX.com"
npm config set init.license "MIT"
```

### npm ls

列出当前目录下npm包列表，以及之间的依赖关系。

### npm config get cache 
获取npm包本地缓存的目录

### 私有npm搭建工具

nexus、verdaccio 以及 cnpm

### npx 命令工具
```
npm install -g npx
```

假如，npm安装了webpack-cli，package.json有个script为：
```
"scripts": {
    "dev": "webpack"
}
```
我们可以在命令行运行npm run dev执行webpack命令，但是无法在命令行直接执行webpack。只能这样：
```
./node_modules/.bin/webpack 
```

npx的作用就是帮我们找到命令的路径并执行，其实就是个语法糖。
```
npx webpack 
```

## nrm 镜像管理
设置镜像的命令：
```
npm config set registry http://registry.npm.taobao.org
```
我们不免会出现要切换镜像的时候，但是镜像的地址可能忘记了。这时候需要一个镜像管理工具，nrm。
```
npm install -g nrm
nrm add taobao/*或者其它名字*/ http://registry.npm.taobao.org
nrm ls // 查看镜像列表，以及当前使用的镜像
nrm use taobao // 切换镜像
```

## 参考资料
* [没想到你是这样的npm install](https://mp.weixin.qq.com/s/LATAribargpMvDa_nrzACQ)
* [npm 安装机制及企业级部署私服原理](https://www.yuque.com/allenstone/learn/crd8kf)