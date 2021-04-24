## 前言
其实了解react的源码，并不是一件高成本的事，它的内部架构没有大家想象中的那么难，而阅读源码的收获，却是远远大过了成本，能够学习到其中优秀的设计模式和数据架构，并对业务中的熟练度有非常大的提升。因此，这是在前端进阶上非常值得去做的一件事。

**推荐的学习路线：**

从社区上一些解读文章中了解react的整体架构和运作流程，然后自己再对照源码，对其中的各个环节进行细致的研究和验证。

本文主要解释 hooks 这部分的源码，对于fiber架构和任务调度，只会说一下比较基础的部分。



通过这一篇文章，你可以懂得：
- `fiber`的基础架构
- `hooks`的基础架构
- `hooks`大部分`api`的内部实现
- 异步可中断模型基础




## Fiber 架构

### Fiber 数据结构
正如同`react`推崇的组件化一样，在源码内部，也是由一个个"组件"去组成整个架构。这个"组件"就是`fiber`，它是 `React` 中的一个基本工作单元，`React`的一切操作都要基于它去实现。

```js
interface Fiber {
  // 1. Instance 类型信息
  // 标记 Fiber 的实例的类型, 例如函数组件、类组件、宿主组件(即dom)
  tag: WorkTag,
  // class、function组件的构造函数，或者dom组件的标签名。
  type: any,
	// class、function组件的构造函数，或者dom组件的标签名。
	elementType: any,
	// DOM节点 | Class实例
	// 函数组件则为空
  stateNode: any,
	
	key: key,
	ref: ref,
	
  // 2. Fiber 结构信息
  // 指向父fiber 节点
  return: Fiber | null,
  // 指向子fiber节点
  child: Fiber | null,
  // 指向兄弟fiber节点
  sibling: Fiber | null,
	// 指向另外一颗树中对应的fiber节点
  alternate: Fiber | null,

  // 3. Fiber节点的状态
  // 本次更新的props
  pendingProps: any,
  // 上一次渲染的props
  memoizedProps: any, 
  // 如果是class组件，会保存上一次渲染的state
  // 如果是hooks组件，会保存所有hooks组成的链表
  memoizedState: any,
	// 如果是class，将保存setState产生的update链表
	// 如果是hooks，这个地方会存放effect链表
  // 如果是dom节点，会存放他所需更新的props
  updateQueue: UpdateQueue<any> | null, 

  // 4. 副作用
  // 用二进制来存储的当前节点的所需执行的操作，如节点更新、删除、移动
  flags: Flags,
  // 副作用链表，会把所有需要执行副作用的fiber串联起来
  nextEffect: Fiber | null,
  firstEffect: Fiber | null, 
  lastEffect: Fiber | null, 
	
	// 5. 调度优先级相关
  lanes = NoLanes;
  childLanes = NoLanes;

}
```



通俗易懂的说，所有的`element`都是一个独立的`fiber`，`element`的同级元素用`sibling`链接，子元素用`child`链接，这样就由上至下形成了一个`fiber tree`。

例如：

```js
function App() {
  return (
    <div className="App">
      <SubTree />
    </div>
  );
}

function SubTree() {
  return (
    <p>
      subTree
    </p>
  )
}
```



![fiberTree](/Users/noel/note/文章/images/fiberTree.png)



### 工作流程

`react`的工作流程实际上就是遍历`fiber tree`，对每个`fiber`去执行对应的工作。

```js
// 异步可中断任务暂时不提
function workLoopSync() {
  // workInProgress 是一个全局变量，保存当前所要执行的fiber节点，它会从root节点开始
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```



`performUnitOfWork`中会执行当前`fiber`，然后把这个`fiber`的`child`子节点赋值给`workInProgress`，当子节点不存在时，就把`sibling`兄弟节点赋值给`workInProgress` 。

上层的`workLoopSync`函数的`  while`循环会根据下个`workInProgress`去遍历。这样就能实现一个深度优先遍历，从而把所有的`fiber`执行完毕。



在`performUnitOfWork`里，又分为两个阶段，一个是`beginWork`，一个是`completeWork`。

- `beginWork`
  - 执行组件`render`
    - 在`class`组件，会执行实例化，处理`state`，调用挂载前生命周期钩子等等。最后执行`render`，获取返回的`jsx`。
    - 在`function`组件，会执行组件的构造函数，里面包括了`hooks`的一系列调用，最后获取返回的`jsx`。
  - 对返回的`jsx`执行`reconcile`,也就是俗称的`diff。`
    - 根据`diff`生成当前`fiber`的子节点，并标记上对应的`flag`，比如这个节点是更新、删除、移动。
    - 这个生成的子节点，会返回出去，赋值给`workInProgress`，然后上层函数`workLoopSync`进行下一轮遍历，执行这个新生成的`fiber`节点
- `completeWork`,当遍历到叶子节点，会执行它，对`fiber tree`进行一个回溯，去迭代`return`，也就是父节点。在发现有`sibling`兄弟节点时，会退出遍历，并赋值给`workInProgress`，以便上层`workLoopSync`函数遍历。
  - 生成`dom`节点，并把子孙`dom`节点插入进去。组成一个虚拟`dom`树
  - 处理`props`
  - 把所有含有副作用的`fiber`节点用`firstEffect`和`lastEffect`链接起来，组成一个链表，以便在`commit`时去遍历执行。



熟悉算法的人不难发现，`beginWork`和`completeWork`的交替遍历，其实就是一个回溯算法。



在`completeWork`执行到`root`根节点时，证明所有的工作已经完成，就会执行`commitRoot`，它又分为三个阶段：

- before mutation(执行`dom`操作前)
  - 调用挂载前的生命周期钩子，比如`getSnapshotBeforeUpdate`，调度`useEffect`
- mutation(执行`dom`操作)
  - 执行`dom`操作，如果有组件被删除，那么还会调用`componentWilUnmount`或`useLayoutEffect`的销毁函数
- layout(执行`dom`操作后)
  - 切换`fiber tree`
  - 调用`componentDidUpdate | componentDidMount`或者`useLayoutEffect`的回调函数。
  - `layout`结束后，执行之前调度的`useEffect`的创建和销毁函数。



总结上文，在`performUnitOfWork`时，我们称之为协调阶段，主要依靠`beginWork`和`completeWork`去交替执行每个`fiber`，在`commitRoot`时，我们称之为提交阶段。




### 双缓冲
我们在看数字电视的时候，切换下一台往往要经过一个黑屏，然后画面才显示出来。双缓冲就是在切换的时候，先在内存中进行构建UI，完成后直接渲染在界面上。这样就省掉了中间过渡的时间，从而优化体验。



在`react`中，有两棵`fiber tree`，一个是`current fiber`，一个是`workInProcess fiber`。这两个`fiber`通过`alternate`属性来进行联系。

`current fiber`是已经渲染在界面上的`fiber`。`fiber`的根节点叫做`rootFiber`，`rootFiber.current` = `current fiber`，它的`current`指向的那个`fiber`，就是当前已经渲染在界面上的`fiber`

`workInProcess fiber`是由此次更新，而正在内存中构建的`fiber`。构建完成后，`rootFiber.current` = `workInProcess fiber`，就切换为了`current fiber`，从而渲染到界面上。

由于是在内存中构建，所以它可以随时中断和恢复，不阻塞浏览器渲染。根据优先级而选择先后执行的任务，优先级高的先执行，优先级低的后执行。



