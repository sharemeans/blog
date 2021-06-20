---
title: proxy 与 reflect
categories: javascript
tags: [javascript]
date: 2020-3-15
--- 
## Proxy

### 定义
> 在目标`对象`上架设一个`拦截`层， `外界`对该对象的`访问`先经过这一个`拦截`层，这个拦截层可以对外界对访问进行`过滤`和`改写`。翻译为`代理器`。

举个`拦截`和`改写`例子：

```javascript
let proxy = new Proxy({}, {
  get: function() {
    return 35
  }
})

proxy.time // 35
proxy.name // 35
```

上面对代码拦截了一个空对象的属性getter。只要是需要访问到对象的属性，都会触发getter。

### Proxy 能拦截哪些`访问`行为呢？

拦截行为 | 触发条件 | 返回值说明 
-|-|-
get(target, propKey, receiver)| 读取对象属性，proxy.foo，proxy['foo'] | 返回属性值 
set(target, propKey, value, receiver) | 设置对象属性，proxy.foo = v，proxy['foo'] = v | 返回一个布尔值，表示是否设置成功
has(target, propKey) | propKey in proxy的操作 | 返回一个布尔值，表示是否包含此属性
deleteProperty(target, propKey) | delete proxy[propKey] | 返回一个布尔值，表示是否删除成功
ownKeys(target) | Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环 | 返回目标对象所有自身的属性的属性名
getOwnPropertyDescriptor(target, propKey) | Object.getOwnPropertyDescriptor(proxy, propKey) | 返回属性的描述对象
defineProperty(target, propKey, propDesc) | Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs) | 返回一个布尔值表示是否定义成功
preventExtensions(target) | Object.preventExtensions(proxy) | 返回一个布尔值表示是否定义成功
getPrototypeOf(target) | Object.getPrototypeOf(proxy) | 返回一个对象（对象原型）
isExtensible(target) | Object.isExtensible(proxy) | 返回一个布尔值
setPrototypeOf(target, proto) | Object.setPrototypeOf(proxy, proto) | 返回一个布尔值
apply(target, object, args) | 拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...) | 无限制
construct(target, args) | 拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args) | 返回实例对象

## Reflect

### 定义

1. 将Proxy支持的代理方法一一对应到Reflect，方便在Proxy中使用原对象的默认行为。
```javascript
let proxy = new Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target, name, value, receiver);
    if (success) {
      console.log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
proxy.a = 1
```

2. 某些方法和Object原型方法行为高度一致，所以，除了在Proxy中使用，还可以单独使用。
```javascript
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```


## 谁在用

### Vue

#### 设置内置属性只读
设置内置keyCode为只读：
```javascript
const hasProxy =
    typeof Proxy !== 'undefined' &&
    Proxy.toString().match(/native code/)

  if (hasProxy) {
    const isBuiltInModifier = makeMap('stop,prevent,self,ctrl,shift,alt,meta,exact')
    config.keyCodes = new Proxy(config.keyCodes, {
      set (target, key, value) {
        if (isBuiltInModifier(key)) {
          warn(`Avoid overwriting built-in modifier in config.keyCodes: .${key}`)
          return false
        } else {
          target[key] = value
          return true
        }
      }
    })
  }
```

```javascript
Vue.config.keyCodes.ctrl = 86
```
以上代码会报错：
![](/images/2019071701.jpeg)

### 测试框架

### 和defineProperty什么关系？

## 什么时候适合用？