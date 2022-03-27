## babel 组成部分
### transpiler
用于重写代码的 transpiler 程序。开发者在自己的电脑上运行它。它以之前的语言标准对代码进行重写。然后将代码传到面向用户的网站。像 [webpack](http://webpack.github.io/) 这样的现代项目构建系统，提供了在每次代码改变时自动运行 transpiler 的方法，因此很容易集成在开发过程中。

### polyfill
新的语言特性可能不仅包括语法结构，还包括新的内建函数。 Transpiler 会重写代码，将语法结构转换为旧的结构。但是对于新的内建函数，需要我们去实现。JavaScript 是一个高度动态化的语言。脚本可以添加/修改任何函数，从而使它们的行为符合现代标准。

更新/添加新函数的脚本称为 “polyfill”。它“填补”了缺口，并添加了缺少的实现。

两个有意思的 polyfills 是：

-   [core js](https://github.com/zloirock/core-js) 支持很多，允许只包含需要的功能。
-   [polyfill.io](http://polyfill.io/) 根据功能和用户的浏览器，为脚本提供 polyfill 的服务。

## babel 原理
-   解析：将代码转换成 AST
    -   词法分析：将代码(字符串)分割为token流，即语法单元成的数组
    -   语法分析：分析token流(上面生成的数组)并生成 AST
-   转换：访问 AST 的节点进行变换操作生产新的 AST
    -   [Taro](https://github.com/NervJS/taro/blob/master/packages/taro-transformer-wx/src/index.ts#L15)就是利用 babel 完成的小程序语法转换
-   生成：以新的 AST 为基础生成代码


## presets

Babel 的预设（preset）可以被看作是一组 Babel 插件和/或 [`options`](https://www.babeljs.cn/docs/options) 配置的可共享模块。

我们已经针对常用环境编写了一些预设（preset）：

-   [@babel/preset-env](https://www.babeljs.cn/docs/babel-preset-env) for compiling ES2015+ syntax
-   [@babel/preset-typescript](https://www.babeljs.cn/docs/babel-preset-typescript) for [TypeScript](https://www.typescriptlang.org/)
-   [@babel/preset-react](https://www.babeljs.cn/docs/babel-preset-react) for [React](https://reactjs.org/)
-   [@babel/preset-flow](https://www.babeljs.cn/docs/babel-preset-flow) for [Flow](https://flow.org/)

## 集成
### @babel/polyfill
Babel includes a [polyfill](https://en.wikipedia.org/wiki/Polyfill_(programming)) that includes a custom [regenerator runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js) and [core-js](https://github.com/zloirock/core-js).

This will emulate a full ES2015+ environment (no < Stage 4 proposals) and is intended to be used in an application rather than a library/tool. (this polyfill is automatically loaded when using `babel-node`).

This means you can use new built-ins like `Promise` or `WeakMap`, static methods like `Array.from` or `Object.assign`, instance methods like `Array.prototype.includes`, and generator functions (provided you use the [regenerator](https://www.babeljs.cn/docs/babel-plugin-transform-regenerator) plugin). The polyfill adds to the global scope as well as native prototypes like `String` in order to do this.

## 工具
### @babel/types
This module contains methods for building ASTs manually and for checking the types of AST nodes.

### @babel/generator
Turns an AST into code.