联系上文「工作流程」，我们知道`react`的执行是遍历整个`fiber tree`，在遍历中，会根据`current fiber`而去`clone`一个新的`workInPorcess fiber`，这个操作是在`reconcile`中执行，如果是`mount`时，那么没有`current fiber`，会直接创建。

一直向下遍历和`clone`，就会创建出一个新的`fiber tree`，也就是`workInPorcess fiber tree`。然后根据这个新生成的树去提交，渲染到界面上。



在下文介绍的源码中，有许多是以`current`为前缀，这一般就是`current fiber`，老节点，已经渲染在界面上的。另外则是以`workInProcess`为前缀，一般就是`workInProcess fiber`，新节点，正在内存中构建的。

`current fiber`在下文统称`cur fiber`，`workInProcess fiber`在下文统称`wip fiber`。



### 总结

1. `react`的组件架构是由一个个`fiber`组成的树组成，他的工作流程就是遍历`fiber tree`去执行每一个工作单元。分为协调阶段，主要负责处理更新和`reconcile`，收集副作用并链接起来。和提交阶段，负责把副作用节点更新到界面上。
2. `fiber`有新旧两棵树，一个是`current fiber`，是已经渲染在界面上的。一个是`workInPorcess`，由当前的更新触发而在内存中构建的。构建完成，`wip fiber`就会替换`cur fiber`，渲染到界面上。



## Hooks 架构

### 数据结构

先简单看一下`hooks`所要用到的数据结构，心理有个大概的印象就行。

#### hook

每一个`hook`方法的声明，都会生成一个对应的`hook`对象，来存储一些数据。各自生成的`hook`会以`next`链接在一起，组成一个链表。然后挂载到`fiber`节点的`memoizedState`中

```js
export type Hook = {
  // 上次渲染后的state
  memoizedState: any,
  // 通过已处理好的update，来计算出的state，下文再详述。
  baseState: any,
  // 尚需处理的update，通常是上一轮render中遗留下的优先级过低而暂缓执行的update
  baseQueue: Update<any, any> | null,
  // 当前触发的update链表
  queue: UpdateQueue<any, any> | null,
  // 链接下一个hook
  next: Hook | null,
};
```

例如：

```js
const [count, setCount] = useState(0);
const [price, setPrice] = useState(10);
```

对应的`hook`链表为：

```js
const hook = {
  memoizedState: 0,
  baseState: 0,
  baseQueue: null,
  queue: null,
  next: {
    memoizedState: 10,
    baseState: 10,
    baseQueue: null,
    queue: null,
	},
}
```



其中，不同的`hook`方法，其`memoizedState`储存的东西各不相同。

| 方法                      | `memoizedState`  |
| ------------------------- | ---------------- |
| useState/useReducer       | state            |
| useEffect/useLayoutEffect | effect对象       |
| useMemo/useCallback       | [callback, deps] |
| useRef                    | {current: any}   |



#### Update

这一部分只有`useState | useReducer`会用到

```js
type Update<S, A> = {
  lane: Lane,
  action: A,
  // 触发dispatch时的reducer
  eagerReducer: ((S, A) => S) | null,
  // 触发dispatch时计算好的state
  eagerState: S | null,
  next: Update<S, A>,
  priority?: ReactPriorityLevel,
};
```

每触发一次`setState`，就会生成一个`update`对象，并链接到`hook`对象的更新队列中，也就是下文的`pending`。

```js
type UpdateQueue<S, A> = {
  // 存放当前触发的update
  pending: Update<S, A> | null,
  // 存放dispatchAction.bind()的值
  dispatch: (A => mixed) | null,
  // 上一次render时的Reducer
  lastRenderedReducer: ((S, A) => S) | null,
  // 上一次render时的state
  lastRenderedState: S | null,
};
```

例子：

```js
const [count, setCount] = useState(0);

setCount(1)
```

这个`hook`对应为：

```js
const hook = {
  memoizedState: 0,
  baseState: 0,
  baseQueue: null,
  queue: {
    {
      action: 1,
    }
	},
  next: null,
}
```

在`render`	时，会遍历`queue`来执行每个`update`并计算更新。



#### Effect

这一部分只有`useEffect  | useLayoutEffect | useImperativeHandle`会用到

```js
export type Effect = {|
  // 标记此effect是否需要执行
  tag: HookFlags,
  // 回调函数
  create: () => (() => void) | void,
  // 销毁函数
  destroy: (() => void) | void,
  // 依赖数组
  deps: Array<mixed> | null,
  next: Effect,
|};
```

例如：

```js
useEffect(() => {
  console.log('effect')
}, [])
```

对应的数据结构：

```js
const hook = {
  memoizedState: {
    create: () => {console.log('effect')},
    destroy: undefined,
    deps: [],
  },
  baseState: null,
  baseQueue: null,
  queue: null,
  next: null,
}
```



#### dispatcher

事实上，组件`mount`时，与组件`update`时，调用的是不同的`hook`方法，会通过`dispatcher.current`来指向当前所需的方法。

在`render`执行前，会根据`cur fiber`是否存在而决定全局的`dispatcher.current`指向`mount`方法还是`update`方法。

当`FunctionComponent`的`render`执行完毕后，`dispatcher.current`会指向`ContextOnlyDispatcher`，不再允许`hooks`方法的声明。

```js
// mount时的Dispatcher
const HooksDispatcherOnMount: Dispatcher = {
  useCallback: mountCallback,
  useContext: readContext,
  useEffect: mountEffect,
  useImperativeHandle: mountImperativeHandle,
  useLayoutEffect: mountLayoutEffect,
  useMemo: mountMemo,
  useReducer: mountReducer,
  useRef: mountRef,
  useState: mountState,
  // ...省略
};

// update时的Dispatcher
const HooksDispatcherOnUpdate: Dispatcher = {
  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect,
  useImperativeHandle: updateImperativeHandle,
  useLayoutEffect: updateLayoutEffect,
  useMemo: updateMemo,
  useReducer: updateReducer,
  useRef: updateRef,
  useState: updateState,
  // ...
};

export const ContextOnlyDispatcher: Dispatcher = {
  readContext,

  useCallback: throwInvalidHookError,
  useContext: throwInvalidHookError,
  useEffect: throwInvalidHookError,
  useImperativeHandle: throwInvalidHookError,
  useLayoutEffect: throwInvalidHookError,
  useMemo: throwInvalidHookError,
  useReducer: throwInvalidHookError,
  useRef: throwInvalidHookError,
  useState: throwInvalidHookError,
  // ...
};
```



### 执行流程

#### renderWithHooks

调用顺序是`beginWork --> updateFunctionComponent --> renderWithHooks`, 这个函数是`FunctionComponent`的`render`主函数。

它主要做两件事，一个是配置`hooks`所需的全局变量，一个是执行`FunctionComponent`的`render`。



**简版：**

