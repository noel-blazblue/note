
## useState
useState是一个让函数组件中拥有自己的内部状态的一个方法，

### 数据结构
函数组件内的所有hooks会组成一个单链表

```js
// react-reconciler/src/ReactFiberHooks.js
export type Hook = {
  memoizedState: any,
  baseState: any,
  baseUpdate: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null, // 更新队列
  next: Hook | null,  // 指向下一个Hook
};
```

### mount 阶段
在mount阶段会进行一些初始化的操作，把组件内的所有hooks依次创建一个节点，并组成一个链表，然后挂载到当前的[[Fiber]]节点的`memoizeState`属性中。

对于class组件而言这个属性就是他自身的state，对于hook 组件而言，则是他里面的所有hooks方法。

在mount阶段，每当我们调用Hooks方法，比如useState，mountState就会调用**mountWorkInProgressHook** 来创建一个Hook节点，并把它添加到Hooks链表上。以 `workInProgressHook` 这个全局变量进行储存。
![](https://user-gold-cdn.xitu.io/2020/3/3/170a0d46dd647c0c?imageslim)

#### mountState
```js
// react-reconciler/src/ReactFiberHooks.js
function mountState (initialState) {
  // 获取当前的Hook节点，同时将当前Hook添加到Hook链表中
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  // 声明一个链表来存放更新
  const queue = (hook.queue = {
    last: null,
    dispatch: null,
    lastRenderedReducer,
    lastRenderedState,
  });
  // 返回一个dispatch方法用来修改状态，并将此次更新添加update链表中
  const dispatch = (queue.dispatch = (dispatchAction.bind(
    null,
    currentlyRenderingFiber,
    queue,
  )));
  // 返回当前状态和修改状态的方法 
  return [hook.memoizedState, dispatch];
}

```



### update 阶段
每个hooks会以`queue`这个属性来存放一个更新链表，每次dispatch，就会把他的action存放到链表的尾部。
在执行更新时，会遍历queue更新队列，执行action逻辑来更新state。然后返回最新的state和dispatch方法。

![](https://user-gold-cdn.xitu.io/2020/3/3/170a091cef83acd4?imageslim)

#### dispatchAction
```js
function dispatchAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {

  const eventTime = requestEventTime();
  const lane = requestUpdateLane(fiber);

  const update: Update<S, A> = {
    lane,
    action,
    eagerReducer: null,
    eagerState: null,
    next: (null: any),
  };

  // Append the update to the end of the list.
  // 把更新对象链接到更新队列中，组成单循环链表
  // pending指向队列的最后一个
  const pending = queue.pending;
  if (pending === null) {
    // This is the first update. Create a circular list.
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
    // currentlyRenderingFiber仍然存在，证明这是在本 FunctionComponent的render中发生的更新。
    // 需要标记，然后会在本次FunctionComponent的render中结束后立即re-render
    // This is a render phase update. Stash it in a lazily-created map of
    // queue -> linked list of updates. After this render pass, we'll restart
    // and apply the stashed updates on top of the work-in-progress hook.
    didScheduleRenderPhaseUpdateDuringThisPass = didScheduleRenderPhaseUpdate = true;
  } else {
    if (
      fiber.lanes === NoLanes &&
      (alternate === null || alternate.lanes === NoLanes)
    ) {
      // The queue is currently empty, which means we can eagerly compute the
      // next state before entering the render phase. If the new state is the
      // same as the current state, we may be able to bail out entirely.
      // fiber.lanes === NoLanes，说明此前未发生更新，本次是第一个
      // 我们可以直接计算触发更新的state值，如果与当前值相同，则跳过更新。
      // 如果值不同，则保存在eagerState，下次render时可以直接使用，而无需再计算。
      const lastRenderedReducer = queue.lastRenderedReducer;
      if (lastRenderedReducer !== null) {
        let prevDispatcher;
        try {
          // 上次render时的state
          const currentState: S = (queue.lastRenderedState: any);
          // 通过reducer计算好的state
          const eagerState = lastRenderedReducer(currentState, action);
          // Stash the eagerly computed state, and the reducer used to compute
          // it, on the update object. If the reducer hasn't changed by the
          // time we enter the render phase, then the eager state can be used
          // without calling the reducer again.
          // 保存计算出来的state
          update.eagerReducer = lastRenderedReducer;
          update.eagerState = eagerState;
          if (is(eagerState, currentState)) {
            // 如果计算好的state和当前的state相同，则不进行更新调度
            // 但是update对象仍然会在组件下次重新渲染时去执行
            // Fast path. We can bail out without scheduling React to re-render.
            // It's still possible that we'll need to rebase this update later,
            // if the component re-renders for a different reason and by that
            // time the reducer has changed.
            return;
          }
        } catch (error) {
          // Suppress the error. It will throw again in the render phase.
        } finally {
          if (__DEV__) {
            ReactCurrentDispatcher.current = prevDispatcher;
          }
        }
      }
    }
    scheduleUpdateOnFiber(fiber, lane, eventTime);
  }

  if (enableSchedulingProfiler) {
    markStateUpdateScheduled(fiber, lane);
  }
}
```

#### updateReducer

1. 取出或`clone`对应的`hook`节点
2. 更新`lastRenderedReducer`
3. `baseQueue`若存在，证明存在上次`render`未处理完的`update`，则把`pendingQueue`添加链接进去，一齐执行
4. 迭代`baseQueue` ，计算新的`state`。
   - 若`update`优先级过低，则跳过执行，并添加入`newBaseUpdate`，留待下次`render`执行
   - 优先级足够
     - 若`update`的`reducer`未改变，则可以用其计算好的`state`
     - 以其`reducer`执行`action`，计算新的`state`
5. 若不存在`newBaseQueueLast`，证明所有的`update`都已经执行完毕，更新`baseState`
6. 更新`hook`节点的`memoizedState`，`baseState`,`baseQueue`

```js
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;

  queue.lastRenderedReducer = reducer;

  const current: Hook = (currentHook: any);

  // The last rebase update that is NOT part of the base state.
  // 上次render未能处理完的update
  let baseQueue = current.baseQueue;

  // The last pending update that hasn't been processed yet.
  // 把pendingQueue链接到baseQueue的尾部，一起迭代执行
  const pendingQueue = queue.pending;
  if (pendingQueue !== null) {
    // We have new updates that haven't been processed yet.
    // We'll add them to the base 品；‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’ queue.
    if (baseQueue !== null) {
      // Merge the pending queue and the base queue.
      const baseFirst = baseQueue.next;
      const pendingFirst = pendingQueue.next;
      baseQueue.next = pendingFirst;
      pendingQueue.next = baseFirst;
    }
    current.baseQueue = baseQueue = pendingQueue;
    queue.pending = null;
  }

  if (baseQueue !== null) {
    // We have a queue to process.
    const first = baseQueue.next;
    let newState = current.baseState;

    let newBaseState = null;
    let newBaseQueueFirst = null;
    let newBaseQueueLast = null;
    let update = first;
    // 遍历队列，计算state
    do {
      const updateLane = update.lane;
      if (!isSubsetOfLanes(renderLanes, updateLane)) {
        // 优先级过低，跳过更新。
        // Priority is insufficient. Skip this update. If this is the first
        // skipped update, the previous update/state is the new base
        // update/state.
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
          newBaseState = newState;
        } else {
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }
        // Update the remaining priority in the queue.
        // TODO: Don't need to accumulate this. Instead, we can remove
        // renderLanes from the original lanes.
        currentlyRenderingFiber.lanes = mergeLanes(
          currentlyRenderingFiber.lanes,
          updateLane,
        );
        markSkippedUpdateLanes(updateLane);
      } else {
        // This update does have sufficient priority.

        if (newBaseQueueLast !== null) {
          // newBaseQueueLast 存在证明此前有update被跳过
          // 因为update互相之间可能存在依赖，所以从那个被跳过的update起，所有后面的update都链接到newBaseQueueLast中
          // 在下次render一齐执行
          const clone: Update<S, A> = {
            // This update is going to be committed so we never want uncommit
            // it. Using NoLane works because 0 is a subset of all bitmasks, so
            // this will never be skipped by the check above.
            lane: NoLane,
            action: update.action,
            eagerReducer: update.eagerReducer,
            eagerState: update.eagerState,
            next: (null: any),
          };
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }

        // Process this update.
        if (update.eagerReducer === reducer) {
          // reducer 未改变，直接使用其计算出来的值
          // If this update was processed eagerly, and its reducer matches the
          // current reducer, we can use the eagerly computed state.
          newState = ((update.eagerState: any): S);
        } else {
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

    // Mark that the fiber performed work, but only if the new state is
    // different from the current state.
    if (!is(newState, hook.memoizedState)) {
      markWorkInProgressReceivedUpdate();
    }

    hook.memoizedState = newState;
    hook.baseState = newBaseState;
    hook.baseQueue = newBaseQueueLast;

    queue.lastRenderedState = newState;
  }

  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```