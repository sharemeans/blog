---
title: fabricjs+opencv实现橡皮擦+画笔功能
categories: canvas
tags: [canvas]
date: 2022-3-20
---

## 需求背景

有一张物体图片，物体被AI识别后会对应显示出物体区域的半透明遮罩图层。

有时候AI识别会不准确，需要人工干预去补充缺失的区域，或者擦除掉多余的区域：
<img src="https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2022-3-13/1647157387754-image.png" width = "300" align=center />
<img src="https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2022-3-13/1647157635171-image.png" width = "300" align=center />

这两个功能分别用到了fabricjs的“画笔”和“橡皮擦”功能。

### 画笔
> PencilBrush类

```
canvas.freeDrawingBrush = new fabric.PencilBrush(this._canvas)
```

##### 开启画笔模式

```
canvas.isDrawingMode = true
```

##### 设置画笔颜色和粗细

```
canvas.freeDrawingBrush.width = brushSize * 2
canvas.freeDrawingCursor = 'none'
canvas.freeDrawingBrush.color = 'rgba(255,0,0,0.2)'
```

画笔刷默认的cursor是crosshair，觉得不好看的话可以模拟一个画笔，跟随鼠标移动。

##### 模拟画笔cursor
设置画笔为描边空心圆：

```
const followCircle = new fabric.Circle({
  radius: brushSize,
  left: centerX, // 初始位置
  top: centerY,
  stroke: '#05C1C1',
  strokeWidth: 1 / canvasScale,
  fill: 'transparent'
})
```
鼠标原本的cursor隐藏：

```
canvas.hoverCursor = 'none'
```
由于画布会缩放，为了保证画布缩放时，描边不会跟着变粗/变细，需要在`canvasScale`变化时及时更新画笔描边：

```
followCircle.setOptions({
    strokeWidth: 1 / canvasScale
})
```
监听鼠标移动，画笔跟随鼠标移动。为了让虚拟画笔和真正的笔刷轨迹一致，需要将虚拟画笔的中心设置为鼠标的位置：
```
canvas.on('mouse:move', () => {
  if (!followCircle || !e.absolutePointer) return
  // 获取鼠标相对画布的地址
  canvas.remove(followCircle)
  followCircle.setPositionByOrigin(e.absolutePointer, 'center', 'center')
  canvas.add(followCircle)
  canvas.renderAll()
})
```
效果如下：

<img src="https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2022-3-13/1647185848779-image.png" width = "300" align=center />

由于画笔是在画布上叠加画上去的，因此在画的过程中和遮罩层重叠的区域并不能融合，颜色会产生叠加。不过，可以在一次连续轨迹结束（mouse:up）之后运用图像处理工具将区域融合。


但是，实际使用过程中发现这种方法有个致命的缺点：性能很差，更新模拟画笔的位置的代价是更新画布上的所有元素，会造成视觉上以及交互上的卡顿感。有以下方法可以解决这个问题：
1.采用style.cursor属性中的url自定义cursor
2.采用独立的canvas容纳模拟鼠标，设置鼠标移动监听器，在监听器回调中使用fixed定位更新位置。

其中，方法1也有问题：css自带的cursor有大小限制，且不同浏览器的限制不同。

因此采用方法2。具体过程不再赘述。


##### 画笔区域生成图片
画笔抬起时，我们可以获取到一条轨迹：

```
const saveBrushResult = ({ path }: { path: fabric.Path }) => {
    //检测和遮罩是否有交集，否则直接将path从画布移除
    if (!path.intersectsWithObject(this._currentMask)) {
      this._canvas?.remove(path)
      return
    }
    // 移除画笔路径
    this._canvas?.remove(path)
    
    // TODO 利用path生成一张和遮罩一样大小的图片
}
canvas.on('path:created', saveBrushResult)
```
融合之前，首先要生成一张和遮罩一样大小的图片。

目前的情况是这样的：
* 不管画布如何缩放，path的比例默认为1，即，如果path视觉上宽度占了canvas的一半，那么，它的width就是canvas width的一半。
* 遮罩的宽高取决于图片。为了让canvas 将图片全部显示出来，因此基本是缩放过的。
* 我们需要根据path生成和原图一样大小的图片，用来和原图的遮罩做叠加计算。

所以，思路是这样的：

