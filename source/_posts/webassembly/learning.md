---
title: webassembly 初学笔记
categories: webassembly
tags: [wasm]
date: 2021-10-23
---

> 最近在学webassembly和webGL的，看看能不能多产出几篇学习笔记。

#### 理解
##### 运行原理
二进制格式文件，充分发挥硬件能力提高执行效率。不是编程语言，是一种中间格式，处于高级语言和汇编语言之间，无需关注运行平台。

##### 运行环境
- web环境
- 非web环境

##### 关键概念
- Module。被浏览器编译过的wasm二进制字节码。可使用类似es6的方式导入导出。
- Memory。可伸缩的ArrayBuffer。wasm的内存读写指令对其进行读写操作。
- Table。不能作为原始字节存储在内存里的对象的引用，如函数。
- Instance。wasm模块运行时所包含的Module、Memory、Table信息集合。

##### 特点/限制
1. 不能直接存取DOM，只能通过调用javascript来吊椅哦那个Web API，即胶水代码（针对浏览器环境）
2. 只能传入整数和浮点数

##### 胶水代码（javascript）
1. 调用浏览器提供的webassembly API，实现获取、加载、运行wasm文件
2. 部分胶水代码实现了高级语言（C/C++）的部分库

##### webassembly API

###### compile/Instance
wasm是1个二进制文件，通过fetch方法读取出来的是arrayBuffer。该传入compile方法接收arrayBuffer生成模块module。module是无状态的，类似class，需要instantiate实例化。

###### memory与data
memory表示一段连续的无类型字节。通过读写指令操作内存，但是每个wasm实例可操作的内存被限制在一个wasm Memory对象范围内。

wasm中内存是以page为单位的，1个page为2^16即25536字节，即64KB。目前规范限制一个wasm实例最多使用65536个page，即4GB。

```
var memory = new WebAssembly.Memory({initial:10, maximum:100})
```
该内存对象会被传入wasm中，供wasm内部使用。

data用于往内存写入数据：

```
(module
    (memory 1)
    (data (offset (i32.const 0)) "Hi") ;; 使用数据段把字符串写入全局内存中
)
```
###### 共享内存



###### table与elem
js和wasm都可访问的“带类型的数组”，空间可扩展。目前仅能存储“anyfunc”类型的引用。出于安全/可以指/稳定性可考虑，引擎信任的引用值不能被直接读写。

静态函数可以直接通过定义1个引用，并通过call指令调用（如下面的例子：log.wat）。

但是如果call调用的是动态的函数索引，这些索引需要指向table中的元素（通过elem声明），且这些元素需要通过`call_indirect`指令调用。

```
// js
fetchAndInstantiate('wasm-table.wasm').then(function(instance) {
  console.log(instance.exports.callByIndex(0)); // 返回42
  console.log(instance.exports.callByIndex(1)); // 返回13
  console.log(instance.exports.callByIndex(2));
  // 返回一个错误，因为在表格中没有索引值2
});


// wasm-table.wat
(module
  (table 2 anyfunc)
  (func $f1 (result i32)
    i32.const 42)
  (func $f2 (result i32)
    i32.const 13)
  (elem (i32.const 0) $f1 $f2)
  (type $return_i32 (func (result i32)))
  (func (export "callByIndex") (param $i i32) (result i32)
    local.get $i
    call_indirect (type $return_i32))
)
```

#### 运用
##### wasm调用javascript


```
// js
var importObject = {
  console: {
    log: function(arg) {
      console.log(arg);
    }
  }
};

fetchAndInstantiate('logger.wasm', importObject).then(function(instance) {
  instance.exports.logIt();
});


// log.wat
(module
  (import "console" "log" (func $log (param i32)))
  (func (export "logIt")
    i32.const 13
    call $log))
```

##### javascript调用wasm

```
// js
function fetchAndInstantiate(url, importObject) {
  return fetch(url).then(response =>
    response.arrayBuffer()
  ).then(bytes =>
    WebAssembly.instantiate(bytes, importObject)
  ).then(results =>
    results.instance
  );
}

fetchAndInstantiate('add.wasm').then(function(instance) {
   console.log(instance.exports.add(1, 2));  // "3"
});


// add.wat
(module
  (func $add (param $lhs i32) (param $rhs i32) (result i32)
    local.get $lhs
    local.get $rhs
    i32.add)
  (export "add" (func $add))
)

```

###### wasm操作非number类型
非number类型数据需要存储在内存中。
1. js自己创建1个内存实例传入wasm
2. 通过获取wasm内部的内存实例

