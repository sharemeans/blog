---
title: base64字符串保存
categories: js
date: 2020-5-8
--- 

```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>

</body>
<script>
//方法1
function saveAsLink(base64Str) {
  var a = document.createElement('a');   // 转换完成，创建一个a标签用于下载
  a.download = fileName;
  a.href = 'data:application/msword;base64,' + base64Str;
  a.style.display = 'none';
  document.body.appendChild(a)    // 修复firefox中无法触发click
  a.click();
  document.body.removeChild(a)
}
saveAsLink(fileBse64Str)
//方法2(firefox浏览器不兼容)
var blob = new Blob([_base64ToArrayBuffer(fileBse64Str)], {type: 'application/msword'}), fileName = '文件名称.doc'
function _base64ToArrayBuffer(base64) {
    var binary_string =  window.atob(base64);
    var len = binary_string.length;
    var bytes = new Uint8Array( len );
    for (var i = 0; i < len; i++)        {
        bytes[i] = binary_string.charCodeAt(i);
    }
    return bytes.buffer;
}
function downFile(blob, fileName) {
  if (window.navigator.msSaveOrOpenBlob) {
      navigator.msSaveBlob(blob, fileName);
  } else {
      var link = document.createElement('a');
      debugger
      link.href = window.URL.createObjectURL(blob);
      link.download = fileName;
      link.click();
      window.URL.revokeObjectURL(link.href);
  }
}
downFile(blob, fileName)
</script>
</html>
```
