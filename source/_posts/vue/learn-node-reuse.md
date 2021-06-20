---
title: 通过一个细节学习节点复用
categories: vue
tags: [vue, node-reuse]
date: 2019-12-6
---

## 背景

有一个图片列表，被transition-group包裹：
```
<template>
<div>
  <transition-group name="movee">
    <template v-for="(img, index) in images">
      <!-- key绑定为index -->
      <!-- <img class="image-item movee-item" :src="img" :key="index" alt=""> -->
      <!-- key绑定为img值 -->
      <img class="image-item movee-item" :src="img" :key="img" alt="">
    </template>
  </transition-group>
  <div>
    <button @click="swapImage">第一张和第二张交换顺序</button>
  </div>
    
</div>
 
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      images: [
        '/static/image1.png',
        '/static/image2.png',
        '/static/image3.png'
      ]
    }
  },
  methods: {
    swapImage() {
      let first = this.images.shift()
      let last = this.images.pop()
      this.images.unshift(last)
      this.images.push(first)
    }
  }
}
</script>

<style>
.image-item {
  height: 200px;
}
.movee-item {
  transition: all 0.3s;
}
</style> 
```
当for循环绑定的key为index时，没有任何动画；当key为img值时，效果如下：
![](/images/2021061401.gif)

为什么key绑定为img值，过渡效果就生效了呢？带着这个问题重新学习了一下vue的节点复用。

## key绑定

根据官方文档的说法，独特的 key，可以用于强制替换元素/组件而不是重复使用它。当你遇到如下场景时它可能会很有用：
* 完整地触发组件的生命周期钩子
* 触发过渡

那么，vue是怎么判断节点复用的呢？

vue的节点树存在vDOM中。包含了节点的所有信息。当template中绑定的data属性发生变化，就会触发新的虚拟节点生成。新旧虚拟节点会进行对比，可以复用的节点不需要重新渲染到DOM中。

同一个层级下，相同的虚拟节点才可以复用真实DOM，复用其实就是把节点对应的整个element对象粘贴到新的虚拟节点elm属性值上。新旧节点需要同时满足以下条件才能判定为相同：
  * key 相同（不绑定key 的情况下也相同，因为都是null）
  * tag 相同（没有tag的情况下也相同，如组件和文本节点）
  * 如果是输入框，输入框类型也要相同

贴出源码更加直观：
```
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}

function sameInputType (a, b) {
  if (a.tag !== 'input') return true
  let i
  const typeA = isDef(i = a.data) && isDef(i = i.attrs) && i.type
  const typeB = isDef(i = b.data) && isDef(i = i.attrs) && i.type
  return typeA === typeB || isTextInputType(typeA) && isTextInputType(typeB)
}
```
> 需要注意的是，输入框等表单项的value并没有作为判断依据。即，如果input的所有属性都一样，就会被认为可以复用，input并不会被重新渲染。这也解释了为什么会存在2个输入框交换顺序后绑定值和之前的顺序一致。

关于对容一个层级的理解，看下图就清楚了：
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-14/1623672057253-image.png)

对于整个树状的vDOM，对比过程就是深度遍历的过程。
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-14/1623672201578-image.png)



## 节点复用的图示
vue是如何对比同一个层级新旧子节点的呢？它其实是2种方法的结合：
* 两两对比交叉验证
* 绑定key的情况下，保存一份旧的子节点key:index键值对

两两对比始终是比较消耗性能的，这也是为什么vue针对for循环要求我们绑定key。

以同一个层级的新旧列表为例，假设数组的顺序变更为：**[A, B, C, D, E] => [F, B, A, E, C, G]**，用图展示绑定key为value的过程。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-16/1623804238240-image.png)
<div style="color: #999;padding: 2px;">👆4个箭头分别为oldStartIndex，oldEndIndex，newStartIndex, newEndIndex。旧A-新F，旧A-新G，旧E-新F，旧E-新G这4对对应的vNode进行比较。</div>

<div style="color: #999;padding: 2px;">👆由于绑定的key值不同，认定为不同的节点。接下来将通过key:index映射来尝试找到newStartNode。</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623889372354-image.png)
<div style="color: #999;padding: 2px;">👆newStartNode通过key也没找到，因此新建一个DOM元素，插入到oldStartNode指向的DOM节点之前</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623889877910-image.png)

<div style="color: #999;padding: 2px;">👆newStartVnode已经完成DOM创建和插入，接下来右移newStartIndex</div>
<div style="color: #999;padding: 2px;">👆新B-旧A，新B-旧E，新G-旧A，新G-旧E这4对又开始对比（这里发现有个问题，新G-旧A，新G-旧E重复对比了，这算不算一个优化点呢？vue@2.6.11）。</div>
<div style="color: #999;padding: 2px;">👆对比结果又是没匹配上。新B通过key:index映射找到了原身，旧B对应的DOM节点则移动到oldStartNode的前面。</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623890393456-image.png)

