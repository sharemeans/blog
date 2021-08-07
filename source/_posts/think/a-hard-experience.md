---
title: 一次被折磨到头疼的经历
categories: 思考
tags: [学习, debug]
date: 2021-7-8
--- 

## 背景
我们有个多应用公共仓库，不同应用打包时需要不同的manifest.json文件。

每个应用都有一个manifest.xxx.json文件，打包时需要使用该文件替换掉manifest.json。我们使用了nodeJs的child_process.exec来执行cp命令：

```shell
cp -f F:\pathToJson\manifest.xxx.json F:\pathToJson\manifest.json 
```
在某个同事的windows电脑上执行时报错：

```
`Uncaught Error: Command failed: cp -f F:\pathToJson\manifest.xxx.json F:\pathToJson\manifest.json

at ChildProcess.exithandler (child_process.js:303)
at ChildProcess.emit (events.js:182)
at maybeClose (internal/child_process.js:961)
at Process.ChildProcess._handle.onexit (internal/child_process.js:248)`
```

这个报错信息只是简单的说明child_process中断，并没有详细的错误原因。

## 摸索

试着在cmd中执行一样的命令，发现报错：

```
系统找不到指定文件
```

为啥？？这个文件的确是存在的哇？我换了个文件，也是一样的报错。目前还不清楚是什么原因。

根据网上的经验，尝试过以下可能原因：
- 文件路径名太长
- 出现了中文路径
- 权限问题
- 试下其它的文件类型

发现以上可能性都排除。

接着在powershell中执行一样的命令，竟然成功了。作为命令行小白，去查了下二者的区别，大致结论是：powershell就是cmd的超集，不仅可以调命令，还可以连接数据库甚至编程。然而这些信息并没有给我带来灵感。

无路可走的我尝试对这句命令进行各种改参数尝试，发现删掉`-f`参数就成功了。

基于这一点线索，我尝试将参数改成-i，-d等都是一样的报错，我得出1个结论：cp后面的参数都是多余的！基于以前了解过命令别名，瞬间得出结论：cp命令本身包含了某个参数！换句话说，cp本身是某个命令别名！

在`C:\windows\system32\`目录下发现了一个cp.bat文件，里面赫然写着：

```
@echo off  
DOSKEY cp=copy 
```

所以啊，cp命令本身是Linux系统的，为什么之前就没质疑过呢？害得我浪费了这么长时间。

先把命令别名这个干扰项删除，然后再执行一次：
```shell
cp -f F:\pathToJson\manifest.xxx.json F:\pathToJson\manifest.json 
```
这次报错变为：

```
cp不是内部或外部命令,也不是可运行的程序
```
因此，接下来的问题是，NodeJs在调用child_process.exec时，使用的命令行工具是哪个呢？



## 结论
- 如果对命令行不熟的话就不要随便对命令行改造，否则在遥远对将来将成为一个巨坑。
- 当对一个问题产生怀疑时，不要轻易让它擦肩而过。