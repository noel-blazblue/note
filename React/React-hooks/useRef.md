
useRef是一个可变数据，可以直接修改.current属性，一般用来保存一些数据或者dom节点引用。修改current并不会一起组件的渲染

内部实现原理也是创建一个hook节点，把ref对象挂载到memoizedState属性上，并push到hooks链表中，最终也会挂载到fiber节点上。但区别在于往后的每一次更新，都只是取出这个hook节点的memoizedState，也就是他的ref对象。因此，在整个生命周期，这个hook节点都不会再改变。

## Ref 声明

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



## 组件更新ref

### render阶段

只有`hostComponent`(`Dom`组件)和`classComponent`会根据挂载的`ref`而在更新时标记需要更新`ref`



```js
function markRef(current: Fiber | null, workInProgress: Fiber) {
  const ref = workInProgress.ref;
  if (
    (current === null && ref !== null) ||
    (current !== null && current.ref !== ref)
  ) {
    // Schedule a Ref effect
    workInProgress.flags |= Ref;
  }
}
```



`classComponent`:

在这之前是处理`state`，之后是`render`

```js
function finishClassComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  shouldUpdate: boolean,
  hasContext: boolean,
  renderLanes: Lanes,
) {
 //...
 // Refs should update even if shouldComponentUpdate returns false
  markRef(current, workInProgress);
 //...
}
```



`hostComponent`：

```js
function updateHostComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
) {
  // ...
  markRef(current, workInProgress);
  reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  //...
}
```



### Commit 阶段

在`mutation`阶段会去分离老`fiber`节点中的`ref`，然后再在`layout`阶段去赋值新`fiber`节点的`ref`

#### 清空

```js
function commitMutationEffects(
  root: FiberRoot,
  renderPriorityLevel: ReactPriorityLevel,
) {
  // ...
    // 有ref的tag，则需要执行更新
    if (flags & Ref) {
      const current = nextEffect.alternate;
      if (current !== null) {
        commitDetachRef(current);
      }
      if (enableScopeAPI) {
        // TODO: This is a temporary solution that allowed us to transition away
        // from React Flare on www.
        if (nextEffect.tag === ScopeComponent) {
          commitAttachRef(nextEffect);
        }
      }
    }
  // ...
```



```js
function commitDetachRef(current: Fiber) {
  const currentRef = current.ref;
  if (currentRef !== null) {
    if (typeof currentRef === 'function') {
      if (
        enableProfilerTimer &&
        enableProfilerCommitHooks &&
        current.mode & ProfileMode
      ) {
        try {
          startLayoutEffectTimer();
          currentRef(null);
        } finally {
          recordLayoutEffectDuration(current);
        }
      } else {
        currentRef(null);
      }
    } else {
      currentRef.current = null;
    }
  }
}
```



#### 赋值

```js
function commitLayoutEffects(root: FiberRoot, committedLanes: Lanes) {
  // ...
  if (enableScopeAPI) {
    // TODO: This is a temporary solution that allowed us to transition away
    // from React Flare on www.
    if (flags & Ref && nextEffect.tag !== ScopeComponent) {
      commitAttachRef(nextEffect);
    }
  } else {
    if (flags & Ref) {
      commitAttachRef(nextEffect);
    }
  }
  // ...
}
```



```js
function commitAttachRef(finishedWork: Fiber) {
  const ref = finishedWork.ref;
  if (ref !== null) {
    const instance = finishedWork.stateNode;
    let instanceToUse;
    switch (finishedWork.tag) {
      case HostComponent:
        instanceToUse = getPublicInstance(instance);
        break;
      default:
        instanceToUse = instance;
    }
    // Moved outside to ensure DCE works with this flag
    if (enableScopeAPI && finishedWork.tag === ScopeComponent) {
      instanceToUse = instance;
    }
    if (typeof ref === 'function') {
      if (
        enableProfilerTimer &&
        enableProfilerCommitHooks &&
        finishedWork.mode & ProfileMode
      ) {
        try {
          startLayoutEffectTimer();
          ref(instanceToUse);
        } finally {
          recordLayoutEffectDuration(finishedWork);
        }
      } else {
        ref(instanceToUse);
      }
    } else {
      ref.current = instanceToUse;
    }
  }
}
```



## 用法

### 控制按钮

由于hook function 实际上会调用非常多次，在某一次调用时，他里面的方法获取到的是旧的state值。所以如果用state来控制按钮是否允许点击的话，在频繁点击时是控制不到的。

因此可以用ref来作为保存和控制，他不用通过更新调度就能直接修改数据，随时就能获取到最新的值。

### 防抖节流

由于传入的fn引用值会在每一次更新的时候都会改变，所以如果单纯的用useEffect去监听的话会造成死循环调用的问题，因此需要用ref来作为保存。

```js
const useTimeout = (fn, time) => {
    const callback = useRef(fn);
    callback.current = fn;
    useEffect(
        () => {
            const tick = setTimeout(callback.current);
            return () => clearTimeout(tick);
        },
        [time]
    );
};
```

### 配合 `useImperativeHandle`使子组件暴露出去属性和方法

