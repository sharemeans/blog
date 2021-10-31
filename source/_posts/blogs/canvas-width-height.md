---
title: canvas width height
categories: canvas
tags: [canvas]
date: 2021-10-31
---

canvas元素本身具有width和 height属性，它和css的width和height属性是什么关系呢？

### css的width和height
表示canvas画布在document占有的宽度和高度。直观表现就是，元素审查的时候可以看到canvas元素的大小。

### width和height属性
表示canvas可视区域内可以渲染的像素宽度和高度。

- 如果width属性和css的width是1:1的关系，那么画出来的图片也是1:1大小。
- 如果width:500，css width:1000，那么画出来的图片相当于被放大1倍，看起来会比较模糊。
- width:500，css width:1000的canvas，无论图片有多高清，塞进去之后都会很模糊，因为canvas本身就将css像素放大2倍了。
- 因此，width至少是css width的1倍大才会显得清晰。


### 图片在canvas中水平垂直居中

如何让无论多大的图片都可以放进去而且水平垂直居中呢？需要确定是否缩放、缩放比例，然后计算居中位置。

```
const canvas = document.getElementById('myCanvas')
const context2d = canvas.getContext('2d')
fileInput.addEventListener("change", event => {
    const imgEle = new Image()
    imgEle.src = URL.createObjectURL(event.target.files[0])
    imgEle.addEventListener("load", () => {
        const imgW = imgEle.width
        const imgH = imgEle.height
        const cvsW = canvas.width
        const cvsH = canvas.height
        const fullX = (imgW/cvsW > 1) && (imgW/imgH > cvsW/cvsH)
        const fullY = (imgH/cvsH > 1) && (imgW/imgH <= cvsW/cvsH)
        let dx = dy = 0
        if (fullX) {
            dy = cvsH/2 - (cvsW/imgW)*imgH/2
        } else if (fullY) {
            dx = cvsW/2 - (cvsH/imgH)*imgW/2
        } else {
            dx = (cvsW - imgW)/2
            dy = (cvsH - imgH)/2
        }
        context2d.drawImage(imgEle, 0, 0, imgEle.width, imgEle.height, dx, dy, cvsW-2*dx, cvsH-2*dy)
    })
})
```

### 将canvas调整为css尺寸

```
export function resizeCanvasToDisplaySize(canvas) {
  // Lookup the size the browser is displaying the canvas in CSS pixels.
  const displayWidth  = canvas.clientWidth;
  const displayHeight = canvas.clientHeight;

  // Check if the canvas is not the same size.
  const needResize = canvas.width  !== displayWidth ||
                     canvas.height !== displayHeight;

  if (needResize) {
    // Make the canvas the same size
    canvas.width  = displayWidth;
    canvas.height = displayHeight;
  }

  return needResize;
}
```

