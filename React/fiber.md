#  Fiber

 Fiber 也称[协程](https://www.liaoxuefeng.com/wiki/897692888725344/923057403198272) 

协程和线程并不一样，协程本身是没有并发或者并行能力的（需要配合线程），它只是一种控制流程的让出机制

 React Fiber 的思想和协程的概念是契合的: **🔴React 渲染的过程可以被中断，可以将控制权交回浏览器，让位给高优先级的任务，浏览器空闲后再恢复渲染**。 



## 调度

 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecc37fdd60d7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 



###  **requestIdleCallback** 

 *确定一个合理的运行时长，然后在合适的检查点检测是否超时(比如每执行一个小任务)，如果超时就停止执行，将控制权交换给浏览器*。 

 为了让视图流畅地运行，可以按照人类能感知到最低限度每秒60帧的频率划分时间片，这样每个时间片就是 16ms。 

1帧 = 16ms  (1000ms / 60) 

```
window.requestIdleCallback(
  callback: (dealine: IdleDeadline) => void,
  option?: {timeout: number}
  )
```



`IdleDeadline`的接口如下：

```
interface IdleDealine {
  didTimeout: boolean // 表示任务执行是否超过约定时间
  timeRemaining(): DOMHighResTimeStamp // 任务可供执行的剩余时间
}
```



 **浏览器在一帧内可能会做执行下列任务，而且它们的执行顺序基本是固定的:** 

- 处理用户输入事件
- Javascript执行
- requestAnimation 调用
- 布局 Layout
- 绘制 Paint



 如果浏览器处理完上述的任务(布局和绘制之后)，还有盈余时间，浏览器就会调用 `requestIdleCallback` 的回调 

 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecc43c710e16?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 



**但是在浏览器繁忙的时候，可能不会有盈余时间，这时候`requestIdleCallback`回调可能就不会被执行。 为了避免饿死，可以通过requestIdleCallback的第二个参数指定一个超时时间**。

> 另外不建议在`requestIdleCallback`中进行`DOM`操作，因为这可能导致样式重新计算或重新布局(比如操作DOM后马上调用 `getBoundingClientRect`)，这些时间很难预估的，很有可能导致回调执行超时，从而掉帧。



目前 `requestIdleCallback` 目前只有Chrome支持。所以目前 React [自己实现了一个](https://github.com/facebook/react/blob/master/packages/scheduler/src/forks/SchedulerHostConfig.default.js)。它利用[`MessageChannel`](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageChannel) 模拟将回调延迟到'绘制操作'之后执行:



### `MessageChannel`

```js
const el = document.getElementById('root')
const btn = document.getElementById('btn')
const ch = new MessageChannel()
let pendingCallback
let startTime
let timeout

ch.port2.onmessage = function work()  {
  // 在绘制之后被执行
  if (pendingCallback) {
    const now = performance.now()
    // 通过now - startTime可以计算出requestAnimationFrame到绘制结束的执行时间
    // 通过这些数据来计算剩余时间
    // 另外还要处理超时(timeout)，避免任务被饿死
    // ...
    if (hasRemain && noTimeout) {
      pendingCallback(deadline)
    }
  }
}

// ...

function simpleRequestIdleCallback(callback, timeout) {
  requestAnimationFrame(function animation() {
    // 在绘制之前被执行
    // 记录开始时间
    startTime = performance.now()
    timeout = timeout
    dosomething()
    // 调度回调到绘制结束后执行
    pendingCallback = callback
    ch.port1.postMessage('hello')
  })
}

```



####  **任务优先级** 

 为了避免任务被饿死，可以设置一个超时时间. **这个超时时间不是死的，低优先级的可以慢慢等待, 高优先级的任务应该率先被执行**. 目前 React 预定义了 5 个优先级 ：

- `Immediate`(-1) - 这个优先级的任务会同步执行, 或者说要马上执行且不能中断
- `UserBlocking`(250ms) 这些任务一般是用户交互的结果, 需要即时得到反馈
- `Normal` (5s) 应对哪些不需要立即感受到的任务，例如网络请求
- `Low` (10s) 这些任务可以放后，但是最终应该得到执行. 例如分析通知
- `Idle` (没有超时时间) 一些没有必要做的任务 (e.g. 比如隐藏的内容), 可能会被饿死



#### 工作单元

每一个fiber都是一个 **工作单元** 



 假设用户调用 `setState` 更新组件, 这个待更新的任务会先放入队列中, 然后通过 `requestIdleCallback` 请求浏览器调度： 

```js
updateQueue.push(updateTask);
requestIdleCallback(performWork, {timeout});
```



 现在浏览器有空闲或者超时了就会调用`performWork`来执行任务： 

```js
// 1️⃣ performWork 会拿到一个Deadline，表示剩余时间
function performWork(deadline) {

  // 2️⃣ 循环取出updateQueue中的任务
  while (updateQueue.length > 0 && deadline.timeRemaining() > ENOUGH_TIME) {
    workLoop(deadline);
  }

  // 3️⃣ 如果在本次执行中，未能将所有任务执行完毕，那就再请求浏览器调度
  if (updateQueue.length > 0) {
    requestIdleCallback(performWork);
  }
}
```



 **`workLoop`  它会从更新队列(updateQueue)中弹出更新任务来执行，每执行完一个‘`执行单元`‘，就检查一下剩余时间是否充足，如果充足就进行执行下一个`执行单元`，反之则停止执行，保存现场，等下一次有执行权时恢复**: 

```js
// 保存当前的处理现场
let nextUnitOfWork: Fiber | undefined // 保存下一个需要处理的工作单元
let topWork: Fiber | undefined        // 保存第一个工作单元

function workLoop(deadline: IdleDeadline) {
  // updateQueue中获取下一个或者恢复上一次中断的执行单元
  if (nextUnitOfWork == null) {
    nextUnitOfWork = topWork = getNextUnitOfWork();
  }

  // 🔴 每执行完一个执行单元，检查一次剩余时间
  // 如果被中断，下一次执行还是从 nextUnitOfWork 开始处理
  while (nextUnitOfWork && deadline.timeRemaining() > ENOUGH_TIME) {
    // 下文我们再看performUnitOfWork
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork, topWork);
  }

  // 提交工作，下文会介绍
  if (pendingCommit) {
    commitAllWork(pendingCommit);
  }
}
```



 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deed1711f281b3?imageslim) 



## fiber架构

### 模仿函数调用栈

React Fiber 也被称为虚拟栈帧(Virtual Stack Frame), 你可以拿它和函数调用栈类比一下, 两者结构非常像:

|          | 函数调用栈   | Fiber            |
| -------- | ------------ | ---------------- |
| 基本单位 | 函数         | Virtual DOM 节点 |
| 输入     | 函数参数     | Props            |
| 本地状态 | 本地变量     | State            |
| 输出     | 函数返回值   | React Element    |
| 下级     | 嵌套函数调用 | 子节点(child)    |
| 上级引用 | 返回地址     | 父节点(return)   |



 它其实就是一个深度优先的遍历： 

```js
/**
 * @params fiber 当前需要处理的节点
 * @params topWork 本次更新的根节点
 */
function performUnitOfWork(fiber: Fiber, topWork: Fiber) {
  // 对该节点进行处理
  beginWork(fiber);

  // 如果存在子节点，那么下一个待处理的就是子节点
  if (fiber.child) {
    return fiber.child;
  }

  // 没有子节点了，上溯查找兄弟节点
  let temp = fiber;
  while (temp) {
    completeWork(temp);

    // 到顶层节点了, 退出
    if (temp === topWork) {
      break
    }

    // 找到，下一个要处理的就是兄弟节点
    if (temp.sibling) {
      return temp.sibling;
    }

    // 没有, 继续上溯
    temp = temp.return;
  }
}

```

 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecca7850a24d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 







### 数据结构

每个fiber节点：

```js
interface Fiber {
  /**
   * ⚛️ 节点的类型信息
   */
  // 标记 Fiber 类型, 例如函数组件、类组件、宿主组件
  tag: WorkTag,
  // 节点元素类型, 是具体的类组件、函数组件、宿主组件(字符串)
  type: any,

  /**
   * ⚛️ 结构信息
   */ 
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,
  // 子节点的唯一键, 即我们渲染列表传入的key属性
  key: null | string,

  /**
   * ⚛️ 节点的状态
   */
  // 节点实例(状态)：
  //        对于宿主组件，这里保存宿主组件的实例, 例如DOM节点。
  //        对于类组件来说，这里保存类组件的实例
  //        对于函数组件说，这里为空，因为函数组件没有实例
  stateNode: any,
  // 新的、待处理的props
  pendingProps: any,
  // 上一次渲染的props
  memoizedProps: any, // The props used to create the output.
  // 上一次渲染的组件状态
  memoizedState: any,


  /**
   * ⚛️ 副作用
   */
  // 当前节点的副作用类型，例如节点更新、删除、移动
  effectTag: SideEffectTag,
  // 和节点关系一样，React 同样使用链表来将所有有副作用的Fiber连接起来
  nextEffect: Fiber | null,

  /**
   * ⚛️ 替身
   * 指向旧树中的节点
   */
  alternate: Fiber | null,
}
```



Fiber 包含的属性可以划分为 5 个部分:

- **🆕 结构信息** - 这个上文我们已经见过了，Fiber 使用链表的形式来表示节点在树中的定位
- **节点类型信息** - 这个也容易理解，tag表示节点的分类、type 保存具体的类型值，如div、MyComp
- **节点的状态** - 节点的组件实例、props、state等，它们将影响组件的输出
- **🆕 副作用** - 这个也是新东西. 在 Reconciliation 过程中发现的'副作用'(变更需求)就保存在节点的`effectTag` 中(想象为打上一个标记). 那么怎么将本次渲染的所有节点副作用都收集起来呢？ 这里也使用了链表结构，在遍历过程中React会将所有有‘副作用’的节点都通过`nextEffect`连接起来
- **🆕 替身** - React 在 Reconciliation 过程中会构建一颗新的树(官方称为workInProgress tree，**WIP树**)，可以认为是一颗表示当前工作进度的树。还有一颗表示已渲染界面的**旧树**，React就是一边和旧树比对，一边构建WIP树的。 alternate 指向旧树的同等节点。



###  Diff （Reconcilation）

 `beginWork` 是如何对 Fiber 进行比对的: 

```js
function beginWork(fiber: Fiber): Fiber | undefined {
  if (fiber.tag === WorkTag.HostComponent) {
    // 宿主节点diff
    diffHostComponent(fiber)
  } else if (fiber.tag === WorkTag.ClassComponent) {
    // 类组件节点diff
    diffClassComponent(fiber)
  } else if (fiber.tag === WorkTag.FunctionComponent) {
    // 函数组件节点diff
    diffFunctionalComponent(fiber)
  } else {
    // ... 其他类型节点，省略
  }
}
```



 宿主节点比对: 

```js
function diffHostComponent(fiber: Fiber) {
  // 新增节点
  if (fiber.stateNode == null) {
    fiber.stateNode = createHostComponent(fiber)
  } else {
    updateHostComponent(fiber)
  }

  const newChildren = fiber.pendingProps.children;

  // 比对子节点
  diffChildren(fiber, newChildren);
}
```



 类组件节点对比:

```js
function diffClassComponent(fiber: Fiber) {
  // 创建组件实例
  if (fiber.stateNode == null) {
    fiber.stateNode = createInstance(fiber);
  }

  if (fiber.hasMounted) {
    // 调用更新前生命周期钩子
    applybeforeUpdateHooks(fiber)
  } else {
    // 调用挂载前生命周期钩子
    applybeforeMountHooks(fiber)
  }

  // 渲染新节点
  const newChildren = fiber.stateNode.render();
  // 比对子节点
  diffChildren(fiber, newChildren);

  fiber.memoizedState = fiber.stateNode.state
}
```



 子节点比对: 

```js
function diffChildren(fiber: Fiber, newChildren: React.ReactNode) {
  let oldFiber = fiber.alternate ? fiber.alternate.child : null;
  // 全新节点，直接挂载
  if (oldFiber == null) {
    mountChildFibers(fiber, newChildren)
    return
  }

  let index = 0;
  let newFiber = null;
  // 新子节点
  const elements = extraElements(newChildren)

  // 比对子元素
  while (index < elements.length || oldFiber != null) {
    const prevFiber = newFiber;
    const element = elements[index]
    const sameType = isSameType(element, oldFiber)
    if (sameType) {
      newFiber = cloneFiber(oldFiber, element)
      // 更新关系
      newFiber.alternate = oldFiber
      // 打上Tag
      newFiber.effectTag = UPDATE
      newFiber.return = fiber
    }

    // 新节点
    if (element && !sameType) {
      newFiber = createFiber(element)
      newFiber.effectTag = PLACEMENT
      newFiber.return = fiber
    }

    // 删除旧节点
    if (oldFiber && !sameType) {
      oldFiber.effectTag = DELETION;
      oldFiber.nextEffect = fiber.nextEffect
      fiber.nextEffect = oldFiber
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index == 0) {
      fiber.child = newFiber;
    } else if (prevFiber && element) {
      prevFiber.sibling = newFiber;
    }

    index++
  }
}
```



 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecce3162b355?imageslim) 

 对于需要变更的节点，都打上了'标签'。 在提交阶段，React 就会将这些打上标签的节点应用变更。 



