useEffect 也是会创建一个hooks对象，并push到`work-in-progress` hooks链表中。

## mountEffect

1. 创建一个hook节点，并push到`work-in-progress` hooks链表中。
2. 创建一个Effect节点，并push到`componentUpdateQueue`Effect链表中。这个链表会在之后赋值到[[Fiber]]节点的updateQueue属性上。
3. 把那个Effect节点赋值到Hooks节点的momeizeState属性中。
	useState中momeizeState属性是存放state值，在useEffect中是存放自己的Effect对象。
4. 在组件渲染完成之后才会调用updateQueue中的所有方法。
	
	![](https://user-gold-cdn.xitu.io/2020/3/3/170a091e5c2e0641?imageslim)

```js
// react-reconciler/src/ReactFiberHooks.js
// 简化去掉特殊逻辑

function mountEffect( create,deps,) {
  return mountEffectImpl(
    create,
    deps,
  );
}

function mountEffectImpl(fiberEffectTag, hookEffectTag, create, deps) {
  // 创建一个Hook节点，并把当前节点添加到Hooks链表中
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  // 将当前effect保存到Hook节点的memoizedState属性上，
  // 以及添加到fiberNode的updateQueue上
  hook.memoizedState = pushEffect(hookEffectTag, create, undefined, nextDeps);
}

function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag,
    create,
    destroy,
    deps,
    next: (null: any),
  };
  // componentUpdateQueue 会被挂载到fiberNode的updateQueue上
  if (componentUpdateQueue === null) {
    // 如果当前Queue为空，将当前effect作为第一个节点
    componentUpdateQueue = createFunctionComponentUpdateQueue();
   // 保持循环
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    // 否则，添加到当前的Queue链表中
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

## updateEffect

1. 取出当前的hooks节点，并把`nextWorkInProgressHook`指针后移。
	`nextWorkInProgressHook`专门为了指向下一个要执行的hook，通过这个方式可以遍历hooks链表，来不断的执行。
2. 从fiber中取出该hook节点的memoizedState，也就是effect对象。然后执行他的`destory`销毁函数，也就是useEffect中返回的方法。
	每个effect在更新之前，都会执行上一轮effect的销毁方法。然后再执行更新。
3. 判断他的deps有没有变化，如果没有，就打上NoHookEffect的tag，在提交阶段就不会执行他的更新。
4. 新创建Effect节点，push到componentUpdateQueue中。

```js
// react-reconciler/src/ReactFiberHooks.js
// 简化去掉特殊逻辑

function updateEffect(create,deps){
  return updateEffectImpl(
    create,
    deps,
  );
}

function updateEffectImpl(fiberEffectTag, hookEffectTag, create, deps){
  // 获取当前Hook节点
  const hook = updateWorkInProgressHook();
  // 依赖 
  const nextDeps = deps === undefined ? null : deps;
 // 清除函数
  let destroy = undefined;

  if (currentHook !== null) {
    // 拿到前一次渲染该Hook节点的effect
    const prevEffect = currentHook.memoizedState;
    destroy = prevEffect.destroy;
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps;
      // 对比deps依赖
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        // 如果依赖没有变化，就会打上NoHookEffect tag，在commit阶段会跳过此
        // effect的执行
        pushEffect(NoHookEffect, create, destroy, nextDeps);
        return;
      }
    }
  }
  hook.memoizedState = pushEffect(hookEffectTag, create, destroy, nextDeps);
}
```


## commitHookEffectList
- 提交阶段会遍历Fiber节点的updateQueue
- 如果effect.tag等于NoHookEffect，也就是依赖项没更新的情况下，会跳过执行
- 卸载阶段，执行销毁函数
- mount阶段，执行effect的回调，并把返回的卸载方法赋值给effect.destroy属性中。

```js
function commitHookEffectList(unmountTag,mountTag,finishedWork) {
  const updateQueue = finishedWork.updateQueue;
  let lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & unmountTag) !== NoHookEffect) {
        // Unmount 阶段执行tag !== NoHookEffect的effect的清除函数 （如果有的话）
        const destroy = effect.destroy;
        effect.destroy = undefined;
        if (destroy !== undefined) {
          destroy();
        }
      }
      if ((effect.tag & mountTag) !== NoHookEffect) {
        // Mount 阶段执行所有tag !== NoHookEffect的effect.create，
        // 我们的清除函数（如果有）会被返回给destroy属性，一遍unmount执行
        const create = effect.create;
        effect.destroy = create();
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}

```