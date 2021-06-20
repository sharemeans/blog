---
title: html property & attribute
categories: html
tags: [html]
date: 2018-7-3
--- 
## property

DOM元素的属性，如selectedIndex, tagName, nodeName, nodeType, ownerDocument, defaultChecked, defaultSelected

DOM是提供给js访问页面元素的对象

## attribute

HTML标签上绑定的属性，包括id, value, name, type，除了value之外，其它属性基本都是字符串类型的

### JQuery的attr()和prop()方法的区别

#### prop()就是获取property属性的值，attr()是获取attribute的值

#### 如果要改变DOM的值，获取会发生变化的属性值，需要调用prop()方法，比如checked, selected, or disabled


```
<input type="checkbox" checked="checked" />
```

对于以上标签，获取值对应关系如下：

```
elem.checked	true (Boolean) Will change with checkbox state
$( elem ).prop( "checked" )	true (Boolean) Will change with checkbox state
elem.getAttribute( "checked" )	"checked" (String) Initial state of the checkbox; does not change
$( elem ).attr( "checked" ) (1.6)	"checked" (String) Initial state of the checkbox; does not change
$( elem ).attr( "checked" ) (1.6.1+)	"checked" (String) Will change with checkbox state
$( elem ).attr( "checked" ) (pre-1.6)	true (Boolean) Changed with checkbox state
```