```js
export function renderWithHooks<Props, SecondArg>(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: (p: Props, arg: SecondArg) => any,
  props: Props,
  secondArg: SecondArg,
  nextRenderLanes: Lanes,
): any {
	// 用这个变量来记录当前所执行的fiber
  currentlyRenderingFiber = workInProgress;
  
  // cur fiber 不存在或者不存在hooks链表都视为未挂载
  // 更新Dispatcher指向对应的hooks方法
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
    ? HooksDispatcherOnMount
    : HooksDispatcherOnUpdate;
  
  // Component就是组件的构造函数
  // 执行render，其中就包括了组件里面声明的hooks方法，他们都是在这个地方被执行的。
  let children = Component(props, secondArg);
  
  // 当前fiber已执行结束，重置这些全局变量
  currentlyRenderingFiber = (null: any);
  
  // 返回在render后返回的jsx
  return children;
}
```

**完整版：**

```js
export function renderWithHooks<Props, SecondArg>(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: (p: Props, arg: SecondArg) => any,
  props: Props,
  secondArg: SecondArg,
  nextRenderLanes: Lanes,
): any {
  renderLanes = nextRenderLanes;
  // 用这个变量来记录当前所执行的fiber
  currentlyRenderingFiber = workInProgress;

  // 重置wip fiber的状态，然后在后续重新创建。
  // memoizedState保存了hook链表，这一步先置空，到了render时再从cur fiber中clone每个hook
  workInProgress.memoizedState = null;
  workInProgress.updateQueue = null;
  workInProgress.lanes = NoLanes;

  // cur fiber 不存在或者不存在hooks链表都视为未挂载
  // 更新Dispatcher指向对应的hooks方法
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
    ? HooksDispatcherOnMount
    : HooksDispatcherOnUpdate;

  // Component就是组件的构造函数
  // 执行render，其中就包括了组件里面声明的hooks方法，他们都是在这个地方被执行的。
  let children = Component(props, secondArg);

  // 如果在render阶段发生了更新，会直接re-render，重新执行。
  if (didScheduleRenderPhaseUpdateDuringThisPass) {
    let numberOfReRenders: number = 0;
    do {
      didScheduleRenderPhaseUpdateDuringThisPass = false;

      numberOfReRenders += 1;

      currentHook = null;
      workInProgressHook = null;

      workInProgress.updateQueue = null;

      ReactCurrentDispatcher.current = __DEV__
        ? HooksDispatcherOnRerenderInDEV
        : HooksDispatcherOnRerender;

      children = Component(props, secondArg);
    } while (didScheduleRenderPhaseUpdateDuringThisPass);
  }
  
  // 函数组件的render已结束，关闭hooks调用接口
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

  // 当前fiber已执行结束，重置这些全局变量
  renderLanes = NoLanes;
  currentlyRenderingFiber = (null: any);

  currentHook = null;
  workInProgressHook = null;

  didScheduleRenderPhaseUpdate = false;

  // 返回在render后返回的jsx
  return children;
}
```



注意看这个地方

```js
workInProgress.memoizedState = null;
```

上文说过`fiber`的`memoizedState`中保存着`hooks`链表，在`render`之前，会先把这个引用置空，然后在`render`中，会根据`cur fiber`的`memoizedState`来`clone`出来一个个`hook`，这个在下文`updateWorkInProgress`中会详述。

这个过程跟`wip fiber tree`的创建是一样的，也是要根据老节点来一个个`clone`，生成新的节点。



再看这个地方

```js
if (didScheduleRenderPhaseUpdateDuringThisPass) {
  do {
  // ...
  } while (didScheduleRenderPhaseUpdateDuringThisPass);
}
```

这个遍历，是在`render`阶段发生了更新，然后就会`re-render`，重新执行。一直到不产生新的更新为止。

如这个例子：

```js
const [count, setCount] = useState(0);

if (count === 0) {
	setCount(1)
}
```

另外，使用`useMemo`来做这个操作的话，也是同样的效果，因为它的回调函数也同样是在`render`阶段执行的。

```js
const [count, setCount] = useState(0);

useMemo(() => {
	if (count === 0) {
		setCount(1)
	}
}, [count])
```



**流程图：**

![renderWithHooks](/Users/noel/note/文章/images/renderWithHooks.png)

(图画得不够细，`re-render`时，只需要清空`fiber`的`updateQueue`，而不是全部清空)



```js
let children = Component(props, secondArg);
```

`hooks`的调用都集中在这个步骤，也就是`render`,接下来详细讲一下。



#### mountWorkInProgressHook

每个`hooks`方法，都要创建或者取出一个`hook`节点，然后对这个节点进行写入数据。

`mount`阶段，会调用`mountWorkInProgressHook`这个方法，来创建一个`hook`节点并与其他`hook`链接在一起，然后挂载到`wip fiber`的`memoizedState`属性中，以便下次在`update`时可以从中取出链表。

```js
function mountWorkInProgressHook(): Hook {
  // 创建一个空的hook节点
  const hook: Hook = {
    memoizedState: null,

    baseState: null,
    baseQueue: null,
    queue: null,

    next: null,
  };

  // workInProgressHook 是一个全局变量，保存当前执行的最后一个hook节点。
  if (workInProgressHook === null) {
    // 把hooks链表保存到wip fiber的memoizedState中
    // currentlyRenderingFiber 这个全局变量在上文renderWithHooks中，被赋值为了当前的 wip fiber
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    workInProgressHook = workInProgressHook.next = hook;
  }
  // 返回创建好的hook节点，然后不同的hook方法就会写入各自的数据。
  return workInProgressHook;
}
```

每执行一次，`workInProgressHook`的指针就会后移，始终指向链表中最后一个节点。

注意`currentlyRenderingFiber`这个全局变量，是在上文`renderWithHooks`中，被赋值为了当前的` wip fiber`。



#### updateWorkInProgressHook

`update`阶段，会调用这个方法。主要功能是取出保存在当前`fiber`的`hooks`链表中对应的`hook`节点。

保存在`cur fiber`的`memoizedState`中的`hooks`链表，下文统称`cur hook`。

保存在`wip fiber`的`memoizedState`中的`hooks`链表，下文统称`wip hook`。



**简版：**

```js
function updateWorkInProgressHook(): Hook {
  // 迭代cur hooks链表
  const current = currentlyRenderingFiber.alternate;
  nextCurrentHook = current.memoizedState;
  
  currentHook = nextCurrentHook;
  nextCurrentHook = nextCurrentHook.next
  
  // 根据 cur hook 节点，clone 一个新的 wip hook 节点
  const newHook: Hook = {
    memoizedState: currentHook.memoizedState,

    baseState: currentHook.baseState,
    baseQueue: currentHook.baseQueue,
    queue: currentHook.queue,

    next: null,
  };
  
  if (workInProgressHook === null) {
    // 即首次创建，挂载到 wip fiber 的 memoizedState上
    currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
  } else {
    // 链接到链表尾部
    workInProgressHook = workInProgressHook.next = newHook;
  }
}
```

**完整版：**

