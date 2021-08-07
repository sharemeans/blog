---
title: 手写deepClone
categories: js
tags: [js]
date: 2021-6-3
---

通过Object.prototype.toString判断
```
let mapObj = new Map();
mapObj['key1'] = 1;
mapObj['key2'] = 2;
let obj = {
  a: 100,
  b: [10, 20, 30],
  c: {
    x: 10
  },
  d: /^\d+$/,
  e: mapObj
};
let arr = [10, [100, 200], {
  x: 10,
  y: 20
}];

function deepClone(obj) {
  let newObj = {};
  let type = Object.prototype.toString.call(obj).match(/\[object\s(\w+)\]/)[1];

  if (type == 'Array' || type == 'Set') {
    newObj = [];

    for (const val of obj) {
      newObj.push(deepClone(val));
    }
  } else if (type == 'Set') {
    newObj = new Set()
    for (const key of obj) {
      newObj.add(deepClone(obj[key]));
    }
  } else if (type == 'Object' || type == 'Map') {
    if(type == 'Map') newObj = new Map()

    for (const key in obj) {
      if (Object.hasOwnProperty.call(obj, key)) {
        newObj[key] = deepClone(obj[key]);
      }
    }
  } else {
    return obj;
  }

  return newObj;
}

let obj2 = deepClone(obj);
let arr2 = deepClone(arr); // console.log(obj2)
console.log(arr2)
console.log(obj2)
console.log('-----------')
obj.e.key1 = 3
console.log(obj2)
console.log('-----------')
arr[1][0] = 999
console.log(arr2)
```

以上写法有个缺点是，
1. 针对所有引用类型都要做判断，考虑的越周到，代码就越长
2. 自定义类型


通过typeof判断
```
function deepClone(obj) {
  // typeOf针对null值结果也是object，需要特殊处理
  if (typeof obj != 'object') return obj
  if (obj == null) return obj
  if (obj instanceof RegExp) return new RegExp(obj)
  if (obj instanceof Date) return new Date(obj)
  
  let newObj = new obj.constructor;
  for (const key in obj) {
    if (Object.hasOwnProperty.call(obj, key)) {
      newObj[key] = deepClone(obj[key]);
    }
  }
  return newObj;
}
```

以上方法需要注意:
- typeof针对null值结果也是object，需要特殊处理
- typeof针对所有的引用类型结果都是object，需要特殊处理

偶然看到一个写的很全面的深拷贝方法：
https://javascript.plainenglish.io/write-a-better-deep-clone-function-in-javascript-d0e798e5f550