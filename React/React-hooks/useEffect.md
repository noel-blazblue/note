

### effect

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



### pushEffect

创建一个`effect`节点并添加到`fiber`节点的`updateQueue`的`lastEffect`中，如果这个`fiber`	没有`updateQueue`，则帮它创建一个头节点。

```js
function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag,
    create,
    destroy,
    deps,
    // Circular
    next: (null: any),
  };
  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  if (componentUpdateQueue === null) {
    // 首个副作用，创建一个UpdateQueue并把effect节点push进去。
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    // 链接到当前fiber节点的updateQueue的lastEffect中
    const lastEffect = componentUpdateQueue.lastEffect;
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



### mountEffect

1. 创建一个hook节点，并push到`work-in-progress` hooks链表中。
2. 创建一个Effect节点，并`push`到[[Fiber]]节点的`updateQueue`的`lastEffect`上。
3. 把那个Effect节点赋值到Hooks节点的momeizeState属性中。

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
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  currentlyRenderingFiber.flags |= fiberFlags;
  // 创建一个effect对象并添加到memoizedState中
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    undefined,
    nextDeps,
  );
}
```

### updateEffect

1. 取出当前的hooks节点，并把`nextWorkInProgressHook`指针后移。
2. 从上一轮`render`产生的`hook`节点中取出memoizedState，也就是effect对象。判断前后的`deps`是否发生改变
	
	- 未改变，则跳过更新`memoizedState`，并`push`这个`effect`到`fiber`节点的`updateQueue`中
	- 改变，则把这个`effect`更新到`memoizedState`中，并且`tag`标记为`HookHasEffect`，表示需要执行此副作用。

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
        // 如果上一轮effect和当前effect的依赖项未发生变化，就无需更新hook的memoizedState
        // clone一个effect对象链接到updateQueue中，但是tag不添加hasEffect
        pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
    }
  }

  currentlyRenderingFiber.flags |= fiberFlags;

  hook.memoizedState = pushEffect(
    // 标记此effect对象的tag为HookHasEffect，表示有副作用，需要执行。
    HookHasEffect | hookFlags,
    create,
    destroy,
    nextDeps,
  );
}
```


### 执行Effect

1.  `before mutation阶段`在`scheduleCallback`中调度`flushPassiveEffects`
2. `layout`阶段结束后， `scheduleCallback`触发`flushPassiveEffects`，`flushPassiveEffects`调用`commitHookEffectListUnmount`和`commitHookEffectListMount`



#### commitBeforeMutationEffects

在这个阶段，发起`useEffect`的调度。

```js
function commitBeforeMutationEffects() {
  while (nextEffect !== null) {
		// .....
    if ((flags & Passive) !== NoFlags) {
      // If there are passive effects, schedule a callback to flush at
      // the earliest opportunity.
      if (!rootDoesHavePassiveEffects) {
        rootDoesHavePassiveEffects = true;
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



#### flushPassiveEffectsImpl

先调用卸载回调，再调用传入的回调。

```js
function flushPassiveEffectsImpl() {
  if (rootWithPendingPassiveEffects === null) {
    return false;
  }

  commitPassiveUnmountEffects(root.current);
  commitPassiveMountEffects(root, root.current);

  flushSyncCallbackQueue();

  return true;
}
```



#### commitHookEffectListUnmount

```js
function commitHookEffectListUnmount(flags: HookFlags, finishedWork: Fiber) {
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
          safelyCallDestroy(finishedWork, destroy);
        }
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}
```



#### commitHookEffectListMount

```js
function commitHookEffectListMount(tag: number, finishedWork: Fiber) {
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
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