```js
function updateWorkInProgressHook(): Hook {
  // 迭代cur hooks链表
  let nextCurrentHook: null | Hook;
  if (currentHook === null) {
    // 取出 cur fiber
    const current = currentlyRenderingFiber.alternate;
    if (current !== null) {
      nextCurrentHook = current.memoizedState;
    } else {
      nextCurrentHook = null;
    }
  } else {
    nextCurrentHook = currentHook.next;
  }

  // 迭代wip hooks链表
  let nextWorkInProgressHook: null | Hook;
  // workInProgressHook为null说明这是第一次取出wip hook，即首次创建。
  if (workInProgressHook === null) {
    // 只有re-render的情况下，wip fiber节点中仍然存在hooks链表，因为在之前的render中已经创建过wip hook了
    // 普通情况下，currentlyRenderingFiber.memoizedState会为null
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    // 普通情况下，workInProgressHook.next会为null，需要到下文创建新的hook节点然后连接上去
    nextWorkInProgressHook = workInProgressHook.next;
  }

  if (nextWorkInProgressHook !== null) {
    // 走入这个分支，是只有在render阶段setState了，导致re-render，这个时候wip hooks 链表已经创建过了。
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;

    currentHook = nextCurrentHook;
  } else {
		// 根据 cur hook 节点，clone 一个新的 wip hook 节点
    
    currentHook = nextCurrentHook;

    const newHook: Hook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      baseQueue: currentHook.baseQueue,
      queue: currentHook.queue,

      next: null,
    };

    if (workInProgressHook === null) {
      // 即首次创建，挂载到 wip fiber 的 memoizedState上
      currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
    } else {
      workInProgressHook = workInProgressHook.next = newHook;
    }
  }
  return workInProgressHook;
}
```



上面的源码可能大家会看得有点迷糊，我这里理一下逻辑：

1. 在`render`前，`wip fiber`的`memoizedState`被置空了。

   ```js
   // renderWithHooks 
   workInProgress.memoizedState = null;
   ```

   所以在`render`中，是获取不到`hooks`链表的，需要从`cur fiber`来获取。

   ```js
   const current = currentlyRenderingFiber.alternate;
   if (current !== null) {
     nextCurrentHook = current.memoizedState;
   } else {
     nextCurrentHook = null;
   }
   ```

   然后，再由这个链表来创建`wip hook`

   ```js
   currentHook = nextCurrentHook;
   
   const newHook: Hook = {
     memoizedState: currentHook.memoizedState,
   
     baseState: currentHook.baseState,
     baseQueue: currentHook.baseQueue,
     queue: currentHook.queue,
   
     next: null,
   };
   ```

   再挂载到`wip fiber`的`memoizedState`中

   ```js
   if (workInProgressHook === null) {
     currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
   } else {
     workInProgressHook = workInProgressHook.next = newHook;
   }
   ```

   这样就完成了从`cur hook`中`clone`出`wip hook`的过程。

2. 但是有个例外，那就是`re-render`，`wip hook`已经创建过了

   ```js
   if (workInProgressHook === null) {
     // 成功获取到hooks链表
     nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
   } else {
     nextWorkInProgressHook = workInProgressHook.next;
   }
   ```
   
   那么，直接迭代就行，不用再重新`clone`
   
   ```js
   if (nextWorkInProgressHook !== null) {
     workInProgressHook = nextWorkInProgressHook;
     nextWorkInProgressHook = workInProgressHook.next;
   
     currentHook = nextCurrentHook;
   }
   ```



他的主要流程就是：

- 迭代`cur hook`
- 迭代`wip hook`
  - 普通情况下`wip hook`的`next`为`null`，`re-render`情况下存在`next`
- 从`cur hook`里`clone`一个新节点，成为`wip hook`。并链接到链表尾部，或`wip fiber`的`memoizedState`(首个`wip hook`需要这样)
  - `re-render`情况下不用`clone`，迭代`wip hook`就完了
- 返回这个`hook`



### 总结

1. `render`前，会保存当前执行的`fiber`到`currentlyRenderingFiber`这个全局变量中，并清空`fiber`的状态，然后开启`hooks`方法的调用接口。

   `render`中，执行组件内的`hook`方法，如果在这个过程发生了`setState`，那就会触发`re-render`。

   `render`后，重置全局变量，关闭`hooks`方法的调用接口。

2. 每一个`hook`方法的声明，都会生成一个对应的`hook`对象，来存储一些数据。各自生成的`hook`会以`next`链接在一起，组成一个链表。然后挂载到`fiber`节点的`memoizedState`中

3. `mount`时，需要创建`hook`节点，`update`时，需要取出`hook`节点，确切的说，是从`cur hook`中`clone`一个新的`wip hook`。



题外话：

现在大家知道为什么`hooks`不能条件渲染了吧？如果前后的`hook`节点不一致，那么取值的顺序就会错误，`cur hook`也会迭代到一个`null`。



自此，`hooks`的通用架构已经讲完，接下来就到了各自的`hooks`方法的内部实现。也就是对`mountWorkInProgressHook`或`updateWorkInProgressHook`中返回的`hook`节点进行的操作。

```js
return workInProgressHook;
```



## Hooks 方法

### useState&useReducer
先看基本用法：

```js
const [state, setState] = useState(0);

setState(1);

const [state, dispatch] = useReducer(
  function reducer(state, action) {
    switch (action.type) {
      case 'increment':
        return {count: state.count + 1};
      case 'decrement':
        return {count: state.count - 1};
      default:
        throw new Error();
    }
  }, 
  {count: 0}
);

dispatch({type: 'increment'})
```



尽管后者要比前者看似复杂不少，但二者实际上是同一个方法，`useState`可视为`useReducer`的一个语法糖和简化版，是程序内部帮你赋予了一个`reducer`.

```js
// update阶段useState实际上调用的方法
function updateState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  return updateReducer(basicStateReducer, (initialState: any));
}
// 默认的reducer
function basicStateReducer<S>(state: S, action: BasicStateAction<S>): S {
	// 如果你在setState中传入的是一个方法，如setState(state => state + 1)，那么，他会先进行调用再返回。
  return typeof action === 'function' ? action(state) : action;
}
```

所以，这两个方法会合在一起说。



#### mountState

主要流程就是初始化`state`和更新队列，然后返回`state`和`dispatch`(也就是`setState`)。

```js
// mount阶段useState实际上调用的方法
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  // 如上文所述，创建一个hook节点并link到链表中
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
	// 对于useState, 其memoizedState会保存state值
  hook.memoizedState = hook.baseState = initialState;
	// 创建一个更新队列
  const queue = (hook.queue = {
    pending: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });
	// dispatchAction 下文再详述
  const dispatch: Dispatch<
    BasicStateAction<S>,
  > = (queue.dispatch = (dispatchAction.bind(
    null,
    // 把当前的 wip fiber 传入dispatch参数中
    currentlyRenderingFiber,
    queue,
  ): any));
  return [hook.memoizedState, dispatch];
}
```



来看一下这个`queue`，它就是上文「数据结构」中提到的`updateQueue`，其中`pending`属性会存放由一个个`update`	组成的链表，`update`是由`setState`时创建，这个之后再说。

```js
const queue = (hook.queue = {
  pending: null,
  dispatch: null,
  lastRenderedReducer: basicStateReducer,
  lastRenderedState: (initialState: any),
});
```

如果你熟悉链表数据结构的话，不难发现这就是一个链表的头节点，专门用来存放一些通用的数据。

- `pending`: 头指针，指向最后一个`update`，`update`会组成一个循环链表，所以只要`pending.next`就能获取到第一个节点。

- `lastRenderedState`: 存放每次`render`时的`state`

- `dispatch`

   注意，这是个由`bind`保存下来的方法，预先传入了`wip fiber`和`queue`，所以，你调用`setState`时一定能获取到最新的`state`

  ```js
  const dispatch: Dispatch<
    BasicStateAction<S>,
    > = (queue.dispatch = (dispatchAction.bind(
      null,
      currentlyRenderingFiber,
      queue,
    ): any));
  ```

  

