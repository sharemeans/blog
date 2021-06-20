---
title: vue-router 总结
categories: vue
tags: [vue, vue-router]
date: 2018-8-20
---

## 路由钩子（导航守卫）

### 全局钩子

#### beforeEach

```js
router.beforeEach((to, from, next) => {
  // ...
})
```

异步回调，按照钩子挂载的顺序执行。有点类似于tapable的waterfall任务队列。所有的回调必须执行next才能进行下一步。

next可以做以下事情：
* 中断路由跳转
* 跳转到一个不同的地址
* 中断跳转并抛出错误

#### beforeResolve
和 router.beforeEach 类似，在 beforeEach 和 组件内beforeRouteEnter 之后，afterEach之前调用。

#### afterEach
```js
router.afterEach((to, from) => {
  // ...
})
```

路由切换后的回调。无法修改路由。

### 路由内钩子

#### beforeEnter

和beforeEach一样。区别是，这个钩子是挂在特定路由下的，表示只有跳转到当前路由前才执行的钩子。

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

### 组件内钩子

#### beforeRouteEnter
```js
beforeRouteEnter(to, from, next) {
  // 在渲染该组件的对应路由被 confirm 前调用
  // 不！能！直接获取组件实例 `this`
  // 但是可以在next回调获取`this`
  // 因为当守卫执行前，组件实例还没被创建
  // next一定要调用
}
```

#### beforeRouteUpdate
```js
beforeRouteUpdate(to, from, next) {
  // 在当前路由改变，但是该组件被复用时调用
  // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
  // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
  // 可以访问组件实例 `this`
  // next一定要调用
  // next无需传参
}
```
#### beforeRouteLeave
```js
beforeRouteLeave(to, from, next) {
  // 导航离开该组件的对应路由时调用
  // 可以访问组件实例 `this`
  // next一定要调用
  // next无需传参
  // next(false)可以阻止跳转
}
```

## 完整的导航流程

1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 (2.5+)。
9. 导航被确认。confirmed
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。