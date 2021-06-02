chunk是webpack运行时的概念，bundle是webpack的执行结果，即生成了多少个文件。

## chunk和entry

一个entry对应一个chunk。chunk属于一个webpack构建过程中的概念。一个entry对应一个依赖树，这个依赖树所有依赖的集合就是一个chunk。

原始条件下，每个chunk对应生成一个bundle。

如果多个entry之间存在包含关系，则可能一个bundle打包多个chunk。
```
entry: {
    index: './src/index.js',
    add: './src/add.js'
},
```
 index引用了add.js。最终的打包结果，index.js中包含了chunk 0，即出现了重复打包。
```
Hash: 8471c9024ea2740e855f
Version: webpack 4.46.0
Time: 3338ms
Built at: 2021-06-02 11:59:15 AM
   Asset       Size  Chunks             Chunk Names
  add.js  987 bytes       0  [emitted]  add
index.js   72.5 KiB    1, 0  [emitted]  index
Entrypoint index = index.js
Entrypoint add = add.js
[0] ./src/add.js 49 bytes {0} {1} [built]
[1] ./node_modules/lodash/lodash.js 531 KiB {1} [built]
[2] ./src/index.js 89 bytes {1} [built]
[3] (webpack)/buildin/global.js 472 bytes {1} [built]
[4] (webpack)/buildin/module.js 497 bytes {1} [built]
```

## sourcemap和chunk

soucemap选项如果不含inline，则会针对每个bundle生成生成一个map文件。

同名js文件和map文件同属于一个chunk。

## runtimechunk
它的作用是将包含chunks 映射关系的 list单独从 app.js里提取出来，因为每一个 chunk 的 id 基本都是基于内容 hash 出来的。

想一下这个场景，app.js->about.js。about因为某种原因（如按需加载）被打包为单独的bundle。每次about变化，就意味着about的hash变化，app.js中存在对about.js的引用路径，进而导致app.js也变化。

单独抽离 runtimeChunk 之后，每次打包都会生成一个runtimeChunk.xxx.js，其实这个文件非常的小，gzip 之后一般只有几 kb，但这个文件又经常会改变，我们每次都需要重新请求它，它的 http 耗时远大于它的执行时间了，所以建议不要将它单独拆包，而是将它内联到我们的 index.html 之中(index.html 本来每次打包都会变)。可以使用 inline-manifest-webpack-plugin或者 assets-webpack-plugin等来实现内联的效果。

runtimeChunk值为true或者'multiple'时会为每个entry生成1个文件。值为false时，所有的entry bundle共用一个chunk。

生成的runtime chunk需要插入index.html。

参考：[webpack中的runtimeChunk](https://daihaoxin.github.io/post/97178d24.html)

## spiltChunks

该选项可以配置chunk之间的公共模块独立生成chunk，或者满足某些正则的模块独立出一个chunk。因此，该选项影响最终的bundle和chunk数目。
```
const path = require('path')
module.exports = {
  mode: 'production',
  devtool: 'none',
  entry: {
    index: './src/index.js',
    add: './src/add.js'
  },
  output: {
    filename: '[name].[contenthash:8].js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    runtimeChunk: true,
    minimize: false,
    splitChunks: {
      cacheGroups: {
        common: {
          chunks: 'initial',
          minChunks: 2,
          minSize: 0,
          name: 'common'
        },
        vendor: {
          test: /node_modules/,
          chunks: 'initial',
          name: 'vendor',
          enforce: true
        }
      }
    }
  }
}
```

以上配置生成结果如下：
```
                    Asset       Size  Chunks                                Chunk Names
          add.559b3164.js   81 bytes       1  [emitted] [immutable]         add
       common.a3657c09.js  326 bytes       0  [emitted] [immutable]         common
        index.56507b59.js  739 bytes       2  [emitted] [immutable]         index
  runtime~add.a5423113.js   6.11 KiB       3  [emitted] [immutable]         runtime~add
runtime~index.626b59fe.js   6.11 KiB       4  [emitted] [immutable]         runtime~index
       vendor.4939a6bb.js    533 KiB       5  [emitted] [immutable]  [big]  vendor
Entrypoint index [big] = runtime~index.626b59fe.js common.a3657c09.js vendor.4939a6bb.js index.56507b59.js
Entrypoint add = runtime~add.a5423113.js common.a3657c09.js add.559b3164.js
[0] ./node_modules/lodash/lodash.js 531 KiB {5} [built]
[1] ./src/add.js 49 bytes {0} [built]
[2] ./src/index.js 147 bytes {2} [built]
[3] (webpack)/buildin/global.js 472 bytes {5} [built]
[4] (webpack)/buildin/module.js 497 bytes {5} [built]
```

* add.559b3164.js和index.56507b59.js是由于entry生成的。
* common.a3657c09.js和vendor.4939a6bb.js是由于splitChunks生成的。
* runtime~add.a5423113.js和runtime~index.626b59fe.js是由于runtimeChunk: true生成的。