其他的逻辑就比较简单了，具体的一些细节留待下文再详述。

接下来讲一下这个`dispatch`



#### dispatchAction

这个就是`setState`时调用的方法，也是上文的`dispatch`，主要功能是创建一个更新对象，链接到`hook`节点的更新队列中。

其中有关调度的部分本文先不讲

**简版：**

```js
function dispatchAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  // 创建一个更新事件
  const update: Update<S, A> = {
    lane,
    action,
    eagerReducer: null,
    eagerState: null,
    next: (null: any),
  };
  
  // 把update链接到更新队列中，组成单循环链表
  const pending = queue.pending;
  if (pending === null) {
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  // pending指向链表的最后一个
  queue.pending = update;
  
  // 开启调度，触发新的一轮更新，也就是走beginWork,completeWork那一套流程。
  scheduleUpdateOnFiber(fiber, lane, eventTime);
}
```

**完整版：**

```js
function dispatchAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  
  // 获取事件所需时间，和优先级
  const eventTime = requestEventTime();
  const lane = requestUpdateLane(fiber);

  const update: Update<S, A> = {
    lane,
    action,
    eagerReducer: null,
    eagerState: null,
    next: (null: any),
  };

  // 把update链接到更新队列中，组成单循环链表
  // pending指向链表的最后一个
  const pending = queue.pending;
  if (pending === null) {
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  queue.pending = update;

  const alternate = fiber.alternate;
  if (
    fiber === currentlyRenderingFiber ||
    (alternate !== null && alternate === currentlyRenderingFiber)
  ) {
    // currentlyRenderingFiber仍然存在，证明这是在render中发生的更新.
    didScheduleRenderPhaseUpdateDuringThisPass = didScheduleRenderPhaseUpdate = true;
  } else {
    if (
      fiber.lanes === NoLanes &&
      (alternate === null || alternate.lanes === NoLanes)
    ) {
      // fiber.lanes === NoLanes，说明此前未发生更新，本次是第一个
      // 我们可以预先根据reducer来计算state值，如果与当前值相同，则跳过更新。
      // 如果值不同，则保存在eagerState，下次render时可以直接使用，而无需再计算。
      const lastRenderedReducer = queue.lastRenderedReducer;
      if (lastRenderedReducer !== null) {
        // 上次render时的state
        const currentState: S = (queue.lastRenderedState: any);
        const eagerState = lastRenderedReducer(currentState, action);
        update.eagerReducer = lastRenderedReducer;
        // 保存计算出来的state
        update.eagerState = eagerState;
        if (is(eagerState, currentState)) {
          // 如果计算好的state和当前的state相同，则不进行更新调度
          return;
        }
      }
    }
    // 开启调度，触发新的一轮更新，也就是走beginWork,completeWork那一套流程。
    scheduleUpdateOnFiber(fiber, lane, eventTime);
  }

  if (enableSchedulingProfiler) {
    markStateUpdateScheduled(fiber, lane);
  }
}
```



`pending`是一个循环链表，并且指向链表中最后一个节点

```js
const pending = queue.pending;
if (pending === null) {
  update.next = update;
} else {
  update.next = pending.next;
  pending.next = update;
}
queue.pending = update;
```

那么，为什么要用这种形式呢？因为它既有把节点`push`到尾部的需求，即`setState`。也有从头开始遍历链表的需求，即下文的`updateReducer`。所以用循环链表可以很简单的实现这两个需求，`pending.next`就是第一个节点，而`pending`本身就是最后一个节点。在`react`源码内的很多地方都采用了这种方式，以最简单的数据结构来实现获取链表两端的需求。



这个地方，这是为了在`render`阶段发生的`setState`，然后触发`re-render`，而要做的一个标记。

```js
if (
  fiber === currentlyRenderingFiber ||
  (alternate !== null && alternate === currentlyRenderingFiber)
) {
  didScheduleRenderPhaseUpdateDuringThisPass = didScheduleRenderPhaseUpdate = true;
}
```

还记得`renderWithHooks`中，对`currentlyRenderingFiber`的赋值吗？

```js
// `renderWithHooks`
currentlyRenderingFiber = workInProgress;
let children = Component(props, secondArg);
currentlyRenderingFiber = (null: any);
```

正常情况下，	`render`结束后，`	currentlyRenderingFiber`就不存在了。所以如果`fiber === currentlyRenderingFiber`，就证明是在`render`阶段发生的更新

```js
// `renderWithHooks`

  do {
    didScheduleRenderPhaseUpdateDuringThisPass = false;

    numberOfReRenders += 1;

    // Start over from the beginning of the list
    currentHook = null;
    workInProgressHook = null;

    workInProgress.updateQueue = null;

    ReactCurrentDispatcher.current = __DEV__
      ? HooksDispatcherOnRerenderInDEV
    : HooksDispatcherOnRerender;

    children = Component(props, secondArg);
  } while (didScheduleRenderPhaseUpdateDuringThisPass);

```

如果被标记为`didScheduleRenderPhaseUpdateDuringThisPass`，就会重新执行`render`，直到没有为止。



这个地方，则是一个小优化，如果当前`setState`是第一个，那么在`setState`时，就会计算一下他的`state`值。

```js
if (
  fiber.lanes === NoLanes &&
  (alternate === null || alternate.lanes === NoLanes)
) {
  // ...
}
```

`fiber.lanes === NoLanes`证明当前没有发生过更新，接下来，会根据它的`reducer`计算`state`值

```js
const lastRenderedReducer = queue.lastRenderedReducer;
if (lastRenderedReducer !== null) {
  // 上次render时的state
  const currentState: S = (queue.lastRenderedState: any);
  const eagerState = lastRenderedReducer(currentState, action);
  // 保存计算出来的state
  update.eagerReducer = lastRenderedReducer;
  update.eagerState = eagerState;
  if (is(eagerState, currentState)) {
    // 如果计算好的state和当前的state相同，则不进行更新调度
    return;
  }
}
```

比如这个例子：

```js
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(1) // 第一个setState，会直接计算state，保存在eagerState中
  setCount(2) // 第二个，就不进行计算了，push到queue链表中，留待下次render时执行
}, [])
```



`dispatch`之后，会触发一轮新的`react`更新调度，`scheduleUpdateOnFiber(fiber, lane, eventTime);`。也就是走`beginWork ---> updateFunction ---> renderWithWork`那一套流程。再到`render`时，重新执行那些`hooks`方法，也就到了`update`阶段。



#### updateReducer

`update`阶段，会循环执行`update`链表，计算最新的`state`。

**简版：**

```js
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
	// 取出对应的hook节点
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;
  let baseQueue = queue.pending;
  let newState = current.baseState;

  // 遍历链表，计算state
	do {
		const action = update.action;
    newState = reducer(newState, action);
	} while (update !== null && update !== first);

  hook.memoizedState = newState;

	const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```

**完整版：**

