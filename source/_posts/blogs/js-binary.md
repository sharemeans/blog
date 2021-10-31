---
title: ArrayBuffer,TypeArray,Blob,File,atob
categories: javascript
tags: [blob, file]
date: 2021-10-8
---

> 随着前端工具库发展日益成熟，文件处理相关工具库都封装的比较完美，平时自然比较少接触`ArrayBuffer` `TypeArray` `Blob` `File` `ReadableStream` `atob`等二进制相关的类与方法，对它们总是一知半解。趁假期有整块的时间好好看了看文档，顺便做个笔记帮助理解。

## Blob
用来存储数据，通常是图片音视频或者普通文件数据。构造函数可传入ArrayBuffer、ArrayBufferView、Blob、String（以UTF8编码）类型的数据，并转化为Blob对象。File继承于Blob。

##### api
- slice： 截取子集并创建新的Blob对象
- stream： 返回1个ReadableStream对象
- text： 返回对象的文本格式
- arrayBuffer：返回对象的二进制格式

##### 应用场景
######  分片上传（看参考资料1）
使用slice对大文件分片上传。

```
const file = new File(["a".repeat(1000000)], "test.txt");

const chunkSize = 40000;
const url = "https://httpbin.org/post";

async function chunkedUpload() {
  for (let start = 0; start < file.size; start += chunkSize) {
      const chunk = file.slice(start, start + chunkSize + 1);
      const fd = new FormData();
      fd.append("data", chunk);

      await fetch(url, { method: "post", body: fd }).then((res) =>
        res.text()
      );
  }
}
```

###### 下载文件
XHRHttpRequest通过指定xhr.responseType = 'blob'，可以使得xhr的response属性为Blob类型，需要注意的是，response header的content-type是服务端返回的格式，xhr.responseType定义了请求返回后浏览器的行为，无法改变response header的content-type值。

fetch方法回调[Response.blob()](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)获取blob对象。

######  用作a/img标签的url
URL.createObjectURL(blob)生成如下格式的链接：

```
// blob:<origin>/<uuid>
blob:http://127.0.0.1:5555/c1a6586b-b9e6-4f82-a56a-992e05ce20e1
```
uuid是浏览器生成的URL->blob的映射id。文件很大时，blob常驻内存会导致内存泄漏。blob url不再使用时，及时调用URL.revokeObjectURL(url)清除blob。

###### 与base64（Data URL）互相转化

格式为`data:[<mediatype>][;base64],<data>`，通常用于图片预览。

- blob转化为Data URL

初始化FileReader实例，调用readAsDataURL传入blob获取base64。

```
function blobToDataURL(blob, callback) {
  return new Promise(resolve => {
    let a = new FileReader();
    a.onload = function (e) {
      resolve(e.target.result);
    }
    a.readAsDataURL(blob);
  })
}
```
- Data URL转化为blob 

```
function dataURLtoBlob(dataurl) {
    let arr = dataurl.split(',')
    let mime = arr[0].match(/:(.*?);/)[1]
    let bstr = atob(arr[1]) // base64字符串转化为二进制数组（中文会乱码）
    let n = bstr.length
    
    let u8arr = new Uint8Array(n)
    while (n--) {
        u8arr[n] = bstr.charCodeAt(n) // ascii字符转化为ascii值
    }
    return new Blob([u8arr], { type: mime }) // 基于ascii值创建blob 对象（字符串默认会按照UTF-8编码，所以不会出现乱码）
}
```


