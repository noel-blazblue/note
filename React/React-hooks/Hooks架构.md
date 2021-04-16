



### renderWithHooks

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
  // 函数render已结束，关闭hooks调用接口
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

  // This check uses currentHook so that it works the same in DEV and prod bundles.
  // hookTypesDev could catch more cases (e.g. context) but only in DEV bundles.
  const didRenderTooFewHooks =
    currentHook !== null && currentHook.next !== null;

  // 当前fiber的render已执行结束，重置这些全局变量
  renderLanes = NoLanes;
  currentlyRenderingFiber = (null: any);

  currentHook = null;
  workInProgressHook = null;

  didScheduleRenderPhaseUpdate = false;

  return children;
}
```

