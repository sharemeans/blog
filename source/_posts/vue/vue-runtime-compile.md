---
title: vue runtime 和 esm
categories: vue
date: 2020-8-1
---
## 一. esm是vue 的“运行时”和“编译器”的集合

### 运行时

运行时是用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切。


### 编译器

用来将模板字符串编译成为 JavaScript 渲染函数的代码。

在客户端编译模板包含以下情况：

1. el属性指定模版挂载的DOM，和该DOM包含的html，也就是说，既没有指定template，又没有render函数

2. 用template属性指定模板渲染的字符串

### 如何选择

vue的package.json文件中的module属性指定了模块的入口文件为vue.runtime.esm.js，为什么呢，因为这个只包含了运行时，不包含编译器。相比全部包含的vue.esm.js文件而言，vue.runtime.esm.js体积小了将近三分之一，初始化运行速度相对来说会高一些。

我发现，在开发单页应用的时候，绝大多数情况下可以避免出现使用编译器的情况，所以，我只说下我遇到的情况。

通常，我们的单页应用的入口html，入口文件和顶级组件分别是index.html, main.js和App.js。代码如下：

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title></title>
  </head>
  <body>
    <div id="app">
    </div>
    <!-- built files will be auto injected -->
  </body>
</html>

<!-- main.js -->
import Vue from 'vue'
import App from './App'

Vue.config.productionTip = false

new Vue({
  el: '#app',
  template: '<App/>',
  components: {App}
})

<!-- App.vue -->
<template>
  <div>
    HelloWorld
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
```

就像上面这种情况，初始化根实例的时候，用template指定了挂载元素，这就意味着，一定要用编译器。如果你的webpack.config.js里面没有设置vue的alias，并且，在main.js文件中，直接使用import Vue from 'vue'，你会发现，浏览器会报错:


```
[Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.
```

但是，如果你把

```
import Vue from 'vue'
```
改成

```
import Vue from 'vue/dist/vue.esm.js'
```
就会运行正常。
不知道有多少项目都只是因为这个根实例导致不得不引入vue.esm.js。

那么，针对这个情况怎么解决呢？想必很多人都知道答案了。代码如下：


```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title></title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>

<!-- main.js -->
import Vue from 'vue'
import App from './App'

Vue.config.productionTip = false

new Vue({
  el: '#app',
  render(h) {
    return h(App)
  }
})

<!-- App.vue -->
<template>
  <div>
    HelloWorld
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
```
其实只改了一行代码，就是把main.js中的template替换成render函数。之前发现有的项目用render有的直接template，现在才知道是这么回事。

## 二. esm和common什么关系，为什么除了runtime.esm和esm之外，还有vue.common.js和vue.js呢？

### 1. esm全称是ESModule，意思是遵循es6的import export模块化规则。

### 2. common的意思是遵循common.js的exports模块化规则。

### 3. 如何选择?

我们知道common.js的模块化是输出一个exports对象，没法做到按需引入，一旦引入，就是用 require引入整个exports对象。但是ES6的import 可以选择引入哪些属性。

前者是运行时引入，后者是编译时引入（NodeJS打包）。由于编译时即可知道哪些是没有用到的，这样就可以达到tree shaking的目的（通过webpack配合达到）。

esm.js文件里面是符合ES6模块规则的es5的语法，是经过babel编译过的。为什么要是es5的语法呢？一般情况下业务代码在使用babel-loader的时候会exclude掉node_module目录，这是因为，node_module目录里面有大量的源代码，处理起来很浪费时间，所以干脆模块开发者帮你编译好，别人直接用你编译好的，而且能tree shaking的代码版本即可。