###### 参考资料
- [你不知道的 Blob](https://juejin.cn/post/6844904178725158926)

## ArrayBuffer

固定长度的原始二进制缓冲区。表示一段内存区域，不能直接读写，因为我们不知道它的格式，需要指定格式（TypedArray视图和DataView视图）来读写。

TypedArray有9种类型：
- Int8Array：8 位有符号整数，长度 1 个字节。
- Uint8Array：8 位无符号整数，长度 1 个字节。
- Uint8ClampedArray：8 位无符号整数，长度 1 个字节，溢出处理不同。
- Int16Array：16 位有符号整数，长度 2 个字节。
- Uint16Array：16 位无符号整数，长度 2 个字节。
- Int32Array：32 位有符号整数，长度 4 个字节。
- Uint32Array：32 位无符号整数，长度 4 个字节。
- Float32Array：32 位浮点数，长度 4 个字节。
- Float64Array：64 位浮点数，长度 8 个字节。

跨视图转换时，存在1个字节序的问题。将一个多字节对象的低位放在较小的地址处，高位放在较大的地址处，则称小端序；反之则称大端序。

如`0x12345678`有4个字节，如果是小端序，则内存写入如下：

```
0x78|0x56|0x34|0x12
//---地址增序--->
```

举个视图转换的例子，电脑写入顺序为小端序：

```
const buffer = new ArrayBuffer(16);
const int32View = new Int32Array(buffer);
// int32View长度为4，元素的值分别为0，2，4，6
for (let i = 0; i < int32View.length; i++) {
  // 每次写入4个字节，这4个字节倒过来
  int32View[i] = i * 2;
}
// 最终buffer中的内容为：0x00|0x00|0x00|0x00 0x02|0x00|0x00|0x00 0x04|0x00|0x00|0x00 0x06|0x00|0x00|0x00

const int16View = new Int16Array(buffer);

// int16View长度为8，将buffer内容按照2个字节的长度分割
for (let i = 0; i < int16View.length; i++) {
  // 读取的时候2个字节为1组，按照小端序调整这2个字节的顺序后输出
  console.log("Entry " + i + ": " + int16View[i]);
}
// 输出结果为：0 0 2 0 4 0 6 0

```

##### ArrayBuffer转字符串

```javascript
const decoder = new TextDecoder(outputEncoding)
const str = decoder.decode(input) // input:ArrayBuffer | Uint8Array | Int8Array | Uint16Array | Int16Array | Uint32Array | Int32Array
```

##### 字符串转ArrayBuffer

```javascript
const encoder = new TextEncoder()
const view = encoder.encode(input) // input：String view:默认UTF-8编码，类型为Uint8Array
const buf = view.buffer
```
## atob/btoa

#### binary string
类似于ASCII子集，binary string可以表示0-255，作用是为了表示二进制的1个字节。

#### btoa
binary string to base64 ASCII，将二进制转化为ASCII字符，并进行base64编码。参数若超出1个字节则该方法报错。

#### atob
base64 ASCII to binary string，将base64转化回binary string。

binary string只能表示1个字节的字符。由于base64字符串不一定来源于btoa（比如说来源于canvas或者fileReader），可能其源字符占用2个或以上的字节数。因此，调用该方法返回的字符串可能是乱码：

- 源字符串：一文彻底掌握 Blob Web API
- base64：5LiA5paH5b275bqV5o6M5o+hIEJsb2IgV2ViIEFQSQ==
- atob(base64)：ä¸æå½»åºææ¡ Blob Web API

如果遇到这种情况，该如何把base64还原为字符串呢？
1. 把base64字符串使用atob转化为binary string
2. 获取每个字符的编码并转化为16进制，并在前面加上字符‘%’
3. decodeURIComponent解码

这个方法，利用了`encodeURIComponent`函数的原理之一：将多个字节表示的字符，按照字节拆分并使用16进制表示，并使用%分割。`atob`的方法得到的结果刚好和`encodeURIComponent`不谋而合，因此使用decodeURIComponent刚好合适。

```
function b64DecodeUnicode(str) {
    // Going backwards: from bytestream, to percent-encoding, to original string.
    return decodeURIComponent(atob(str).split('').map(function(c) {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
    }).join(''));
}
```

还有另外一种比较曲折的方法：
1. 把base64字符串使用atob转化为binary string
2. 使用前文的dataUrlToBlob方法将binary string存入Uint8Array
3. 使用TextDecoder将Uint8Array转化为UTF-8或者Unicode字符串

```
// 字符串原文为：一文彻底掌握 Blob Web API
const b64 = '5LiA5paH5b275bqV5o6M5o+hIEJsb2IgV2ViIEFQSQ=='
let bytes = window.atob(b64);
let ab = new ArrayBuffer(bytes.length);
let ia = new Uint8Array(ab);
for (let i = 0; i < bytes.length; i++) {
  ia[i] = bytes.charCodeAt(i);
}
const decoder = new TextDecoder('UTF-8')
const str = decoder.decode(ia)
console.log('str:', str)
```

## File
文件对象。inout type="file"标签files属性的元素。继承于Blob，新增`lastModified` `lastModifiedDate` `name` `webkitRelativePath`属性，相对于Blob没有任何新增方法，完全可以按照Blob的方式去处理。

#### FileReader
用于获取用户选择的文件的内容。File对象继承于Blob，虽然Blob自带方法可以读取文件内容，但是FileReader可以做到更多：
- 将原始二进制以字符串的格式输出（readAsBinaryString）
- 将文件内容以base64格式输出（readAsDataURL）
- 将文件内容以文本方式输出（readAsText）

Blob的text方法和FileReader.readAsText类似，区别如下：
- Blob.text() 返回的是一个 promise 对象，而 FileReader.readAsText() 是一个基于事件的 API
- Blob.text() 总是使用 UTF-8 进行编码，而 FileReader.readAsText() 可以使用不同编码方式，取决于 blob 的类型和一个指定的编码名称

Blob.prototype.arrayBuffer和FileReader.prototype.readAsArrayBuffer类似，除了promise和事件回调方式区别之外，返回的内容是一致的。

## Streams API
Streams API能够让我们直接处理通过网络接收的数据流或通过本地任何方式创建的数据流。

流，根据块类型可以分为，传统流（chunk为typed array）和字节流（chunk为byte）。字节流可以传入自定义的buffer来接收chunk。通过具体stream构造函数的type字段可以声明流类型。

关于这块我本人目前还没理解透彻，先mark下，将来再填坑。

#### ReadableStream
可读流。有2个来源。推流和拉流。

推流是和对方建立连接后，对方主动推送推送过来的数据，我方可以自行决定是否终端或者终止。

拉流是和对方连接后，我方主动请求数据，如通过fetch和XHR请求访问。

#### WritableStream
可写入数据的对象。通过writer写入数据。1次只能有1个write写入stream。

#### 使用场景
- 视频特效：读取视频流，然后通过管道与转换流连接起来，逐帧进行转换处理，实现诸如水印、剪辑、添加音轨等功能。
- 数据解压缩：压缩包、视频、图片解压缩等。
- 图像转码：流处理可以是基于字节的，因此可以逐个字节地处理请求到图片资源，来对它进行转码。例如JPG转PNG等格式转换。

###### 参考资料
- [深入理解JS中的Stream API](https://juejin.cn/post/6992007156320960542)