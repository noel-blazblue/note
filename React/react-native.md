## React Native 架构

![img](https://pic3.zhimg.com/80/v2-eaa08de96442aed01387827a2ec345ca_hd.jpg) 





- 绿色的是我们应用开发的部分。我们写的代码基本上都是在这一层
- 蓝色代表公用的跨平台的代码和工具引擎，一般我们不会动蓝色部分的代码
- 黄色代码平台相关的代码，做定制化的时候会添加修改代码。不跨平台，要针对平台写不同的代码。iOS写OC, android写java，web写js. 每个bridge都有对应的js文件，js部分是可以共享的，写一份就可以了。如果你想做三端融合，你就得理解这一个东西。如果你要自己定制原生控件，你就得写bridge部分
- 红色部分是系统平台的东西。红色上面有一个虚线，表示所有平台相关的东西都通过bridge隔离开来了



**JavaScriptCore + ReactJS + Bridges 就成了React Native** 



- React是一个纯JS库，所有的React代码和所有其它的js代码都需要JS Engine来解释执行。因为种种原因，浏览器里面的JS代码是不允许调用自定义的原生代码的，而React又是为浏览器JS开发的一套库，所以，比较容易理解的事实是React是一个纯JS库，它封装了一套Virtual Dom的概念，实现了数据驱动编程的模式，为复杂的Web UI实现了一种无状态管理的机制, 标准的HTML/CSS之外的事情，它无能为力。调用原生控件，驱动声卡显卡，读写磁盘文件，自定义网络库等等，这是JS/React无能为力的
- 你可以简单理解为React是一个纯JS 函数， 它接受特定格式的字符串数据，输出计算好的字符串数据
- JS Engine负责调用并解析运行这个函数

- Bridge的作用就是给RN内嵌的JS Engine提供原生接口的扩展供JS调用。所有的本地存储、图片资源访问、图形图像绘制、3D加速、网络访问、震动效果、NFC、原生控件绘制、地图、定位、通知等都是通过Bridge封装成JS接口以后注入JS Engine供JS调用。理论上，任何原生代码能实现的效果都可以通过Bridge封装成JS可以调用的组件和方法, 以JS模块的形式提供给RN使用。
- 每一个支持RN的原生功能必须同时有一个原生模块和一个JS模块，JS模块是原生模块的封装，方便Javascript调用其接口。Bridge会负责管理原生模块和对应JS模块之间的沟通, 通过Bridge, JS代码能够驱动所有原生接口，实现各种原生酷炫的效果。

`Bridge` 原生代码负责管理原生模块并生成对应的JS模块信息供JS代码调用。每个功能JS层的封装主要是针对ReactJS做适配，让原生模块的功能能够更加容易被用ReactJS调用。`MessageQueue.js`是`Bridge`在JS层的代理，所有JS2N和N2JS的调用都会经过`MessageQueue.js`来转发。JS和Native之间不存在任何指针传递，所有参数都是字符串传递。所有的instance都会被在JS和Native两边分别编号，然后做一个映射,然后那个数字/字符串编号会做为一个查找依据来定位跨界对象。 



###  层次架构

![preview](https://pic2.zhimg.com/v2-dfe825c504b3b20082222184d275a8c1_r.jpg) 

- **Java层**：该层主要提供了Android的UI渲染器`UIManager`（将JavaScript映射成`Android Widget`）以及一些其他的功能组件（例如：Fresco、Okhttp）等，在java层均封装为Module，java层核心jar包是react-native.jar，封装了众多上层的interface，如Module，Registry，bridge等
- **C++层**：主要处理Java与JavaScript的通信以及执行JavaScript代码工作，该层封装了JavaScriptCore，执行对js的解析。基于`JavaScriptCore`，`Web`开发者可以尽情使用ES6的新特性，如class、箭头操作符等，而且 React Native运行在`JavaScriptCore`中的，完全不存在浏览器兼容的情况。Bridge桥接了java ， js 通信的核心接口。JSLoader主要是将来自assets目录的或本地file加载javascriptCore，再通过`JSCExectutor`解析js文件
- **Js层**：该层提供了各种供开发者使用的组件以及一些工具库。
  `Component`：Js层通js/jsx编写的`Virtual Dom`来构建`Component`或Module，Virtual DOM是DOM在内存中的一种轻量级表达方式，可以通过不同的渲染引擎生成不同平台下的UI。component的使用在 React 里极为重要, 因为component的存在让计算 DOM diff 更高效。
  ReactReconciler : 用于管理顶层组件或子组件的挂载、卸载、重绘