1. 获取遮罩的缩放比例，反过来作为path的缩放比例。
2. 计算path相对与图片的left和top

举个例子：

假设原图遮罩是**700\*1000**，在canvas上的位置为**left:20 & top:20**在画布上是缩放**0.5**倍，即**350\*500**。

而path在canvas的大小为**100\*30**，在canvas上的位置为**left:32 & top:32**，即相对与画布上的图片而言为**left:12 & top:12**。

我们需要将path变为2倍（1/0.5），即**200\*60**，且相对图片的真实位置为**left:24 & top:24**

计算方法如下：

```
const multiplierX = 1 / maskObject.scaleX
const multiplierY = 1 / maskObject.scaleY
// 先将path放大为和原遮罩一致
path.setOptions({
  scaleX: multiplierX,
  scaleY: multiplierY,
})
const canvasEle = path.toCanvasElement({
  format: 'png',
  quality: 1,
  // 计算path相对于遮罩的位置，并从遮罩边界处裁剪path
  left: multiplierX * (maskObject.left - left),
  top: multiplierY * (maskObject.top - top),
  
  width: maskObject.width,
  height: maskObject.height
})
```
**maskObject.width**和**maskObject.height**就是遮罩的原图大小即**700\*1000**

其实一开始我用的是另外一种方式：

```
const canvasEle = path.toCanvasElement({
  format: 'png',
  quality: 1,
  multiplier: 1 / maskObject.scaleX,
  left: maskObject.left - left,
  top: maskObject.top - top,
  width: maskObject.width! * maskObject.scaleX,
  height: maskObject.height! * maskObject.scaleY
})
```

这种方式先将图片设置为画布上的遮罩图尺寸，再通过`multiplier`进行放大
。

表面上看起来没错，实际上有个致命问题：最终的图片宽高可能和遮罩差一两个像素，原因是：**scaleX**是浮点数，**1 / scaleX**也是浮点数，二者相乘和原图有误差的可能性还是很大的。

##### 图片融合
原遮罩图，和画笔路径图已经生成了。怎么融合呢？此处采用opencv，步骤如下：
1. 使用`cv.imread`方法获取2张图片的imageData

```
const blobToDataURL = (blob: Blob): Promise<string> => {
  return new Promise((resolve, reject) => {
    let reader = new FileReader();
    reader.onload = function () {
      resolve(reader.result as string);
    }
    reader.onerror = function () {
      reject();
    }
    reader.readAsDataURL(blob);
  })
}

const blob2Mat = (blob: Blob): Promise<any> => {
  return new Promise((resolve, reject) => {
    blobToDataURL(blob)
      .then((dataUrl: string) => {
        const img = new Image()
        img.src = dataUrl
        img.onload = () => {
          const mat = cv.imread(img)
          resolve(mat)
        }
      })
      .catch(err => {
        console.log('[Error:blob2Mat] blobToDataURL fail', err)
        reject()
      })
  })
}

// 获取2张图片的像素矩阵
const mask1Mat = await blob2Mat(baseMask)
const mask2Mat = await blob2Mat(addMask)
```

合并矩阵：

```
const mulMat = new cv.Mat()
cv.add(mask1Mat, mask2Mat, mulMat)
```

由于合并后的图片选区交叉部分颜色也会重叠，我们需要把二者的选区先合并，再对选区内的颜色重置：

非选区部分，颜色为透明黑色，因此可以将背景抠出来：

```
const bgGrayMat = new cv.Mat()
// 非mask区域
const bgStartScalar = new cv.Mat(mulMat.rows, mulMat.cols, cv.CV_8UC4, new cv.Scalar(0, 0, 0, 0));
const bgEndScalar = new cv.Mat(mulMat.rows, mulMat.cols, cv.CV_8UC4, new cv.Scalar(0, 0, 0, 255));
cv.inRange(mulMat, bgStartScalar, bgEndScalar, bgGrayMat)
```
注意，inRange抠出来的是灰度图。

根据背景，可以调用cv.subtract取反获取选区灰度图：

```
let onesMat = new cv.Mat(bgGrayMat.rows, bgGrayMat.cols, bgGrayMat.type())
onesMat.setTo(new cv.Scalar(255))
cv.subtract(onesMat, bgGrayMat, garyMat);
```

然后再对选区和背景区域换色：

