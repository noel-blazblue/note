## 数据结构

### dispatcher

在`renderWithHooks`中，会根据老`fiber`节点是否存在而决定全局的`hooks`方法是挂载阶段还是更新阶段。

当`FunctionComponent`的`render`执行完毕后，会全局指向`ContextOnlyDispatcher`，也就不再允许`hooks`方法的声明。

```ts
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
  // ...省略
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
  useDebugValue: throwInvalidHookError,
  useDeferredValue: throwInvalidHookError,
  useTransition: throwInvalidHookError,
  useMutableSource: throwInvalidHookError,
  useOpaqueIdentifier: throwInvalidHookError,

  unstable_isNewReconciler: enableNewReconciler,
};
```



### Hook

```ts
export type Hook = {|
  // 最新的state
  memoizedState: any,
  // 通过已处理好的update来计算出的state
  baseState: any,
  // 尚需处理的update，通常是上一轮render中遗留下的优先级过低而暂缓执行的update
  baseQueue: Update<any, any> | null,
  // 当前触发的update
  queue: UpdateQueue<any, any> | null,
  next: Hook | null,
|};
```

不同类型`hook`的`memoizedState`保存不同类型数据，具体如下：

- useState：对于`const [state, updateState] = useState(initialState)`，`memoizedState`保存`state`的值
- useReducer：对于`const [state, dispatch] = useReducer(reducer, {});`，`memoizedState`保存`state`的值
- useEffect：`memoizedState`保存包含`useEffect回调函数`、`依赖项`等的链表数据结构`effect`，你可以在[这里 (opens new window)](https://github.com/acdlite/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1181)看到`effect`的创建过程。`effect`链表同时会保存在`fiber.updateQueue`中
- useRef：对于`useRef(1)`，`memoizedState`保存`{current: 1}`
- useMemo：对于`useMemo(callback, [depA])`，`memoizedState`保存`[callback(), depA]`
- useCallback：对于`useCallback(callback, [depA])`，`memoizedState`保存`[callback, depA]`。与`useMemo`的区别是，`useCallback`保存的是`callback`函数本身，而`useMemo`保存的是`callback`函数的执行结果

有些`hook`是没有`memoizedState`的，比如：

- useContext



### Update

```ts
type UpdateQueue<S, A> = {|
  // 存放当前触发的update
  pending: Update<S, A> | null,
  // 存放dispatchAction.bind()的值
  dispatch: (A => mixed) | null,
  // 上一次render时的Reducer
  lastRenderedReducer: ((S, A) => S) | null,
  // 上一次render时的state
  lastRenderedState: S | null,
|};
```



```ts
type Update<S, A> = {|
  lane: Lane,
  action: A,
  // 触发dispatch时的reducer
  eagerReducer: ((S, A) => S) | null,
  // 触发dispatch时计算好的state
  eagerState: S | null,
  next: Update<S, A>,
  priority?: ReactPriorityLevel,
|};
```

### Effect
```ts
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

## 执行流程

### renderWithHooks

函数组件`render`部分的主函数

1. 把`workInProgress`节点记录到全局变量`currentlyRenderingFiber`，并重置`workInProgress`的一些状态
2. 根据老`fiber`节点以及它的`memoizedState`来判断处在挂载阶段还是更新阶段，并替换为对应的`hooks`全局函数
3. 执行函数组件的`render`，其中包括自己声明在组件里的一系列的`hooks`方法调用。
   1. 在此期间如果发生了更新行为，就重新执行步骤3，直到没有。
4. 当前组件已执行完毕，重置全局变量。
5. 返回`children`

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
  currentlyRenderingFiber = workInProgress;

  workInProgress.memoizedState = null;
  workInProgress.updateQueue = null;
  workInProgress.lanes = NoLanes;

	// 老fiber节点不存在以及未存在hooks调用都视为未挂载
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
    ? HooksDispatcherOnMount
  	: HooksDispatcherOnUpdate;

  // 执行函数组件的render
  let children = Component(props, secondArg);

  // Check if there was a render phase update
  // 在渲染阶段发生的更新，会直接re-render，重新执行。
  if (didScheduleRenderPhaseUpdateDuringThisPass) {
    // Keep rendering in a loop for as long as render phase updates continue to
    // be scheduled. Use a counter to prevent infinite loops.
    let numberOfReRenders: number = 0;
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
  }

  // We can assume the previous dispatcher is always this one, since we set it
  // at the beginning of the render phase and there's no re-entrancy.
  // 函数组件的render已结束，关闭hooks调用接口
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

  // This check uses currentHook so that it works the same in DEV and prod bundles.
  // hookTypesDev could catch more cases (e.g. context) but only in DEV bundles.
  const didRenderTooFewHooks =
    currentHook !== null && currentHook.next !== null;

  // 当前fiber已执行结束，重置这些全局变量
  renderLanes = NoLanes;
  currentlyRenderingFiber = (null: any);

  currentHook = null;
  workInProgressHook = null;

  didScheduleRenderPhaseUpdate = false;

  return children;
}
```



### mountWorkInProgressHook

每执行一次，就使得全局的	`workInProgressHook`指针后移

```js
function mountWorkInProgressHook(): Hook {
  const hook: Hook = {
    memoizedState: null,

    baseState: null,
    baseQueue: null,
    queue: null,

    next: null,
  };

  if (workInProgressHook === null) {
    // This is the first hook in the list
    // 挂载到当前fiber节点的memoizedState中
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```



### updateWorkInProgressHook

每执行一次，就使得全局的	`workInProgressHook`和`currentHook`指针后移

- 迭代老`fiber`节点中的`hooks`链表
- 迭代新`fiber`节点中的`hooks`链表
  - `nextWorkInProgressHook`存在，证明现在处于`re-render`，`hooks`链表在之前已经创建过
  - `nextWorkInProgressHook`不存在，从老`fiber`节点中`clone` `hook`，链接到新`hook`链表尾部

```js
function updateWorkInProgressHook(): Hook {
  // This function is used both for updates and for re-renders triggered by a
  // render phase update. It assumes there is either a current hook we can
  // clone, or a work-in-progress hook from a previous render pass that we can
  // use as a base. When we reach the end of the base list, we must switch to
  // the dispatcher used for mounts.
  // 迭代老fiber节点中的hooks链表
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

  // 迭代新fiber节点中的hooks链表
  let nextWorkInProgressHook: null | Hook;
  if (workInProgressHook === null) {
    // 只有re-render的情况下，新fiber节点中仍然存在hooks链表
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    nextWorkInProgressHook = workInProgressHook.next;
  }

  if (nextWorkInProgressHook !== null) {
    // There's already a work-in-progress. Reuse it.
    // 走入这个分支，是只有在render阶段setState了，导致re-render，这个时候wip的 memoizedState hooks 链表已经创建过了。
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;

    currentHook = nextCurrentHook;
  } else {
    // Clone from the current hook.

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

