---
title: LLVM和wasm
categories: webassembly
tags: [wasm]
date: 2022-2-13
---



#### LLVM
LLVM是一系列编译器及相关工具链的几何，主要包含以下项目：

##### 1. LLVM核心
###### LLVM optimizer
> 通过编译参数选项，可以实现对IR代码进行分析和优化。

###### LLVM 后端编译器
> 将LLVM的IR 格式转化为平台相关的目标代码（汇编格式或机器码）

##### 2. CLang 前端编译器
LLVM前端C语言编译器（C/C++/Objctive-C），编译结果为LLVM-IR。LLVM-IR是一种类汇编语言，任何一种高级语言，在基于LLVM的规范上均可编译成IR格式。

##### 3. LLDB
结合CLang作为debugger工具。

##### 4. libc++
C++标准库的实现。需要和libstdc++区分开，二者都是C++标准库。libstdc++是gcc编译器开发的。而libc++能和CLang编译器更好的结合。

##### 5. libclc
OpenCL标准库C语言的实现。

##### 6. compiler-RT
编译运行时。通过软件层面实现硬件不支持的功能实现，比如：
- 四则运算、位运算、类型转换等基础功能。32机器实现64位无符号整型运算。

##### 总结
- LLVM是编译器、调试器、代码优化工具的基础工具的集合。
- 提供了LLVM-IR这种中间代码格式，方便进行代码优化与平台移植。

## Binaryen
一个编译器和基础工具库合集。目的是将将高级语言编译成webassembly。采用C++编写。

#### 优势：
- 简单。
- 快速。并行执行代码生成、优化。
- 高效。在多个环节实施代码体积和执行效率优化。

#### 工具集
Binaryen不仅是个编译器，还内置工具集。能实现以下功能：
- 解析生成wasm
- 解释执行wasm，方便执行测试用例
- polyfill。结合解释器使用，模拟javascript环境。

#### Binaryen IR
一种可以转化为wasm的中间格式。它的存在有以下目的：
- 便于优化wasm
- 便于和wasm之间互相转化

## webassembly
> 作为一个轻量、高效加载的编译目标，利用硬件优势达到原生的执行效率，wasm的最初目标是运行在web平台，并期望支持更多的运行平台，包括移动设备和loT。

wasm希望能够有更多的高级语言将其作为编译目标，LLVM作为一个强大的编译链路工具平台，能够更好的实现这个目标。

wasm既然是一种类IR的汇编语言，为什么不直接使用LLVM—IR呢？以下是我的理解：
- LLVM作为强大的高级语言编译链路工具平台，LLVM-IR作为关键节点，支持的指令和功能是相当完善的，和wasm的“轻便、高效”的目标并不一致。


参考资料：
- [Compiling to WebAssembly: It’s Happening!](https://hacks.mozilla.org/2015/12/compiling-to-webassembly-its-happening/)