https://react.jokcy.me/book/flow/begin-work/update-class-component.html

### getDerivedStateFromProps

参数：`(nextProps, prevState)`

state`挂载或更新后都会调用，在`render`方法之前。返回一个`state`会和之前的`state`合并，返回`null`则不进行任何操作。

**挂载时：**

```js
function mountClassInstance(
  workInProgress: Fiber,
  ctor: any,
  newProps: any,
  renderLanes: Lanes,
): void {
  // ....
  const getDerivedStateFromProps = ctor.getDerivedStateFromProps;
  if (typeof getDerivedStateFromProps === 'function') {
    // 调用DerivedStateFromProp方法并直接修改memoizedState
    applyDerivedStateFromProps(
      workInProgress,
      ctor,
      getDerivedStateFromProps,
      newProps,
    );
    instance.state = workInProgress.memoizedState;
  }
  // ....
}
```

**更新时：**

如果`props | state | context  `未发生改变，则跳过执行。

```js
if (
  unresolvedOldProps === unresolvedNewProps &&
  oldState === newState &&
  !hasContextChanged() &&
  !checkHasForceUpdateAfterProcessing()
) {
  // If an update was already in progress, we should schedule an Update
  // effect even though we're bailing out, so that cWU/cDU are called.
  if (typeof instance.componentDidUpdate === 'function') {
    if (
      unresolvedOldProps !== current.memoizedProps ||
      oldState !== current.memoizedState
    ) {
      workInProgress.flags |= Update;
    }
  }
  if (typeof instance.getSnapshotBeforeUpdate === 'function') {
    if (
      unresolvedOldProps !== current.memoizedProps ||
      oldState !== current.memoizedState
    ) {
      workInProgress.flags |= Snapshot;
    }
  }
  return false;
}

if (typeof getDerivedStateFromProps === 'function') {
  applyDerivedStateFromProps(
    workInProgress,
    ctor,
    getDerivedStateFromProps,
    newProps,
  );
  newState = workInProgress.memoizedState;
}
```

```js
export function applyDerivedStateFromProps(
  workInProgress: Fiber,
  ctor: any,
  getDerivedStateFromProps: (props: any, state: any) => any,
  nextProps: any,
) {
  const prevState = workInProgress.memoizedState;

  const partialState = getDerivedStateFromProps(nextProps, prevState);

  // Merge the partial state and the previous state.
  const memoizedState =
    partialState === null || partialState === undefined
      ? prevState
      : Object.assign({}, prevState, partialState);
  workInProgress.memoizedState = memoizedState;

  // Once the update queue is empty, persist the derived state onto the
  // base state.
  if (workInProgress.lanes === NoLanes) {
    // Queue is always non-null for classes
    const updateQueue: UpdateQueue<any> = (workInProgress.updateQueue: any);
    updateQueue.baseState = memoizedState;
  }
}
```



### shouldComponentUpdate

参数：`(newProps, newState, nextContext)`

在`getDerivedStateFromProps`之后调用，返回一个`false`，则不进行之后的`render`，但是`state`仍然会变化

```js
function checkShouldComponentUpdate(
  workInProgress,
  ctor,
  oldProps,
  newProps,
  oldState,
  newState,
  nextContext,
) {
  const instance = workInProgress.stateNode;
  if (typeof instance.shouldComponentUpdate === 'function') {
    const shouldUpdate = instance.shouldComponentUpdate(
      newProps,
      newState,
      nextContext,
    );

    return shouldUpdate;
  }

  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  }

  return true;
}
```




### getSnapshotBeforeUpdate
`getSnapshotBeforeUpdate()` 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给 `componentDidUpdate()`。

### componentDidUpdate

参数：`(prevProps, prevState, snapshot)`

`componentDidUpdate()` 会在更新后会被立即调用。首次渲染不会执行此方法。
如果`showComponentUpdate`返回false，则不会调用这个函数

## 不安全的生命周期钩子

只要有`getDerivedStateFromProps`或`getSnapshotBeforeUpdate`存在，这些不安全的生命周期钩子都不会调用。

### componentWillReceiveProps

更新阶段，在处理`state`之前，在没有getDerivedStateFromProps和getSnapshotBeforeUpdate的情况下，并且props或context的引用发生了改变，就会调用这个钩子。

```js
function updateClassInstance(
  current: Fiber,
  workInProgress: Fiber,
  ctor: any,
  newProps: any,
  renderLanes: Lanes,
): boolean {
  // ...
	if (
    !hasNewLifecycles &&
    (typeof instance.UNSAFE_componentWillReceiveProps === 'function' ||
      typeof instance.componentWillReceiveProps === 'function')
  ) {
    if (
      unresolvedOldProps !== unresolvedNewProps ||
      oldContext !== nextContext
    ) {
      // 在没有getDerivedStateFromProps和getSnapshotBeforeUpdate的情况下
      // 并且props或context发生了改变，就会调用这个钩子
      callComponentWillReceiveProps(
        workInProgress,
        instance,
        newProps,
        nextContext,
      );
    }
  }
  // ...
}
```



### componentWillUpdate

参数：`(newProps, newState, nextContext)`

更新阶段，在确认要更新之后调用。

```js
if (shouldUpdate) {
  // In order to support react-lifecycles-compat polyfilled components,
  // Unsafe lifecycles should not be invoked for components using the new APIs.
  if (
    !hasNewLifecycles &&
    (typeof instance.UNSAFE_componentWillUpdate === 'function' ||
     typeof instance.componentWillUpdate === 'function')
  ) {
    if (typeof instance.componentWillUpdate === 'function') {
      instance.componentWillUpdate(newProps, newState, nextContext);
    }
    if (typeof instance.UNSAFE_componentWillUpdate === 'function') {
      instance.UNSAFE_componentWillUpdate(newProps, newState, nextContext);
    }
  }
  if (typeof instance.componentDidUpdate === 'function') {
    workInProgress.flags |= Update;
  }
  if (typeof instance.getSnapshotBeforeUpdate === 'function') {
    workInProgress.flags |= Snapshot;
  }
}
```



### componentWillMount
挂载阶段，在`getDerivedStateFromProps`之后，`render`之前执行。如果在这个钩子发生了状态更新，那就会直接处理`state`，而不再额外进行调度。
```js
function mountClassInstance(
  workInProgress: Fiber,
  ctor: any,
  newProps: any,
  renderLanes: Lanes,
): void {
  // ....
  // In order to support react-lifecycles-compat polyfilled components,
  // Unsafe lifecycles should not be invoked for components using the new APIs.
  if (
    typeof ctor.getDerivedStateFromProps !== 'function' &&
    typeof instance.getSnapshotBeforeUpdate !== 'function' &&
    (typeof instance.UNSAFE_componentWillMount === 'function' ||
      typeof instance.componentWillMount === 'function')
  ) {
    callComponentWillMount(workInProgress, instance);
    // If we had additional state updates during this life-cycle, let's
    // process them now.
    // 如果在挂载前的生命周期发生了setState，那此时就再对更新队列进行处理，去更新state。
    processUpdateQueue(workInProgress, newProps, instance, renderLanes);
    instance.state = workInProgress.memoizedState;
  }
  // ....
}
```