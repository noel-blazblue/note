## 前言
其实了解react的源码，并不是一件高成本的事，它的内部架构没有大家想象中的那么难，而阅读源码的收获，却是远远大过了成本，能够学习到其中优秀的设计模式和数据架构，并对业务中的熟练度有非常大的提升。因此，这是在前端进阶上非常值得去做的一件事。

**推荐的学习路线：**
从社区上一些解读文章中了解react的整体架构和运作流程，然后自己再对照源码，对其中的各个环节进行细致的研究和验证。

本文主要解释 hooks 这部分的源码，对于fiber架构和任务调度，只会略微提及。不过我建议大家最好先了解一下，尽管不懂这部分流程，仍然可以看懂hook。

社区上的一些优秀文章：

通过这一篇文章，你可以懂得：
- hooks 是如何保存在组件中的
- setState 的更新流程
- useEffect，useLayoutEffect, useImperativeHandle 的区别
- useMemo 的执行阶段


## Fiber 架构

### 数据结构
正如同`react`推崇的组件化一样，在源码内部，也是由一个个"组件"去组成整个架构。这个"组件"就是`fiber`，它是 `React` 中的一个基本工作单元，`React`的一切操作都要基于它去实现。

```js
interface Fiber {
  // 1. Instance 类型信息
  // 标记 Fiber 的实例的类型, 例如函数组件、类组件、宿主组件(即dom)
  tag: WorkTag,
  // class、function的构造函数，或者dom组件的标签名。
  type: any,
	// class、function的构造函数，或者dom组件的标签名。
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
	// 指向上次渲染的fiber节点
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
- `completeWork`,当遍历到叶子节点，会执行它，对`fiber tree`进行一个回溯，去迭代`return`也就是父节点。在发现有`sibling`兄弟节点时，会赋值给`workInProgress`，以便上层`workLoopSync`函数遍历。
  - 生成`dom`节点，并把子孙`dom`节点插入进去。组成一个虚拟`dom`树
  - 处理`props`
  - 把所有含有副作用的`fiber`节点用`firstEffect`和`lastEffect`链接起来，组成一个链表，以便在`commit`时去遍历执行。



在`completeWork`执行到`root`根节点时，证明所有的工作已经完成，就会执行`commitRoot`，把更新提交并渲染到界面上。



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

每一个`hook`方法的声明，都会生成一个对应的`hook`对象，来存储一些数据。各自生成的`hook`会以`next`链接在一起，组成一个链表。

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

这个过程跟`wip fiber tree`的创建是一样的，也是要根据`cur fiber tree`来一个个`clone`，生成新的节点。



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



```js
let children = Component(props, secondArg);
```

`hooks`的调用都集中在这个步骤，也就是`render`,接下来详细讲一下。



#### mountWorkInProgressHook

每个`hook`方法都要根据当前组件是在`mount`还是在`update`，来决定它是要创建一个`hook`节点，还是取出在`mount`时创建好的`hook`节点。

`mount`时，会调用`mountWorkInProgressHook`这个方法，来创建一个`hook`节点并与其他`hook`链接在一起，然后挂载到`wip fiber`的`memoizedState`属性中，以便下次在`update`时可以从中取出链表。

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
    // 挂载到wip fiber的memoizedState中
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

每执行一次，`workInProgressHook`的指针就会后移，始终指向链表中最后一个节点。



### 挂载阶段

众所周知，每一次`render`，函数组件都会重新执行一遍。因此里面的hooks方法也会跟着执行。在它第一次执行的时候，我们称之为**挂载阶段**，往后就称之为**更新阶段**。

事实上，在react内部，这两个阶段调用的是不同的hooks方法。



```js
// 挂载阶段
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
  useTransition: mountTransition,
  useMutableSource: mountMutableSource,
  useOpaqueIdentifier: mountOpaqueIdentifier,
};

