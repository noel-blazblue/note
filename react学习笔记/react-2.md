## 	render流程：无root



![img](https://user-gold-cdn.xitu.io/2019/5/4/16a8390a2df817dd?imageslim)

![img](https://user-gold-cdn.xitu.io/2019/5/4/16a83644a7e10749?imageslim)

单链表树结构



### render

![img](https://user-gold-cdn.xitu.io/2019/5/4/16a8258bb5c15a6a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`forceHydrate` 为 `false`。这个参数为 `true` 时表明了是服务端渲染



### legacyRenderSubtreeIntoContainer

创建react节点

![img](https://user-gold-cdn.xitu.io/2019/5/4/16a825eff0c107f9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如果没有react-root，也就是第一次渲染，就会创建一个节点，并且挂载到container._reactRootContainer上



### legacyCreateRootFromDOMContainer

![img](https://user-gold-cdn.xitu.io/2019/5/4/16a834011889cfa9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

删除container中所有节点，返回一个react节点



### ReactRoot

![img](https://user-gold-cdn.xitu.io/2019/5/4/16a8346a0317f90a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

把 fiberRoot 绑定到 reactRoot 的 this._internalRoot 中



### createFiberRoot

![img](https://user-gold-cdn.xitu.io/2019/5/4/16a834cb3340bf41?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

创建 FiberRoot ，创建 RootFiber ，并建立互相引用的关系



### **FiberNode**

```javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  this.stateNode = null;
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.effectTag = NoEffect;
  this.alternate = null;
}
```

父节点的 child 指向一个子节点，所有子节点的 return 指向父节点，sibling 指向兄弟节点

**effectTag：** 记录所有的effect，也就是对DOM的操作

**alternate：** 在一个 React 应用中，通常来说都有两个 `fiebr` 树，一个叫做 old tree，另一个叫做 workInProgress tree。前者对应着已经渲染好的 DOM 树，后者是正在执行更新中的 fiber tree，还能便于中断后恢复。两棵树的节点互相引用，便于共享一些内部的属性，减少内存的开销。毕竟前文说过每个组件或 DOM 都会对应着一个 `fiber` 对象，应用很大的话组成的 `fiber` 树也会很大，如果两棵树都是各自把一些相同的属性创建一遍的话，会损失不少的内存空间及性能。当更新结束以后，workInProgress tree 会将 old tree 替换掉，这种做法称之为 double buffering



## render流程：有root

![img](https://user-gold-cdn.xitu.io/2019/5/21/16ad90b614add058?imageslim)



### unbatchedUpdates

![img](https://user-gold-cdn.xitu.io/2019/5/20/16ad34ab771a89a0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

因为是root，所以告知内部不需要批量更新



### ReactRoot.prototype.render

![img](https://user-gold-cdn.xitu.io/2019/5/20/16ad3674981526a9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 获取 FilberRoot 
- 创建work示例，为了在组件渲染或更新后把所有传入 `ReactDom.render` 中的回调函数全部执行一遍。

-->创建 updateContainer



### updateContainer

![img](https://user-gold-cdn.xitu.io/2019/5/20/16ad3c5f7e765e27?imageslim)

- 从 current 属性中取出 fliber 对象
- 计算 currentTime
- 计算 expirationTime

-->updateContainerAtExpirationTime



#### requestCurrentTime

```javascript
function recomputeCurrentRendererTime() {
  const currentTimeMs = now() - originalStartTimeMs;
  currentRendererTime = msToExpirationTime(currentTimeMs);
}
```

now() 是现在的时间

originalStartTimeMs` 是 React 应用初始化时就会生成的一个变量，值为时间戳

那么这两个值相减以后，得到的结果也就是现在离 React 应用初始化时经过了多少时间。



### scheduleRootUpdate

![img](https://user-gold-cdn.xitu.io/2019/5/21/16ad84d47bcfb78e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

update：

```javascript
// update 对象的内部属性
expirationTime: expirationTime,
tag: UpdateState,
// setState 的第一二个参数
payload: null,
callback: null,
// 用于在队列中找到下一个节点
next: null,
nextEffect: null,
```

`next` 属性。因为 `update` 其实就是一个队列中的节点，这个属性可以用于帮助我们寻找下一个 `update`。对于批量更新来说，我们可能会创建多个 `update`，因此我们需要将这些 `update` 串联并存储起来，在必要的时候拿出来用于更新 `state`。



enqueueUpdate：

创建或者获取一个队列，然后把 `update` 对象入队。



## 调度

JS 和渲染引擎是一个互斥关系。如果 JS 在执行代码，那么渲染引擎工作就会被停止。

React 会根据任务的优先级去分配各自的 `expirationTime`，在过期时间到来之前先去处理更高优先级的任务，并且高优先级的任务还可以打断低优先级的任务（因此会造成某些生命周期函数多次被执行），从而实现在不影响用户体验的情况下去分段计算更新（也就是时间分片）。



### expriationTime

![img](https://user-gold-cdn.xitu.io/2019/6/3/16b1cd59bbe4f642?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 帮助我们对比不同任务之间的优先级以及计算任务的 timeout。



**当前时间：**performance.now() 这个 API 会返回一个精确到毫秒级别的时间戳

**常量：**根据不同优先级得出的一个数值

```javascript
var ImmediatePriority = 1;
var UserBlockingPriority = 2;
var NormalPriority = 3;
var LowPriority = 4;
var IdlePriority = 5;
```

**数值：**

```javascript
var maxSigned31BitInt = 1073741823;

// Times out immediately
var IMMEDIATE_PRIORITY_TIMEOUT = -1;
// Eventually times out
var USER_BLOCKING_PRIORITY = 250;
var NORMAL_PRIORITY_TIMEOUT = 5000;
var LOW_PRIORITY_TIMEOUT = 10000;
// Never times out
var IDLE_PRIORITY = maxSigned31BitInt;
```





### requestIdleCallback

![img](https://user-gold-cdn.xitu.io/2019/6/4/16b1e1a1f2969092?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 该函数的回调方法会在浏览器的空闲时期依次调用， 可以让我们在事件循环中执行一些任务，并且不会对像动画和用户交互这样延迟敏感的事件产生影响。



**requestAnimationFrame ：** 回调方法会在每次重绘前执行

```javascript
rAFID = requestAnimationFrame(function(timestamp) {
  // cancel the setTimeout
  localClearTimeout(rAFTimeoutID);
  callback(timestamp);
});
rAFTimeoutID = setTimeout(function() {
  // 定时 100 毫秒是算是一个最佳实践
  localCancelAnimationFrame(rAFID);
  callback(getCurrentTime());
}, 100);
```

当 `requestAnimationFrame` 不执行时，会有 `setTimeout` 去补救，两个定时器内部可以互相取消对方。



**计算时间：** 

> 在一帧当中，浏览器可能会响应用户的交互事件、执行 JS、进行渲染的一系列计算绘制。假设当前我们的浏览器支持 1 秒 60 帧，那么也就是说一帧的时间为 16.6 毫秒。如果以上这些操作超过了 16.6 毫秒，那么就会导致渲染没有完成并出现掉帧的情况，继而影响用户体验；如果以上这些操作没有耗时 16.6 毫秒的话，那么我们就认为当下存在空闲时间让我们可以去执行任务。

```javascript
//计算出当前帧是否还有剩余时间让我们使用。
let frameDeadline = 0
let previousFrameTime = 33
let activeFrameTime = 33
let nextFrameTime = performance.now() - frameDeadline + activeFrameTime
if (
  nextFrameTime < activeFrameTime &&
  previousFrameTime < activeFrameTime
) {
  if (nextFrameTime < 8) {
    nextFrameTime = 8;
  }
  activeFrameTime =
    nextFrameTime < previousFrameTime ? previousFrameTime : nextFrameTime;
} else {
  previousFrameTime = nextFrameTime;
}
```



**MessageChannel：** 宏任务特性，在渲染以后才去执行任务。



`requestAnimationFrame` + 计算帧时间及下一帧时间 + `MessageChannel` 就是我们实现 `requestIdleCallback` 的三个关键点了。



### 调度的流程

- 首先每个任务都会有各自的优先级，通过当前时间加上优先级所对应的常量我们可以计算出 `expriationTime`，**高优先级的任务会打断低优先级任务**

- 在调度之前，判断当前任务**是否过期**，过期的话无须调度，直接调用 `port.postMessage(undefined)`，这样就能在渲染后马上执行过期任务了

- 如果任务没有过期，就通过 `requestAnimationFrame` 启动定时器，在重绘前调用回调方法

- 在回调方法中我们首先需要**计算每一帧的时间以及下一帧的时间**，然后执行 `port.postMessage(undefined)`

- `channel.port1.onmessage` 会在渲染后被调用，在这个过程中我们首先需要去判断**当前时间是否小于下一帧时间**。如果小于的话就代表我们尚有空余时间去执行任务；如果大于的话就代表当前帧已经没有空闲时间了，这时候我们需要去判断是否有任务过期，**过期的话不管三七二十一还是得去执行这个任务**。如果没有过期的话，那就只能把这个任务丢到下一帧看能不能执行了