以第一种为例：
```
// js
const memory = new WebAssembly.Memory({ initial : 1 })
function consoleLogString(offset, length) {
    var bytes = new Uint8Array(memory.buffer, offset, length);
    var string = new TextDecoder('utf8').decode(bytes);
    console.log(string);
  }
var importObject = {
  js: {
    mem: memory
  },
  console: {
    log: consoleLogString
  }
}

fetchAndInstantiate('./logger.wasm', importObject).then(function(instance) {
  console.log(instance.exports.writeHi());
});

function fetchAndInstantiate(url, importObject) {
  return fetch(url).then(response =>
    response.arrayBuffer()
  ).then(bytes =>
    WebAssembly.instantiate(bytes, importObject)
  ).then(results =>
    results.instance
  );
}

// logger.wat
(module
    (import "console" "log" (func $log (param i32 i32)))
    (import "js" "mem" (memory 1))
    (data (i32.const 0) "Hi") ;; 使用数据段把字符串写入全局内存中
    (func (export "writeHi")
      i32.const 0
      i32.const 2
      call $log
    )
)
```

###### wasm实例之间共享memory和table

```
// js 
var importObj = {
  js: {
    memory : new WebAssembly.Memory({ initial: 1 }),
    table : new WebAssembly.Table({ initial: 1, element: "anyfunc" })
  }
};

Promise.all([
  fetchAndInstantiate('shared0.wasm', importObj),
  fetchAndInstantiate('shared1.wasm', importObj)
]).then(function(results) {
  console.log(results[1].exports.doIt());  // prints 42
});

// shared0.wat
(module
  (import "js" "memory" (memory 1))
  (import "js" "table" (table 1 anyfunc))
  (elem (i32.const 0) $shared0func) ;; 将$shared0func存入table作为第一个引用，偏移量为0
  (func $shared0func (result i32) ;; 将
   i32.const 0
   i32.load)
)

// shared1.wat
(module
  (import "js" "memory" (memory 1))
  (import "js" "table" (table 1 anyfunc))
  (type $void_to_i32 (func (result i32)))
  (func (export "doIt") (result i32)

   (i32.store (i32.const 0) (i32.const 42))  ;; store 42 at address 0 to memory

   (call_indirect $void_to_i32 (i32.const 0))) ;; 调用table的第一个引用
)
```

##### 协议支持wasm格式
使用`WebAssembly.instantiateStreaming`实例化wasm时，会有如下报错：

```
Uncaught (in promise) TypeError: Failed to execute 'compile' on 'WebAssembly': Incorrect response MIME type. Expected 'application/wasm'.
```

输出fetch方法返回的reponse对象后发现，类型为`application/octet-stream`。原来是因为本地服务启动工具不支持`application/wasm`类型：
1. [node.js express.js server failed to recoganize file type .wasm](https://stackoverflow.com/a/61826330)
2. [WASM modules are served with the wrong content-type](https://github.com/yandeu/five-server-vscode/issues/4)

由于live server迟迟不修复这个问题，换成five server插件就可以了。

##### 参考资料
- [An Abridged Cartoon Introduction To WebAssembly](https://www.smashingmagazine.com/2017/05/abridged-cartoon-introduction-webassembly/)
- [WebAssembly Specification](https://webassembly.github.io/spec/core/)
- [How webassembly works](https://www.techug.com/post/how-webassembly-works.html)
- [wasm文本语法](https://github.com/WebAssembly/spec/blob/master/interpreter/README.md#s-expression-syntax)
- [webassembly-examples](https://github.com/mdn/webassembly-examples)

#### demo及学习资料
###### wasm现状及提案
- https://webassembly.org/roadmap/
- https://github.com/WebAssembly/proposals

###### emscripten demo
emscripten是将C++编译为wasm的工具，其对wasm以及文本GL具有良好的支持。

- [demo](https://github.com/emscripten-core/emscripten/wiki/Porting-Examples-and-Demos)

###### 结合wasm的WebGL 着色器
- [博客](https://www.freecodecamp.org/news/how-to-use-webgl-shaders-in-webassembly-1e6c5effc813/)
- [源码](https://github.com/DanRuta/webassembly-webgl-shaders)

###### 基于wasm的机器学习框架：
- [jsNet](https://github.com/DanRuta/jsNet/tree/dev)


###### 使用TS编写源码，并编译成wasm的游戏模拟：
- [wasmboy](https://github.com/torch2424/wasmboy)