### 副作用的收集和提交

 接下来就是将所有打了 Effect 标记的节点串联起来，这个可以在`completeWork`中做, 例如: 

```js
function completeWork(fiber) {
  const parent = fiber.return

  // 到达顶端
  if (parent == null || fiber === topWork) {
    pendingCommit = fiber
    return
  }

  if (fiber.effectTag != null) {
    if (parent.nextEffect) {
      parent.nextEffect.nextEffect = fiber
    } else {
      parent.nextEffect = fiber
    }
  } else if (fiber.nextEffect) {
    parent.nextEffect = fiber.nextEffect
  }
}
```



 最后了，将所有副作用提交: 

```js
function commitAllWork(fiber) {
  let next = fiber
  while(next) {
    if (fiber.effectTag) {
      // 提交，偷一下懒，这里就不展开了
      commitWork(fiber)
    }
    next = fiber.nextEffect
  }

  // 清理现场
  pendingCommit = nextUnitOfWork = topWork = null
}
```





### 阶段划分

 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecd830671a70?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

 每次渲染有两个阶段：`Reconciliation`(协调阶段) 和 `Commit`(提交阶段). 

- **⚛️ 协调阶段**: 可以认为是 Diff 阶段, **这个阶段可以被中断**, 这个阶段会找出所有节点变更，例如节点新增、删除、属性变更等等, 这些变更React 称之为'`副作用`(Effect)' . 以下生命周期钩子会在协调阶段被调用：
  - `constructor`
  - `componentWillMount`  废弃
  - `componentWillReceiveProps` 废弃
  - `static getDerivedStateFromProps` 
  - `shouldComponentUpdate`
  - `componentWillUpdate` 废弃
  - `render`
  - `getSnapshotBeforeUpdate()`
- **⚛️ 提交阶段**: 将上一个阶段计算出来的需要处理的**副作用(Effects)**一次性执行了。**这个阶段必须同步执行，不能被打断**. 这些生命周期钩子在提交阶段被执行:
  - `componentDidMount`
  - `componentDidUpdate`
  - `componentWillUnmount`



 协调阶段可能被中断、恢复，甚至重做，**⚠️React 协调阶段的生命周期钩子可能会被调用多次!**, 例如 `componentWillMount` 可能会被调用两次。 



### 双缓冲

 `WIP 树`构建这种技术类似于图形化领域的'**双缓存(Double Buffering)**'技术, 图形绘制引擎一般会使用双缓冲技术，先将图片绘制到一个缓冲区，再一次性传递给屏幕进行显示，这样可以防止屏幕抖动，优化渲染性能。 

放到React 中，WIP树就是一个缓冲，它在Reconciliation 完毕后一次性提交给浏览器进行渲染。它可以减少内存分配和垃圾回收，WIP 的节点不完全是新的，比如某颗子树不需要变动，React会克隆复用旧树中的子树。

双缓存技术还有另外一个重要的场景就是异常的处理，比如当一个节点抛出异常，仍然可以继续沿用旧树的节点，避免整棵树挂掉。