```js
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  // 取出对应的hook节点
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;

  queue.lastRenderedReducer = reducer;

  const current: Hook = (currentHook: any);

  // 上次render未能处理完的update
  let baseQueue = current.baseQueue;

  // 把pending链接到baseQueue的尾部，以便一起迭代执行
	// 下面的这些操作就是把循环链表剪开，然后拼接在一起
  const pendingQueue = queue.pending;
  if (pendingQueue !== null) {
    if (baseQueue !== null) {
      const baseFirst = baseQueue.next;
      const pendingFirst = pendingQueue.next;
      baseQueue.next = pendingFirst;
      pendingQueue.next = baseFirst;
    }
    // 为什么也要赋值给cur hook的baseQueue里呢？
    // 这涉及到异步可中断模型，在当前的更新执行时，发生了另一个高优先级的任务，这会打断前一个任务，先执行后一个。
    // 但是这样会使得前一个任务的update丢失，
    current.baseQueue = baseQueue = pendingQueue;
    queue.pending = null;
  }

  if (baseQueue !== null) {
    const first = baseQueue.next;
    // 最新的state，经由所有被执行过的update计算而来
    let newState = current.baseState;

    // 相当于master分支的state和update链表
    let newBaseState = null;
    let newBaseQueueFirst = null;
    let newBaseQueueLast = null;
    let update = first;
    // 遍历链表，计算state
    do {
      const updateLane = update.lane;
      if (!isSubsetOfLanes(renderLanes, updateLane)) {
        // 当前update优先级过低，跳过执行。
        const clone: Update<S, A> = {
          lane: updateLane,
          action: update.action,
          eagerReducer: update.eagerReducer,
          eagerState: update.eagerState,
          next: (null: any),
        };
        // 把这个update添加到下次的baseQueue中，留待下次render时执行
        if (newBaseQueueLast === null) {
          newBaseQueueFirst = newBaseQueueLast = clone;
          // 更新baseState为之前update计算好的那个值
          newBaseState = newState;
        } else {
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }
        currentlyRenderingFiber.lanes = mergeLanes(
          currentlyRenderingFiber.lanes,
          updateLane,
        );
        markSkippedUpdateLanes(updateLane);
      } else {

        if (newBaseQueueLast !== null) {
          // newBaseQueueLast 存在证明此前有update被跳过
          // 因为update互相之间可能存在依赖，所以从那个被跳过的update起，所有后面的update都链接到newBaseQueueLast中
          // 在下次render一齐执行
          const clone: Update<S, A> = {
            lane: NoLane,
            action: update.action,
            eagerReducer: update.eagerReducer,
            eagerState: update.eagerState,
            next: (null: any),
          };
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }

        if (update.eagerReducer === reducer) {
          // eagerReducer不为null，证明dispatch的时候，使用过这个reducer来计算过state值
          // 所以eagerReducer和eagerState都被赋值了。
          // 再检查一下上个reducer和当前的reducer有没有改变，没有的话就可以直接使用其计算出来的值
          newState = ((update.eagerState: any): S);
        } else {
          // 通过reducer来计算state值
          const action = update.action;
          newState = reducer(newState, action);
        }
      }
      update = update.next;
    } while (update !== null && update !== first);

    if (newBaseQueueLast === null) {
      // 证明所有的update都已经处理完，此时更新baseState
      newBaseState = newState;
    } else {
      newBaseQueueLast.next = (newBaseQueueFirst: any);
    }

    if (!is(newState, hook.memoizedState)) {
      markWorkInProgressReceivedUpdate();
    }

    // 如果有update被跳过，那么memoizedState与baseState会是不同的值
    // baseState会停留在被跳过的update之前的计算值。
    hook.memoizedState = newState;
    hook.baseState = newBaseState;
    hook.baseQueue = newBaseQueueLast;

    queue.lastRenderedState = newState;
  }

  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```



可能大家会觉得这部分代码有点复杂，其实原因在于要实现这两个功能：

1. 异步可中断模型，前一个协调阶段的任务执行可以被打断，让位给更高优先级的任务。这个时候，要保证低优先级的任务的状态不会丢失。

2. 更新队列中，优先级低的`update`会被暂缓执行，只执行高优先级的任务。这个时候，要保证最后所有的任务执行完毕时，`update`的顺序不被打乱。因为互相的`update`是有可能相互依赖的

   比如：

   ```js
   setState(pre => pre + 1)
   setState(pre => pre + 2)
   ```



##### 异步可中断模型

想象一个例子，有一个网页，每隔五秒会自动变幻颜色，那么，这个任务是不是一个低优先级的？因为我们并不会在意他是否及时的改变颜色。

在这期间，我们在网页中输入文字，这个任务是不是一个高优先级的？因为我们需要文字及时的呈现在网页中。

所以，当网页正在变幻颜色(即改变`state`时)，我们恰好输入了一个文字，那么，后面的任务，就会打断前面的任务，然后等到文字呈现到界面上后，再去变幻颜色。

举个例子：

```js
function App() {
  const [color, setColor] = useState('black');

  const [value, setValue] = useState('app');

  useEffect(() => {
    setTimeout(() => {
      setColor(pre => {
        pre === 'red' ? 'black' : 'red'
      })
    }, 5000)
  }, [color]);

  return (
    <div className="App" style={{backgroundColor: color}}>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
    </div>
  );
}
```

![asyc](/Users/noel/note/文章/images/asyc.png)



在实际中，并不只是有这两个任务，在多个任务互相"插队"的情况下，如何保证所有的任务最终都能顺利执行，而不存在丢失掉某个状态更新呢？

可以类比`git`，会有一个主分支`master`，有一堆功能分支，当有一个高优先分支需要提交到`master`时，需要先`pull`，然后再`push`。这样，所有的功能分支都可以以`master`作为一个基准。



在`useState`中，`baseQueue`和`baseState`就是那个`master`分支。

所以你能在这个地方，看到`update`队列也链接到了`cur hook`的`baseQueue`中。

```js
const pendingQueue = queue.pending;
if (pendingQueue !== null) {
  if (baseQueue !== null) {
    const baseFirst = baseQueue.next;
    const pendingFirst = pendingQueue.next;
    baseQueue.next = pendingFirst;
    pendingQueue.next = baseFirst;
  }
  // 同时保存到 cur hook里
  current.baseQueue = baseQueue = pendingQueue;
  queue.pending = null;
}
```

因为后来的任务如果打断了前一个任务，那么他仍然是从`cur fiber`中去`clone`并执行。因此，前一个任务的`update`如果也保存在了`cur hook`里，那么就不会丢失，也会在后来被执行。



##### 保证`update`的执行顺序

再来看一个例子：

```js
function App() {
  const [count, setCount] = useState(0);

  const ref = useRef()

  useEffect(() => {
    setTimeout(() => setCount(1), 0) // 任务1
    setTimeout(() => ref.click(), 4) // 任务2
  }, []);

  return (
    <div className="App" >
      <button ref={ref} onClick={() => setCount(pre => pre + 1)} />
    </div>
  );
}
```



