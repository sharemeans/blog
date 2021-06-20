---
title: 纯js实现打印功能
categories: javascript
tags: [javascript]
date: 2019-8-20
--- 

打印其实最终都是调用window.print方法实现打印。只不过，我们通常不是打印整个页面，所以，诞生了各种各样的方法实现部分打印。适配vue框架的打印组件有vuePlugs_printjs和vue-print-nb。这两者实现的方式是一样的，都是使用iframe插入打印区域和对应样式。区别只是使用方式的区别。

那么，我也实现了一个原生js实现的打印功能。

## 一、新建iframe窗口
为了打印出想要的部分，我们需要新建一个窗口，把需要的dom装进去。为了避免部分浏览器阻止窗口弹出导致无法打印，建议使用iframe。

打印的时候，我们需要防止iframe的显示影响页面布局，只需要在父页面设置iframe的样式为display: none即可。如下：

```javascript
const frame = document.createElement('iframe')
frame.style.display = 'none'
document.body.appendChild(frame)
```

## 二、选择需要打印的部分dom

从当前dom选择需要打印的dom元素，插入iframe。

```javascript
let target = document.getElementById('print-target')
if (target) {
  frame.contentDocument.body.innerHTML = target.outerHTML
}
```

## 三、插入样式

一般情况下，你可以直接将父页面的style和link全部拷贝到iframe中，这样打印到样式和原本样式是一样的，除非打印部分的样式受未打印部分的父元素控制导致样式脱节，所以，打印部分的样式尽量和父元素解耦。

我们知道，浏览器打印的时候你可能需要不一样的样式，只需要加多一个style标签引入打印样式即可。比较典型的就是，打印的时候你想要按照需要打印的纸张尺寸打印，这时候可以在@media print中将容器的宽度改为打印尺寸（如A4是210mm，css支持mm单位的尺寸）。

## 四、特殊元素处理

由于我们将打印区域拷贝到ifame中采用对的是将targetElement.outerHTML赋值给iframe的body的。这样赋值的情况下，部分表单组件会失去值，所以，我们需要在赋值之前将值固定，如下：

```javascript
// https://github.com/Power-kxLee/vue-print-nb/blob/master/src/packages/printarea.js
let copy = ele.cloneNode(true);
let copiedInputs = copy.querySelectorAll('input,select,textarea');
let selectCount = -1;
for (let i = 0; i < copiedInputs.length; i++) {
    let item = copiedInputs[i];
    let typeInput = item.getAttribute('type');
    let copiedInput = copiedInputs[i];
    // 获取select标签
    if (!typeInput) {
        typeInput = item.tagName === 'SELECT' ? 'select' : item.tagName === 'TEXTAREA' ? 'textarea' : '';
    }
    // 处理input框
    if (item.tagName === 'INPUT') {
        // 除了单选框 多选框比较特别
        if (typeInput === 'radio' || typeInput === 'checkbox') {
            copiedInput.setAttribute('checked', item.checked);
        // 
        } else {
            copiedInput.value = item.value;
            copiedInput.setAttribute('value', item.value);
        } 
    // 处理select
    } else if (typeInput === 'select') {
        
        selectCount ++;
        for (let b = 0; b < ele.querySelectorAll('select').length; b++) {
            let select = ele.querySelectorAll('select')[b]; // 获取原始层每一个select
            !select.getAttribute('newbs') && select.setAttribute('newbs', b) // 添加标识
            if (select.getAttribute('newbs') == selectCount) {
                let opSelectedIndex = ele.querySelectorAll('select')[selectCount].selectedIndex;
                item.options[opSelectedIndex].setAttribute('selected', true);

            }
        }
    // 处理textarea
    } else{
        copiedInput.innerHTML = item.value;
        copiedInput.setAttribute('html', item.value);
    }
}
```
