## Node模块机制

### require的模块加载机制



- 计算模块路径
- 如果模块在缓存里面，取出缓存
- 如果是内置模块，则调用内置模块的require
- 生成模块实例，并添加到缓存
- 调用 module.load 方法加载模块
- 输出 module.exports 



```js
// require 其实内部调用 Module._load 方法
Module._load = function(request, parent, isMain) {
  //  计算绝对路径
  var filename = Module._resolveFilename(request, parent);

  //  第一步：如果有缓存，取出缓存
  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;

  // 第二步：是否为内置模块
  if (NativeModule.exists(filename)) {
    return NativeModule.require(filename);
  }
  
  /********************************这里注意了**************************/
  // 第三步：生成模块实例，存入缓存
  // 这里的Module就是我们上面的1.1定义的Module
  var module = new Module(filename, parent);
  Module._cache[filename] = module;

  /********************************这里注意了**************************/
  // 第四步：加载模块
  // 下面的module.load实际上是Module原型上有一个方法叫Module.prototype.load
  try {
    module.load(filename);
    hadException = false;
  } finally {
    if (hadException) {
      delete Module._cache[filename];
    }
  }

  // 第五步：输出模块的exports属性
  return module.exports;
};
```



### exports.xxx=xxx和module.exports={}的区别

exports其实就是module.exports



```js
Node会把整个待加载的hello.js文件放入一个包装函数load中执行。在执行这个load()函数前，Node准备好了module变量：

var module = {
    id: 'hello',
    exports: {}
};
load()函数最终返回module.exports：

var load = function (exports, module) {
    // hello.js的文件内容
    ...
    // load函数返回:
    return module.exports;
};

```



## Node的异步I/O

![img](https://user-gold-cdn.xitu.io/2019/7/19/16c091766912eef7?imageslim)



## https



### https用哪些端口进行通信

- 443端口用来验证服务器端和客户端的身份，比如验证证书的合法性
- 80端口用来传输数据（在验证身份合法的情况下，用来数据传输）



## 进程通信

### node的多进程架构

