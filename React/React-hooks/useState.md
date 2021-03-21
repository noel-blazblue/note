
## useState

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
在mount阶段会进行一些初始化的操作，把组件内的所有hooks依次创建一个节点，并组成一个链表，然后挂载到当前的[[Fiber]]节点的`momeizeState`属性中。

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

#### mountWorkInProgressHook
创建一个hooks节点，并push到`workInProgressHook`hooks链表中。
```js
// react-reconciler/src/ReactFiberHooks.js
function mountWorkInProgressHook(): Hook {
  const hook: Hook = {
    memoizedState: null,
    baseState: null,
    queue: null,
    baseUpdate: null,
    next: null,
  };
  if (workInProgressHook === null) {
    // 当前workInProgressHook链表为空的话，
    // 将当前Hook作为第一个Hook
    firstWorkInProgressHook = workInProgressHook = hook;
  } else {
    // 否则将当前Hook添加到Hook链表的末尾
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}

```

### update 阶段
每个hooks会以`queue`这个属性来存放一个更新链表，每次dispatch，就会把他的action存放到链表的尾部。
在执行更新时，会遍历queue更新队列，执行action逻辑来更新state。然后返回最新的state和dispatch方法。

![](https://user-gold-cdn.xitu.io/2020/3/3/170a091cef83acd4?imageslim)

#### dispatchAction
```js
// react-reconciler/src/ReactFiberHooks.js
// 去除特殊情况和与fiber相关的逻辑
function dispatchAction(fiber,queue,action,) {
    const update = {
      action,
      next: null,
    };
    // 将update对象添加到循环链表中
    const last = queue.last;
    if (last === null) {
      // 链表为空，将当前更新作为第一个，并保持循环
      update.next = update;
    } else {
      const first = last.next;
      if (first !== null) {
        // 在最新的update对象后面插入新的update对象
        update.next = first;
      }
      last.next = update;
    }
    // 将表头保持在最新的update对象上
    queue.last = update;
   // 进行调度工作
    scheduleWork();
}

```

#### updateReducer
```js
// react-reconciler/src/ReactFiberHooks.js
// 去掉与fiber有关的逻辑

function updateReducer(reducer,initialArg,init) {
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;

  // 拿到更新列表的表头
  const last = queue.last;

  // 获取最早的那个update对象
  first = last !== null ? last.next : null;

  if (first !== null) {
    let newState;
    let update = first;
    do {
      // 执行每一次更新，去更新状态
      const action = update.action;
      newState = reducer(newState, action);
      update = update.next;
    } while (update !== null && update !== first);

    hook.memoizedState = newState;
  }
  const dispatch = queue.dispatch;
  // 返回最新的状态和修改状态的方法
  return [hook.memoizedState, dispatch];
}

```