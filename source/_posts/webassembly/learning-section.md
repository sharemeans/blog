---
title: 徒手撸wasm之：认识wasm
categories: webassembly
tags: [wasm]
date: 2022-1-7
---

### 前言

WebAssembly是一种字节码。它可以是任意一种高级语言的中间代码。目前go、rust、c++、python甚至typescript都有自己的wasm编译器。

字节码属于二进制格式，要想执行它，需要一个运行时或者编译器将其转化为平台相关的机器码。目前所有浏览器都实现了wasm的MVP标准，本质上是在遵循这个标准的基础上实现了一个wasm的运行时。

因此，wasm不是浏览器独占的，任何平台想要执行它只需要同样提供对应的运行时即可。只不过，浏览器由于性能瓶颈对wasm对的需求更加旺盛，其周边生态也越来越完善。

## 字节码结构
和所有高级语言的中间代码结构一样，wasm有一套自己的规范。这套规范包含：
- 指令集
- 数据类型
- 模块分区
- 文本格式


这些规范之间的关系如下：
- 指令集就是命令，有语法结构相关对控制指令，如if、loop；有内存读写相关，如load、store；有数值操作相关的，如add、sub；指令之间通过操作码区分。
- 数据类型。wasm不像javascript一样灵活，数据类型只有整数，浮点数，大小分为32位和64位。对于数据操作相关的指令，每种类型都有对应的指令；如i32和i64对应的add指令为：i32.add，i64.add，二者具有不同的操作码。
- 模块分区。模块是wasm编译的基本单位。模块分区是一种提高代码解释效率的约定，比如解释器可以针对每个类型的分区开一个线程，每个分区都有固定的id，线程只有在遇到目标分区时才解释对应代码，以达到并行编译的目的，进而提高执行效率。
- 文本格式。wasm是字节码，为了便于调试和阅读可以转化为文本格式wat，所有的指令集和数据类型都有对应的“助记符”。

指令集、数据类型、模块分区的编码参考[二进制编码](https://www.wasm.com.cn/docs/binary-encoding/)

## 模块分区
模块分区是wasm 二进制文件的概念。为了方便讲解，我们结合文本格式讲解。wasm的内容从上到下有以下几个分区（Section）：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2022-1-9/1641722453317-image.png)

在wasm中，分区必须按照section ID从小到大的顺序排列，即`Type Section`一定要在其它section前面，以此类推。

简单整理下各个分区的关系。

##### Type Section / Function Section / Code Section
`Type Section`可以理解为函数签名，类似于interface。

`Function Section`可以理解为函数声明。定义了函数的调用名，函数继承的函数签名。

`Code Section`为函数内容。

##### Memory Section / Data Section
`Memory Section`内存段用于声明一块内存空间，` Data Section`数据段将数据写入内存段中。这块先一笔带过，后面有实例讲解更加清晰。

##### Table Section / Element Section
`Table Section`和`Element Section`的关系，类似于`Memory Section`和`Data Section`。区别在于，`Table Section`仅用于存放函数引用，这对于执行动态函数非常有用。

##### Import Section / Export Section
导入导出段。导入和导出的wasm的类型支持：

tag Id | 说明
---|---
0 | 函数
1 | 表
2 | 内存
3 | 全局变量

也即是说，通过导入导出，可以实现与宿主环境共享全局变量、内存、函数、以执行及动态函数。

##### Global Sectionn
顾名思义，列出模块内的所有全局变量。

##### Start Section

表示初始化需要执行的函数，只需要记录函数索引即可。

##### Custom Section
自定义段。不同于其它段，自定义段可以放在任何段的前后，通常用来存放调试信息或者第三方扩展信息，对模块执行无影响。

参考：
- [WebAssembly 模块的基本组成结构](https://time.geekbang.org/column/article/284554)
- 《WebAssemby原理与核心技术》张秀宏