```
// 选区颜色改为半透明红色
cv.cvtColor(garyMat, garyMat, cv.COLOR_GRAY2RGBA)
garyMat.setTo(new cv.Scalar(255, 0, 0, 100), garyMat)
// 背景色改为透明黑色
garyMat.setTo(new cv.Scalar(0, 0, 0, 0), bgGrayMat)
```
garyMat现在就是一张背景为透明，选区为半透明红色的图片了。将数据导出到canvas后，可以调用toDataURL或者toBlob生成base64或者blob格式：

```
const canvas = document.createElement('canvas')
const canvasCtx = canvas.getContext('2d')
let imgData = new ImageData(new Uint8ClampedArray(mat.data), mat.cols, mat.rows)

canvas.width = mat.cols
canvas.height = mat.rows
canvasCtx.putImageData(imgData, 0, 0)
```

### 橡皮擦

> EraserBrush类

```
canvas.freeDrawingBrush = new fabric.EraserBrush(this._canvas)
```

##### 开启橡皮擦模式

```
canvas.isDrawingMode = true
```

##### 设置橡皮擦大小

```
canvas.freeDrawingBrush.width = eraserRadius * 2
canvas.freeDrawingCursor = 'none'
```
同理，我们用同画笔一样的方式模拟一个可以自定义形状的橡皮擦出来。

##### 橡皮擦擦除

这一步是实现橡皮擦功能核心。EraserBrush类继承于PencilBrush。区别在于，橡皮擦内部轨迹和图层叠加的方式区别。

canvas图层叠加有一个默认的属性[`globalCompositeOperation`](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Compositing#globalcompositeoperation)。

默认的叠加策略是`source-over`，即在已有图形上画新图形。

针对橡皮擦，使用的是`destination-in`策略
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture20220406074124.png)

原图层就是被擦除的部分本身，叠加图层就是橡皮擦路径。橡皮擦路径是一张黑底，白色路径的图片。

二者叠加，只会现有内容中，和叠加图层重叠区域，且叠加图层的颜色图案是什么都不影响叠加结果。

##### 仅擦除遮罩图层

fabric.Object对象有个`eraserable`属性。橡皮擦会单独针对每个可擦除图层做上一步操作，然后再对擦除结果图层采用`source-over`策略合并。

这一步，对fabric使用者来说，是不可见的。EraserBrush类内部已封装好相关逻辑。

这一步有个地方需要注意。fabric针对橡皮擦路径图层有缓存，内置了[缓存限制](http://fabricjs.com/docs/fabric.html#.perfLimitSizeTotal)以防止爆内存：
- perfLimitSizeTotal 限制canvas width*height <= perfLimitSizeTotal
- maxCacheSideLimit 限制width < maxCacheSideLimit且height < maxCacheSideLimit

默认perfLimitSizeTotal为2097152（2\*1024\*1024)。如果图片尺寸太大，**橡皮擦路径图层**和**图片图层**叠加时会被缩小。在**叠加回最终图层**前才放大，这就导致一个问题：放大后的遮罩图层边界和原始边界有1～2个像素的偏差，这个问题非常的隐蔽，以至于我花了一整天的时间排查原因。

选区遮罩对像素边界是非常敏感的，针对这个问题，我的解决办法是放大`perfLimitSizeTotal`值，保证橡皮擦路径融合时使用的是原尺寸。


##### 导出擦除后的图层

这一步其实是上一步**`destination-in`图层叠加策略**，只不过是重新执行了一遍，这一步也是fabric内置的：


```
ImageObject.setOptions({
  left: 0,
  top: 0,
  scaleX: 1,
  scaleY: 1
})
const canvasEle = ImageObject.toCanvasElement({
  format: 'png',
  left: 0,
  top: 0,
  width: ImageObject.width,
  height: ImageObject.height
})
canvasEle.toBlob(async (blob) => {
    // TODO 遮罩图层最终的blob
})
```

`ImageObject`就是原遮罩图层对应的fabric对象。你可能会好奇：那橡皮擦图层呢？其实，橡皮擦图层作为`ImageObject`的`clipPath`属性存在。调用`ImageObject.toCanvasElement`时，fabric会使用**`destination-in`图层叠加策略**对二者叠加输出到`canvasEle`。

至此，橡皮擦功能完成。