```js
// 任务1触发，进入第一轮render
// 执行updateReducer前，数据结构是这样的
const wipHook = {
  memoizedState: 0,
  queue: {
    pending: {
      action: 1
    }
  }
}

// 执行updateReducer后，数据结构变成了这样：
const wipHook = {
  memoizedState: 1,
  queue: {
    pending: null
  }
}

const curHook = {
  memoizedState: 1,
  queue: null,
  baseQueue: {
    action: 1
  }
}

// 发现了吗？wip hook的pending更新队列执行完毕后，变成了null，但是却赋值给了cur Hook的baseQueue

// 任务2触发，发现任务1处在协调阶段，即打断，执行任务2
// 由于任务1未提交，所以任务2在执行的时候，仍然是从老的那个cur fiber去clone新树
// 进入第二轮render

// 执行updateReducer前的数据结构：
const wipHook = {
  memoizedState: 0,
  queue: {
    pending: {
      action: (pre) => pre + 1
    }
  }
}

const curHook = {
  memoizedState: 0,
	queue: {
    pending: {
      action: (pre) => pre + 1
    }
  }
  baseQueue: {
    action: 1
  }
}
// 注意其中的区别，curHook的baseQueue仍然带着任务1的update

// 执行updateReducer后的数据结构：
const wipHook = {
  // 执行了任务2，(pre) => pre + 1 ， 所以state由0，变为了1
  memoizedState: 1,
  queue: {
    pending: null
  },
  // 由于任务1优先级低，所以先执行了任务2
  // 因此，从被跳过的update开始，把他链接到baseUpdate中
  baseQueue: {
    action: 1,
    next: {
      action: 2
    }
  }
  // baseState 也停留在未被跳过update时的值
  baseState: 0,
}

const curHook = {
  memoizedState: 0,
	queue: {
    pending: null
  }
  baseQueue: {
    action: 1,
    next: {
      action: 2
    }
  }
}

// wip fiber提交，变为cur hook，发现有任务未执行完，再次调度
// 进入第三轮render：
const wipHook = {
  memoizedState: 1,
  baseState: 0,
  queue: {
    pending: null
  },
  baseQueue: {
    action: 1,
    next: {
      action: 2
    }
  }
}

// 执行updateReducer，遍历执行baseQueu：
// action1: state = 1, action2: state = 1(action1计算完的state) + 1
const wipHook = {
  memoizedState: 2,
  baseState: 2,
  queue: {
    pending: null
  },
  // 已无被跳过的update
  baseQueue: null
}

// 提交，state最终为2

// 整个流程是state从 0 -->  1 ---> 2 的变化
```

**只要有`update`被跳过，那么`baseState`就会停留在被跳过的`update`之前的计算值。然后`baseQueue`会保存从那个被跳过的`update`，及其之后的所有成员。**

```js
// updateReducer

// 当前update优先级过低，跳过执行。
if (!isSubsetOfLanes(renderLanes, updateLane)) {
  const clone: Update<S, A> = {
    lane: updateLane,
    action: update.action,
    eagerReducer: update.eagerReducer,
    eagerState: update.eagerState,
    next: (null: any),
  };
  // 把这个update添加到下次的baseQueue中，留待下次render时执行
  if (newBaseQueueLast === null) {
    newBaseQueueFirst = newBaseQueueLast = clone;
    // 更新baseState为之前update计算好的那个值
    newBaseState = newState;
  } else {
    newBaseQueueLast = newBaseQueueLast.next = clone;
  }
  currentlyRenderingFiber.lanes = mergeLanes(
    currentlyRenderingFiber.lanes,
    updateLane,
  );
  markSkippedUpdateLanes(updateLane);
}

// 同时，也保存它之后的所有update
else {

  if (newBaseQueueLast !== null) {
    // newBaseQueueLast 存在证明此前有update被跳过
    // 因为update互相之间可能存在依赖，所以从那个被跳过的update起，所有后面的update都链接到newBaseQueueLast中
    // 在下次render一齐执行
    const clone: Update<S, A> = {
      lane: NoLane,
      action: update.action,
      eagerReducer: update.eagerReducer,
      eagerState: update.eagerState,
      next: (null: any),
    };
    newBaseQueueLast = newBaseQueueLast.next = clone;
  }
  // ...
}

```



除开这两个逻辑，其他的就不难了，不过是遍历`update`来计算`state`而已。



#### 总结

1. `useState`通过一个更新队列来记载所触发的更新，其中，`pending`所记载的是本次任务触发的`update`，`baseUpdate`所记载的是所有异步任务所触发的`update`，相当于一个全局的仓库。

   在`render`中，会遍历执行`update`来计算`state`值，如果某个`update`优先级过低，就会暂缓执行，先执行其他的`update`并`commit`到界面上，然后在下次`render`中，再从被跳过的`update`开始执行任务，保证其中顺序不变。

2. `dispatch`可以生成一个`update`，链接到`hook`的更新队列中。如果本次`dispatch`是第一个，那么会直接计算`state`值，在下次`render`时就可以直接使用，而无需计算。

   

那么到这里`useState`就已经讲完了，有人可能会问疑惑`mountReducer`怎么没有，因为它的代码和`mountState`有90%的相似度，所以就略过。



### useMemo&useCallback

官网例子：

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

> 把“创建”函数和依赖项数组作为参数传入 `useMemo`，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。



```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

> 把内联回调函数及依赖项数组作为参数传入 `useCallback`，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。



可以看到，他们的区别就是有没有回调函数而已。事实上在源码中他们的实现也是90%的相似



#### mountMemo&mountCallback

```js
function mountMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  // 创建一个hook节点并link到链表中
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function mountCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  // 创建一个hook节点并link到链表中
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

这个`hook`的`memoizedState`存放的是一个数组

`[callback, nextDeps]`

第一项是保存的值，第二项是依赖。



#### updateMemo&updateCallback

```js
function updateMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  // 取出hook节点
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      // 对比前后依赖项，如果相同，就直接返回缓存值
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  // 如果不同，则依照最新的create来重新计算，并更新依赖项
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function updateCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  // 取出hook节点
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      // 对比前后依赖项，如果相同，就直接返回缓存值
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  // 如果不同，更新回调函数，更新依赖项
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```



对比前后依赖项是否相同，都是靠着这个方法：

```js
function areHookInputsEqual(
  nextDeps: Array<mixed>,
  prevDeps: Array<mixed> | null,
) {
  for (let i = 0; i < prevDeps.length && i < nextDeps.length; i++) {
    // object.is()
    if (is(nextDeps[i], prevDeps[i])) {
      continue;
    }
    return false;
  }
  return true;
}
```



总的来说，这个`hooks`还是相对简单的。



### useEffect

官网例子:

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```



我把他分为两个阶段，一个是调用阶段，是在`render`中的显式声明。一个是执行阶段，是在`effect`的回调函数执行的时候。



#### 调用阶段

##### mountEffectImpl

`mount`阶段`useEffect`调用的方法：

```js
function mountEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  return mountEffectImpl(
    UpdateEffect | PassiveEffect | PassiveStaticEffect,
    HookPassive,
    create,
    deps,
  );
}

function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
  // 创建一个hook节点并link到链表中
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  currentlyRenderingFiber.flags |= fiberFlags;
  // 创建一个effect对象并添加到hook的memoizedState中
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create, 
    undefined,
    nextDeps,
  );
}
```



##### pushEffect

这个方法主要做两件事，一个是创建`effect`对象并返回，一个是把这个`effect`链接到`wip fiber`的`updateQueue`中。

```js
function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag,
    create,// 回调函数
    destroy, // 依赖项
    deps,
    // 这也是个循环链表
    next: (null: any),
  };
  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  if (componentUpdateQueue === null) {
    // 首个副作用，给当前的fiber创建一个UpdateQueue，并把effect添加进去。
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    // 链接到当前fiber节点的updateQueue的lastEffect中
    const lastEffect = componentUpdateQueue.lastEffect;
    // 组成循环链表
    if (lastEffect === null) {
      componentUpdateQueue.lastEffect = effect.next = effect;
    } else {
      const firstEffect = lastEffect.next;
      lastEffect.next = effect;
      effect.next = firstEffect;
      componentUpdateQueue.lastEffect = effect;
    }
  }
  return effect;
}
```



对于函数组件，其`fiber`的`updateQueue`，也同样是一个链表的头节点，`lastEffect`指向最后一个`effect`。与上文`hook.queue.pending`的实现方式相同。

```js
export type FunctionComponentUpdateQueue = {|lastEffect: Effect | null|};
```

把`effect`放到`fiber`节点的`updateQueue`中，以便在`commit`阶段就可以发起一个调度延迟执行。



##### updateEffectImpl

`update`阶段`useEffect`调用的方法：

```js
function updateEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
    
  return updateEffectImpl(
    UpdateEffect | PassiveEffect,
    HookPassive,
    create,
    deps,
  );
}

