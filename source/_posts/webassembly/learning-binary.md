---
title: 徒手撸wasm之：认识wasm的函数
categories: webassembly
tags: [wasm]
date: 2022-1-9
---

## 前言
wasm是二进制格式，不方便编码，更不方便学习。我们只有通过认识wat结构才能更好的理解wasm。


wasm模块分为多个section，而wat分为“域”（这个词是从参考资料中）。今天先通过最简单的代码认识`Table section`和`Type Section`以及简单的指令。

严格来说，我们需要先了解wasm的二进制编码方式。二进制编码方式主要是为了解决多字节数据存储问题，不过为了方便学习，本程序涉及的编码仅在单字节范围内，仅放出ASCII码表参考就可以满足学习需求。

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2022-1-9/1641734781647-image.png)

再次把分区ID及顺序图放出来，方便理解：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2022-1-9/1641722453317-image.png)

## 示例

以下C++代码：

```
// add.cpp
int main () {
  int a = 1;
  int b = 1;
  return a + b;
}
```

用[emcc](https://emscripten.org/docs/tools_reference/emcc.html?highlight=emcc)编译：

```
emcc add.cpp -O0 -s
```

编译结果有个`add.wasm`。`hexdump`命令可以查看二进制文件的内容。命令行执行：

```
hexdump -C add.wasm
```
输出结果：
```
00000000  00 61 73 6d 01 00 00 00  01 07 01 60 02 7f 7f 01  
00000010  7f 03 02 01 00 07 07 01  03 61 64 64 00 00 0a 09  
00000020  01 07 00 20 00 20 01 6a  0b
00000029
```

安装[wabt](https://github.com/WebAssembly/wabt)，执行：

```
wasm2wat add.wasm -o add.wat
```

生成的wat文件：

```
(module
  (type (;0;) (func (param i32 i32) (result i32)))
  (func (;0;) (type 0) (param i32 i32) (result i32)
    local.get 0
    local.get 1
    i32.add)
  (export "add" (func 0)))

```

#### 分割符
wat使用[S表达式](https://github.com/WebAssembly/spec/blob/master/interpreter/README.md#s-expression-syntax)的语法规范。

简单概括：
- 圆括号是wat语言的分割符。
- 表达式之间要用分隔符隔开，形成并列或者嵌套结构。

#### 魔数与版本号
二进制开头的8个字节：
```
00/**/ 61/*a*/ 73/*s*/ 6d/*m*/ /*魔数：空字符asm*/
01 00 00 00/*版本1*/
```

#### Type Section
以下声明了一个函数签名，参数是2个i32类型，返回值是1个i32类型，type索引是0。
```js
 (type (;0;) (func (param i32 i32) (result i32))) // (;0;)表示索引为0
```

在wasm中的表示为：

```
01/*Type Section ID：1*/
07/*占7个字节*/
01/*总共有1个类型签名*/
60/*函数类型：0x60 [Language Types](https://www.wasm.com.cn/docs/binary-encoding/)*/
02/*2个参数*/
7f/*i32*/ 
7f/*i32*/ 
01/*1个返回值*/
7f/*i32*/ 
```

#### Function Section

```js
(func (;0;) (type 0) (param i32 i32) (result i32)  //  (;0;) 表示函数索引为0
    local.get 0
    local.get 1
    i32.add)
```
wasm中的Function Section仅做函数声明用，具体的函数内容交给Code Section。而在wat中，为了方便表示，二者结合在一起成为以上结构。


```js
03/*Function Section Id: 3*/
02/*2个字节*/
01/*仅一个函数*/
00/*使用索引为0的Type Section*/
```

由于`Export Section`的ID小于`Code Section`，二进制码在前面。我们先讲`Export Section`。
#### Export Section

```js
(export "add" (func 0)) // 导出索引为0的函数，导出名称为add
```


```
07/*Export Section ID：7*/
07/*占7个字节*/
01/*仅导出1个目标*/
// 以下为第一个导出目标的内容
03/*目标名称占3个字节*/ 
61 64 64/*add*/
00/*导出tag只为0，表示函数类型[external_kind](https://www.wasm.com.cn/docs/binary-encoding/#export-section)*/
00/*指向的函数索引为0*/
```

#### Code Section

接下来是代码段。代码段不会指明具体对应哪个函数，自动按照函数段函数索引的顺序一一对应。

我们再看回来唯一的函数：
```js
(func (;0;) (type 0) (param i32 i32) (result i32)  //  (;0;) 表示函数索引为0
    local.get 0
    local.get 1
    i32.add)
```

```
0a/*Code Section ID: 10*/
09/*占9个字节*/
01/*仅1个代码项*/
// 以下为第一个代码项的内容
07/*占7个字节*/ 
00/*0个局部变量*/ 
20/*指令：local.get*/ 
00/*获取第0个参数值*/ 
20/*指令：local.get*/
01/*获取第一个参数值*/ 
6a/*指令：i32.add*/
0b/*控制指令：end*/
```
有人可能会问，如果函数内部有局部变量，local.get指令如何区分参数和局部变量呢？很简单，参数和布局变量共用序列。比如函数有2个参数，那么第一个局部变量的序号就是3，索引值即是2。

至此，一个简单的函数解读完毕。后面我们在继续学习复杂的函数、其它分区、以及多字节编码。
