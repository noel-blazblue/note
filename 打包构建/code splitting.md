通常的单应用程序会把所有的代码都打包到一个 JS 文件中，但如果这个文件体积太大，就会导致首屏加载变慢。
所以可以用代码切割的方式，把代码分为一个个小块，需要时再去加载，并且可以利用浏览器缓存，在下次用到的时候直接从缓存中读取。
通常，业务代码更新频率高，依赖库更新频率低，因此可以将依赖库放到浏览器缓存中，下次用到时直接读取就行了。

**主要有 2 种方式：**
-   [分离业务代码和第三方库](https://link.zhihu.com/?target=https%3A//webpack.js.org/guides/code-splitting/%23resource-splitting-for-caching-and-parallel-loads)（ vendor ）
-   [按需加载](https://link.zhihu.com/?target=https%3A//webpack.js.org/guides/code-splitting/%23resource-splitting-for-caching-and-parallel-loads)（利用 [import()](https://link.zhihu.com/?target=https%3A//github.com/tc39/proposal-dynamic-import) 语法）


### 分离依赖库
把所有 node\_modules 目录下的所有 .js 都自动分离到 vendor.js ，则需要用到 minChunks：
```js
entry: {
  // vendor: ['vue', 'axios'] // 删掉!
},

new webpack.optimize.CommonsChunkPlugin({
  name: 'vendor',
  minChunks: ({ resource }) => (
    resource &&
    resource.indexOf('node_modules') >= 0 &&
    resource.match(/\.js$/)
  ),
}),
```

![](https://pic2.zhimg.com/80/v2-1c4663763de6ffd4bf9fd1f15857f6b9_1440w.png)


### 按需加载
只加载访问到的路由的页面，其他的页面先不加载。

**业务代码：**
```js
// router.js

const Emoji = () => import(
  /* webpackChunkName: "Emoji" */
  './pages/Emoji.vue')

const Photos = () => import(
  /* webpackChunkName: "Photos" */
  './pages/Photos.vue')
```
**webpack 配置：**
```js
output: {
  chunkFilename: '[name].chunk.js',
}
```
**Babel配置：**
需要装上这个插件：[babel plugin syntax dynamic import](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/babel-plugin-syntax-dynamic-import) 来解析 import() 语法。修改 .babelrc ：
```js
{
  "plugins": ["syntax-dynamic-import"]
}
```
![preview](https://pic2.zhimg.com/v2-1bf6253be84dc5f976277105a414f091_r.jpg)

但是由于每个单独的 chunk 都需要用到某个依赖库的话，那么他们各自都会加载一遍这个库。

### 抽离 async 复用的代码库
可以把所有 async 加载的 chunk 复用到的依赖库抽离出来。
```js
// webpack.config.js

new webpack.optimize.CommonsChunkPlugin({
  async: 'common-in-lazy',
  minChunks: ({ resource } = {}) => (
    resource &&
    resource.includes('node_modules') &&
    /axios/.test(resource)
  ),
}),
```

-   所有的 async chunk ，就是 import() 产生的 chunk ，也就是 Emoji.chunk.js 和 Photos.chunk.js
-   Emoji.chunk.js 和 Photos.chunk.js 都包含了 axios ，所以把他移动到名叫 common-in-lazy 的 chunk 中

![preview](https://pic3.zhimg.com/v2-995ed2d93926e62044e54dbb38320cf6_r.jpg)

### 抽离通用模块
在所有的 async chunk ( Emoji.chunk.js 和 Photos.chunk.js ) 中找到引用 2 次以上的模块
```js
new webpack.optimize.CommonsChunkPlugin({
  async: 'used-twice',
  minChunks: (module, count) => (
    count >= 2
  ),
})
```

![](https://pic3.zhimg.com/80/v2-8223cc7dd6ec555582036ebd1771ec62_1440w.png)