## 前言

本文主要解释 hooks 这部分的源码，让大家在用hooks开发业务中能有一个清晰的认识。对于fiber架构和任务调度，只会略微提及。不过我建议大家最好先了解一下，尽管不懂这部分流程，仍然可以看懂hook。



社区上的一些优秀文章：

## 基础架构

### Fiber 简介
fiber 是 React 架构中的一个基本工作单元，里面会存放组件的实例、状态、结构关系、副作用。react中的大部分逻辑都依托于这个结构来实现。

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
	// 指向旧树中的节点
  alternate: Fiber | null,

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
  memoizedProps: any, 
  // 上一次渲染的组件状态
  memoizedState: any,
	// 如果是class，setState时会把action enqueue进去。
	// 如果是hooks，这个地方会存放effect对象
  updateQueue: UpdateQueue<any> | null, 

  /**
   * ⚛️ 副作用
   */
  // 当前节点的副作用类型，例如节点更新、删除、移动
  effectTag: SideEffectTag,
  // 和节点关系一样，React 同样使用链表来将所有有副作用的Fiber连接起来
  nextEffect: Fiber | null,
  firstEffect: Fiber | null, // 第一个需要进行 DOM 操作的节点
  lastEffect: Fiber | null, // 最后一个需要进行 DOM 操作的节点，同时也可用于恢复任务
}
```

对于class组件，stateNode 属性会存放它的实例。这样就能通过实例来访问组件的 jsx、生命周期方法，组件的状态。

伪代码：

```js
const instance = fiber.stateNode
const children = instance.render()
instance.componentDidMount()
```

但是函数组件是没有实例的，如果在里面写hooks方法，要怎样才能拿到这些数据并加以操作呢？

**答案是通过链表**

每一个hook方法在初次调用时，都会创建一个节点，在其中保存一些状态，并把他们之间互相链接起来，组成一个链表。最终挂载到当前fiber节点上，然后就可以通过fiber上的引用，来获取到hooks。

这个过程会在hooks的挂载阶段去执行。

### 挂载阶段

众所周知，每一次`render`，函数组件都会重新执行一遍。因此里面的hooks方法也会跟着执行。在它第一次执行的时候，我们称之为**挂载阶段**，往后就称之为**更新阶段**。

事实上，在react内部，这两个阶段调用的是不同的hooks方法。

```js
// 挂载阶段
const HooksDispatcherOnMount: Dispatcher = {
  readContext,

  useCallback: mountCallback,
  useContext: readContext,
  useEffect: mountEffect,
  useImperativeHandle: mountImperativeHandle,
  useLayoutEffect: mountLayoutEffect,
  useMemo: mountMemo,
  useReducer: mountReducer,
  useRef: mountRef,
  useState: mountState,
  useDebugValue: mountDebugValue,
  useDeferredValue: mountDeferredValue,
  useTransition: mountTransition,
  useMutableSource: mountMutableSource,
  useOpaqueIdentifier: mountOpaqueIdentifier,

  unstable_isNewReconciler: enableNewReconciler,
};

// 更新阶段
const HooksDispatcherOnUpdate: Dispatcher = {
  readContext,

  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect,
  useImperativeHandle: updateImperativeHandle,
  useLayoutEffect: updateLayoutEffect,
  useMemo: updateMemo,
  useReducer: updateReducer,
  useRef: updateRef,
  useState: updateState,
  useDebugValue: updateDebugValue,
  useDeferredValue: updateDeferredValue,
  useTransition: updateTransition,
  useMutableSource: updateMutableSource,
  useOpaqueIdentifier: updateOpaqueIdentifier,

  unstable_isNewReconciler: enableNewReconciler,
};
```

主要是通过下面这个方法来区分，挂载时，`current`参数会传null，更新时，`current`参数会传当前fiber节点。

```js
function renderWithHooks (current) {
	ReactCurrentDispatcher.current =
		current === null || current.memoizedState === null
			? HooksDispatcherOnMount
			: HooksDispatcherOnUpdate;
}
```

挂载阶段的hooks，需要进行一个挂载逻辑，用`mountWorkInProgressHook`这个方法，来创建一个节点，并link到链表的尾部，链表的头部则赋值到当前的fiber节点的`memoizedState`属性中。

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

节点挂载上后，会返回节点本身，然后不同的hooks方法就可以对这个节点进行不同的处理，比如`useState`会把`initialState`放置到节点的`memoizeState`中。这个等到下文讲api时再具体述说。

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

到了更新阶段，与挂载时对应，hooks所要做的就是从链表中取出对应的节点。

```js
function updateWorkInProgressHook(): Hook {
  let nextCurrentHook: null | Hook;
  if (currentHook === null) {
    const current = currentlyRenderingFiber.alternate;
    if (current !== null) {
      nextCurrentHook = current.memoizedState;
    } else {
      nextCurrentHook = null;
    }
  } else {
    nextCurrentHook = currentHook.next;
  }

  let nextWorkInProgressHook: null | Hook;
  if (workInProgressHook === null) {
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    nextWorkInProgressHook = workInProgressHook.next;
  }

  if (nextWorkInProgressHook !== null) {
    // There's already a work-in-progress. Reuse it.
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;

    currentHook = nextCurrentHook;
  } else {
    // Clone from the current hook.

    invariant(
      nextCurrentHook !== null,
      'Rendered more hooks than during the previous render.',
    );
    currentHook = nextCurrentHook;

    const newHook: Hook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      baseQueue: currentHook.baseQueue,
      queue: currentHook.queue,

      next: null,
    };

    if (workInProgressHook === null) {
      // This is the first hook in the list.
      currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
    } else {
      // Append to the end of the list.
      workInProgressHook = workInProgressHook.next = newHook;
    }
  }
  return workInProgressHook;
}
```