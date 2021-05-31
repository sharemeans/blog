---
title: vue 的mount 过程
categories: vue
date: 2020-4-2
---

>> 前段时间学习了vue的源码，理清了vue的mount流程，在此做下总结。

![](/images/2019051301.svg)

mount过程分为2个大阶段，生成vnode，和渲染dom

## 生成vnode

mount方法其实调用了render函数，该函数返回的就是vnode

## 渲染 dom

这个过程主要调用了update方法。每一个vnode的渲染都会调用一次update方法。

针对组件，render返回的对应vnode只是一个占位vnode。在update的时候占位vnode会根据组件配置生成所有内容的dom。

渲染过程其实是一个深度遍历的过程。即，最先生成子dom，逐个层级向上插入，直到插入根结点的dom中。