// 更新阶段
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
  useTransition: updateTransition,
  useMutableSource: updateMutableSource,
  useOpaqueIdentifier: updateOpaqueIdentifier,
};
```



在`renderWithHooks`中，会判断当前这个阶段所需的是哪些方法。

```js
// 挂载时，`current`参数会传null，更新时，`current`参数会传当前fiber节点。
function renderWithHooks (current) {
	ReactCurrentDispatcher.current =
		current === null || current.memoizedState === null
			? HooksDispatcherOnMount
			: HooksDispatcherOnUpdate;
}
```



`mount`系列的hooks，会用`mountWorkInProgressHook`这个方法，来创建一个节点，并link到链表的尾部，链表的头部则赋值到当前的fiber节点的`memoizedState`属性中。

`memoizedState`，顾名思义，是保存一个状态，如 Class组件中的 `this.state` 。而在函数组件中，hooks就是他的状态。



```js
function mountWorkInProgressHook(): Hook {
  // 创建一个hook节点
  const hook: Hook = {
    memoizedState: null,

    baseState: null,
    baseQueue: null,
    queue: null,

    next: null,
  };

  if (workInProgressHook === null) {
// workInProgressHook === null 证明现在创建的是首个hook节点，就需要把他链接到fiber节点上
		// currentlyRenderingFiber: 当前的fiber节点
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
		// workInProgressHook：指向当前的hook节点
		// 把创建好的hook节点链接到链表的尾部
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

节点链接上后，会返回节点本身，然后不同的hooks方法就可以对这个节点进行不同的处理，比如`useState`会把`initialState`放置到节点的`memoizeState`中。这个等到下文讲api时再具体述说。



例子：

```js
function Test() {
	const [state1, setState1] = useState()
	const [state2, setState2] = useState()
	
	useEffect1(
		() => {},
		[]
	)
	useEffect2(
		() => {},
		[]
	)
}
```

会实现成：

![[Pasted image 20210327125830.png]]

这样在下一次更新时，直接迭代链表就可以处理每个hook了。



### 更新阶段

到了更新阶段，与挂载时对应，hooks方法所要做的就是从链表中取出对应的节点。

```js

function updateWorkInProgressHook(): Hook {
	// 这块的逻辑主要是迭代旧树fiber,也即是上次渲染的fiber节点的hooks链表
	// 如果是第一个hook，则从fiber节点的memoizedState属性，取出hook链表的头节点
	// 反之则指针后移，迭代链表。
  let nextCurrentHook: null | Hook;
  if (currentHook === null) {
	  // 如果是第一个hook，即currentHook不存在，
    // 则在上次渲染的fiber节点中，从它的memoizedState属性，取出hook链表的头节点
    // alternate, 为fiber节点在旧树中的引用
    const current = currentlyRenderingFiber.alternate;
    if (current !== null) {
      nextCurrentHook = current.memoizedState;
    } else {
      nextCurrentHook = null;
    }
  } else {
		// 反之，则进行迭代
    nextCurrentHook = currentHook.next;
  }
	
	// 这块的逻辑主要是迭代新树中fiber节点的hooks链表
	// 和上面的逻辑唯一不同点在于他是取新树中的fiber节点
  let nextWorkInProgressHook: null | Hook;
  if (workInProgressHook === null) {
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    nextWorkInProgressHook = workInProgressHook.next;
  }
	
	if (nextWorkInProgressHook !== null) {
		// 迭代
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;
    currentHook = nextCurrentHook;
  } else {
		currentHook = nextCurrentHook;
		// clone 旧hook节点的属性，创建新的hook节点，
		const newHook: Hook = {
			memoizedState: currentHook.memoizedState,

			baseState: currentHook.baseState,
			baseQueue: currentHook.baseQueue,
			queue: currentHook.queue,

			next: null,
		};

		if (workInProgressHook === null) {
			// workInProgressHook：工作中的hook
			// 为null，则当前是第一个调用的hook
			// 把新节点添加到 workInProgressHook ，并连接到fiber节点的memoizedState属性里
			currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
		} else {
			// 添加到链表尾部
			workInProgressHook = workInProgressHook.next = newHook;
		}
	}
  return workInProgressHook;
}
```

上面的代码可能有些人看起来会有些疑惑，比如：

`currentHook`,`nextCurrentHook`

`workInProgressHook`,`nextWorkInProgressHook`

这两组变量有什么区别？

区别在于前者是旧树中的hook链表，后者是新树中的hook链表。这两组属性分别迭代。这样就可以用旧树的hook的属性，来创建新的节点。



**总的流程：**
- 迭代旧树hooks链表
- 迭代新树hooks链表
- 根据旧树创建新的hook节点
- 返回这个hook

然后不同的hook方法，就可以进行各自的处理了。

**要记住，不论是哪个hooks，都要经历这两个阶段，执行挂载和更新逻辑。hook节点是其基础，区别在于对节点上的`memoizedState`和`queue`做不同的赋值。**



### 总结

每个`hooks`对应一个`hook`对象

1. Hooks是如何保存在组件中的？

   **所有hooks会组成一个链表，挂载到当前fiber节点上，之后的每一次更新，都会根据调用顺序来迭代并取出链表中对应的节点，然后再进行操作。**

2. 为什么不能条件语句/循环语句中使用hook?

   hook需要根据调用顺序来取出链表中的节点，如果少了/多了某个hook调用，就会造成取值错误。



## Hooks方法

### useState&useReducer
我们先回忆一下基本用法。

- **useState**

```js
const [state, setState] = useState(initialState);
```

入参：一个初始值

返回：一个 state，以及更新 state 的函数。

| 参数         |      |      |
| ------------ | ---- | ---- |
| initialState |      |      |
|              |      |      |
|              |      |      |



- **useReducer**

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

入参：一个更新state的逻辑，一个初始值，一个初始逻辑

返回：一个state，以及更新 state 的函数。



尽管后者要比前者看似复杂不少，但二者实际上是同一个方法，`useState`可视为`useReducer`的一个语法糖和简化版，是程序内部帮你赋予了一个`reducer`.

```js
// 更新阶段useState实际上调用的方法
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

```js
// 挂载阶段useState实际上调用的方法
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  // 如上文所述，创建一个节点并link到链表中
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  const queue = (hook.queue = {
    pending: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });
  const dispatch: Dispatch<
    BasicStateAction<S>,
  > = (queue.dispatch = (dispatchAction.bind(
    null,
    currentlyRenderingFiber,
    queue,
  ): any));
  return [hook.memoizedState, dispatch];
}
```