<div style="color: #999;padding: 2px;">👆新B的DOM节点已经安顿好了，新B对应的old vNode位置也对应从数组删除，为了不影响现有索引位置，只是old vNode的值设置为undefined。newStartIndex右移一位。</div>
<div style="color: #999;padding: 2px;">👆新A-旧A识别为相同的节点，由于都是startIndex，因此二者对应的DOM节点在父元素中的位置保持不变。oldStartIndex和newStartIndex右移一位</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623939477940-image.png)

<div style="color: #999;padding: 2px;">👆新E-旧E识别为相同节点。旧E（oldEndIndex）对应的DOM移动到旧C（oldStartIndex）对应的DOM节点之前👇</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623939669940-image.png)

<div style="color: #999;padding: 2px;">👇oldStartIndex和newStartIndex右移一位，oldStartIndex遇到旧B的位置为undefined，继续右移。</div>

<div style="color: #999;padding: 2px;">根据上一轮的匹配结果，oldEndIndex对应vNode置空，oldEndIndex左移，newStartIndex右移👇</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623940208168-image.png)

<div style="color: #999;padding: 2px;">👆新C-旧C识别为相同的节点，由于都是startIndex，因此二者对应的DOM节点在父元素中的位置保持不变。oldStartIndex对应的vNode置空，oldStartIndex和newStartIndex右移一位。</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623940918268-image.png)

<div style="color: #999;padding: 2px;">👆oldStartIndex和oldEndIndex相遇，newStartIndex和newEndIndex相遇。新G-旧D无法识别为相同节点。通过key:index映射也无法匹配上，说明G是新增节点。</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623941240177-image.png)

<div style="color: #999;padding: 2px;">👆针对G新建DOM节点，插入oldStartIndex对应DOM节点之前。</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623941521350-image.png)

<div style="color: #999;padding: 2px;">👆由于新G已安顿好，newStartIndex右移，但是越界，因此循环终止。</div>

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-17/1623941771908-image.png)

<div style="color: #999;padding: 2px;">👆删除oldStartIndex和oldEndIndex之间的vNode以及DOM节点。</div>

## 总结

* 循环条件：旧startIndex <= 旧endIndex 且 新startIndex <= 新endIndex
    
    * 若旧startIndex 与 新startIndex 匹配，则二者均右移，不需要操作DOM顺序，继续新一轮循环
    * 若旧startIndex 与 新endIndex 匹配，则说明处于当前对比区间最后面，将DOM节点移动到旧endIndex之后。新endIndex左移，继续新一轮循环。
    * 若旧endIndex 与 新startIndex 匹配，则说明处于当前对比区间的最前面，将DOM节点移动到旧startIndex之前。新startIndex右移，继续新一轮循环。
    * 若旧endIndex 与 新endIndex 匹配，则二者均左移，不需要操作DOM顺序，继续新一轮循环。
    * 若以上都不满足，则根据当前查找区间的key:index映射寻找新startNode对应的旧index。
        
        * 若找到匹配元素对应位置为idxInOld，则将idxInOld对应的DOM节点移动到旧startIndex前面。新startIndex右移，继续新一轮循环。
        * 若找不到，则新建一个DOM节点，插入到旧startIndex前面。新startIndex右移，继续新一轮循环。
* 若旧startIndex > 旧endIndex，则为新startIndex -> 新endIndex之前所有节点新建DOM节点并按顺序插入父节点的末尾。
* 若新startIndex > 新endIndex，则删除旧startIndex -> 旧endIndex之前所有节点的DOM节点

##  transition-group

以上分析过程只是普通的节点更新流程。如果一串节点被transition-group包裹，会发生什么呢？

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-18/1623976185660-image.png)

源码中，如果有transition-group包裹，可复用的DOM节点顺序是不会调整的，只会新增和删除。如以上例子 **[A, B, C, D, E] => [F, B, A, E, C, G] ** 对比结束后顺序DOM节点顺序将会是:

**[F, A, B, C, E, G]**：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-20/1624192060131-image.png)

接下来是实施过渡的步骤：

1. 记录当前各个DOM节点的边界位置
2. children更新，触发render，记录旧DOM节点的边界信息（getBoundingClientRect），重新渲染新DOM（对的，没有过渡，直接按照新的顺序渲染）
3. 触发updated钩子，记录新DOM节点的边界信息
4. 遍历所有children cNode，若同时存在新旧位置信息，说明是复用节点，通过transform将位置重新调整到旧位置（对的，立马设置回旧的位置，前面渲染出来的效果时间很短，用户视觉上看不到，可以通过在transition-group组件的updated钩子加断点看到）
5. 通过读取`document.body.offsetHeight`触发重排
6. 将children所有节点再设置回新位置，并添加过渡类
7. 主线程执行完，开始重排，此时会显示过渡效果

关于transition-group这里有个小问题：为什么transition-group不立即更新DOM节点？

因为需要一个过渡效果，不能立即切换为终点状态。过渡过程完全交给transition-group处理。


## 回到一开始的问题

为什么key绑定为img值，过渡效果就生效了呢？

* 若key绑定为img时，img相同的图片才会被视为相同节点，会被记录移动前后的位置，因此有过渡效果。
* 若key不绑定，或者绑定为index，那么相同index的图片被视为相同节点，每个节点的位置都没有变化，因此没有过渡效果。