function updateEffectImpl(fiberFlags, hookFlags, create, deps): void {
  // 取出hooks节点
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined;

  if (currentHook !== null) {
    // 取出上一轮render中的effect
    const prevEffect = currentHook.memoizedState;
    destroy = prevEffect.destroy;
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps;
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        // 如果上一轮render的依赖项和当前的依赖项未发生变化，就无需更新hook的memoizedState
        // clone一个effect对象链接到updateQueue中，但是tag不添加HookHasEffect
        pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
    }
  }

  currentlyRenderingFiber.flags |= fiberFlags;

  // 走到这里说明依赖项变更
  hook.memoizedState = pushEffect(
    // 标记此effect对象的tag为HookHasEffect，表示有副作用，需要执行。
    HookHasEffect | hookFlags,
    create,
    destroy,
    nextDeps,
  );
}
```



这个地方，为什么要用`cur hook`而非`wip hook`?明明这两个的`memoizedState`都是相同的。

```js
const prevEffect = currentHook.memoizedState;
destroy = prevEffect.destroy;
```

原因仍然是`re-render`，这种情况发生时，`wip hook`不会重新从`cur hook`里`clone`	，所以，两次的依赖项会是相同的，导致`effect`的回调不会执行。

```js
// clone一个effect对象链接到updateQueue中，但是tag不添加HookHasEffect,commit阶段不会执行它。
pushEffect(hookFlags, create, destroy, nextDeps);
return;
```

因此，当你需要取到上一轮`render`中的数据时，一定要用`cur hook`，尽管大部分情况下，`wip hook`都是`cur hook`的克隆体。



上面都是`useEffect`的调用，接下来就到`useEffect`的执行了。



#### 执行阶段

在`fiber` 的「工作流程」中说过。`effect`的执行，会在`commit`的`beformMutation`阶段，去发起一个调度，可以暂时理解为一个`setTimeout`。

然后到了`layout`阶段结束后，就开始执行异步任务，也就是`effect`的销毁回调和创建回调。

下面放源码，我会适当简化一下，因为`commit`	阶段对于本文而言属于超纲，有空我再详细说下。



##### commitBeforeMutationEffects

在这个阶段，发起`useEffect`的调度。

```js
// 省略代码
function commitBeforeMutationEffects() {
  while (nextEffect !== null) {
    if ((flags & Passive) !== NoFlags) {
      if (!rootDoesHavePassiveEffects) {
        rootDoesHavePassiveEffects = true;
        // 发起调度
        scheduleCallback(NormalSchedulerPriority, () => {
          flushPassiveEffects();
          return null;
        });
      }
    }
    nextEffect = nextEffect.nextEffect;
  }
}
```



##### flushPassiveEffectsImpl

先调用`effect`的卸载，再调用创建

```js
// 省略代码
function flushPassiveEffectsImpl() {
  if (rootWithPendingPassiveEffects === null) {
    return false;
  }

  // 调用 destroy 回调
  commitPassiveUnmountEffects(root.current);
  // 调用 create 回调
  commitPassiveMountEffects(root, root.current);

  flushSyncCallbackQueue();

  return true;
}
```



##### commitHookEffectListUnmount

调用`effect`的卸载

```js
function commitHookEffectListUnmount(flags: HookFlags, finishedWork: Fiber) {
  // 取出fiber的updateQueue，effect的链表都保存在这里面
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & flags) === flags) {
        // Unmount
        const destroy = effect.destroy;
        effect.destroy = undefined;
        if (destroy !== undefined) {
          // 调用 destroy
          safelyCallDestroy(finishedWork, destroy);
        }
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}
```



##### commitHookEffectListMount

调用`effect`的创建

```js
function commitHookEffectListMount(tag: number, finishedWork: Fiber) {
  // 取出fiber的updateQueue，effect的链表都保存在这里面
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      // 这个tag能控制哪些effect需要执行，一般来说，只有标记了 HookHasEffect 的，才会执行。
      if ((effect.tag & tag) === tag) {
        // Mount
        const create = effect.create;
        effect.destroy = create();
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}
```



#### 总结

1. `useEffect`会生成一个`effect`对象，保存在`hook`节点的`memoizedState`中，同时也`push`到`fiber`节点的`updateQueue`里，组成循环链表。

   每次`render`时，都会对比一下`cur hook`和`wip hook`里保存的`effect`的`deps`有没有改变，如果改变了，那就更新`memoizedState`为最新的`effect`，并且把`effect`的`tag`打上`HasHookEffect`的标记，然后`push`到`fiber`的`updateQueue`里。

2. 在`commit`阶段，`beforeMutation`中，对有副作用的`fiber`，发起一个异步调度。

   等到`layout`结束后，这个异步调度的回调开始执行，处理`effect`的创建和销毁回调。

   它会先调用`effect`的`destroy`，再调用`create`。



`useLayoutEffect`和`useEffect`	大同小异，只是打的`tag`不同，并且执行阶段在`layout`中同步执行，和`componentDidMount`及`componentDidUpdate`的相同。

这个`hooks`我在之后再更新。



### useRef

官网例子：

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

> `useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变。



源码：

```js
function mountRef<T>(initialValue: T): {|current: T|} {
  const hook = mountWorkInProgressHook();
  const ref = {current: initialValue};
  hook.memoizedState = ref;
  return ref;
}

function updateRef<T>(initialValue: T): {|current: T|} {
  const hook = updateWorkInProgressHook();
  return hook.memoizedState;
}
```



可以说是相当简单了，有了上面几个`hooks`基础的大家想必都能明白。

从源码中可以看出，`useRef`只是把一个对象储存了起来然后保存到`hook`里，并且每次`render`都返回同一个对象而不做任何改变。

这也就是它被称为可变数据的原因，你在修改这个对象里面的属性时，不论是`cur hook`还是`wip hook`，还是函数闭包陷阱，他们的`ref`引用都是一致的，从中访问`current`，总是能取得相同的值。

但同样的，修改`ref`不会引起调度，也就是不会触发渲染。只有`setState || React.render`才会引起渲染，从内存中构建一颗新树然后替换老树。这也就是后者必须是`immutable`的原因。



`ref`还涉及到组件更新`ref`，以及`useImperativeHandle`，有空再更新。



## 后记

本文所讲的`react hooks`的源码到这里就结束了，其中还有一些细节尚待补充，以后我有时间会添上去。

大家有发现什么不对的地方，或者有哪里绝对说得不够明白，欢迎在评论区留言~

