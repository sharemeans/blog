---
title: 第99次学习原型链
categories: js
tags: [js, 原型链]
date: 2021-6-4
--- 

变量有基于类（构造函数）创建的，也有基于实例创建的。

简单类型有Number，Boolean，Null，Undefined，String，Symbol，RegExp
引用类型：Array，Object，Function，Set，Map

## 字面量创建的变量
```
let a = [1,2,3]
let b = {}
```

通过字面量创建的变量，底层会调用对应的构造函数，可以输出constructor属性看看：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628309631324-image.png)
字面量生成的变量原型是是这些内置类型的构造函数。

## 构造函数创建的变量
通过new创建的变量，调用构造函数本身，所以原型的constructor就是这个构造函数。
```
function MyClass(name) {
    this.foo = name
}

let myObj = new MyClass('bar')
```
myObj格式如下：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628309653718-image.png)
myObj.constructor取的是__proto__.constructor

Object.create
该方法比较特殊，会通过现有的对象实例创建新对象，新对象的原型是旧实例，而不是类或者构造函数。
```
let obj = {foo: 'bar'}
let obj2 = Object.create(obj)
```

打印obj2

可见，obj2.__proto__没有constructor属性，所以obj2.constructor会继续往上级寻找该属性，所以，obj2.constructor的值为Object函数

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628309680961-image.png)


## js类继承

### 一、改变this指向
  1. call实现构造函数继承
```
function Parent(name,age) {
  this.name = name
  this.age = age
}
Parent.prototype.say = function () {}

function Child (name,age) {
  Parent.call(this, ...arguments)
}

var c = new Child('joe', 12)

c.name = 'joe'
console.log(c.name);
console.log(c.say);
```

该方法只是通过将子类的this指向父类的构造函数并调用。仅仅是继承了父类的既有属性，并没有继承原型对象。
 
2. 实现call
```
var a = 1
function fn () {
  console.log(this.a)
}
fn() // 非严格模式：1 严格模式 ：Uncaught TypeError: Cannot read property 'a' of undefined
var obj = {
  a: 11
}
Function.prototype.myCall = function (context = window, ...args) {
  context.fn = this
  let res = context.fn(...args)
  delete context.fn
  return res
}

fn.myCall(obj) // 1
```

  call方法做的事情很简单，就是将函数挂到传入的对象上，这样通过属性调用执行函数时this自然就会指向对象。

3. this为什么是实例
老生常谈this的指向
* 函数中的this，不会像变量一样从父级作用域查找。函数中this直接指向window，严格模式下为undefined，除非有通过call或者apply重新绑定this。
* 函数作为对象属性时，this指向的是对象
* new后面的构造函数、实例方法调用，this指向都是实例
* 直接调用构造函数的原型方法，this指向当然是原型对象啦（这个函数是哪个对象调用的，this指的就是这个对象）
  
4. 实现myNew
```
function Parent(name,age) {
  this.name = name
  this.age = age
}
Parent.prototype.say = function () {}

function Child (name,age) {
  Parent.call(this, ...arguments)
}
function myNew (fn, ...args) {
  var obj = {
    __proto__: fn.prototype
  }
  var res = fn.apply(obj, ...args)
  return typeof res === 'object' ? res : obj
}

myNew(Child, 'joe')
```

  
### 二、原型链也要继承
先把基本代码写上：
```
function Parent(name,age) {
  this.name = name
  this.age = age
}
Parent.prototype.say = function () {
  console.log('say hello');
}

function Child (name, age, classNumber) {
  Parent.apply(this, arguments)
  this.classNumber = classNumber
}
```


下面说一下Child如何继承Parent 原型链。
1. Child.prototype = new Parent()
```
Child.prototype = new Parent()

var c = new Child('joe', 12, 3)
console.log(c.name); // joe
c.say()
```

这个方法，有个缺点，就是Parent函数执行了2次，一次是给Child.prototype赋值，一次是Child实例化。而且Child.prototype上会挂上多余的name和age属性。

2. Child.prototype = Parent.prototype
```
Child.prototype = Parent.prototype
var c = new Child('joe', 12, 3)
console.log(c.name); // joe
c.say()
```


这个方法，Child和Parent共用了原型对象，当我们想给Child的原型对象上增加Child专属的方法（如study）时，会导致Parent.prototype也会被同时修改。

3. Child.prototype = Object.create(Parent.prototype)
```
Child.prototype = Object.create(Parent.prototype)
Child.prototype.study = function(){
    console.log('go school')
}
```

Object.create通过拷贝一份prototype可以解决第二种方法的问题。

prototype.constructor属性现在指向Parent函数。但是new Child语法会自动执行Child函数，而不是直接执行Parent。而且instanceof也能通过验证
```
var c = new Child('joe', 12)
c instanceof Child // true
```

这一点超出预料，尽管从浏览器的输出结果，并没有找到Child的影子：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628309822567-image.png)
看来，Child.prototype.constructor = Child似乎不是很有必要。

4. 加上constructor
我还是要说，加上这句话：
```
Child.prototype.constructor = Child
```

  
why? 如果原型对象使用了this.constructor之类的语法，那它拿到的就是Parent:
```
// define the Person Class  
function Parent(name) {
    this.name = name;
}  

Parent.prototype.copy = function() {  
    return new this.constructor(this.name);
};  

// define the Student class  
function Child(name) {  
    Parent.call(this, name);
}  

// inherit Person  
Child.prototype = Object.create(Parent.prototype);
var child1 = new Child("trinth");  
console.log(child1 instanceof Child); // => true
console.log(child1.copy() instanceof Child); // => false

```

总之构造函数要像上面那张**构造函数-原型对象-实例**关系图一样完完整整。

## 类型判断

判断变量类型有多种方法，下表列出了这些方法的完整功能范围：

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628309920093-image.png)

