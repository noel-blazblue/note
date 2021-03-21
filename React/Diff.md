# Diff 

## 生命周期函数的处理

### 不同的组件类型的处理

 `beginWork` 是如何对 Fiber 进行比对的: 

```js
function beginWork(fiber: Fiber): Fiber | undefined {
  if (fiber.tag === WorkTag.HostComponent) {
    // 宿主节点diff
    diffHostComponent(fiber)
  } else if (fiber.tag === WorkTag.ClassComponent) {
    // 类组件节点diff
    diffClassComponent(fiber)
  } else if (fiber.tag === WorkTag.FunctionComponent) {
    // 函数组件节点diff
    diffFunctionalComponent(fiber)
  } else {
    // ... 其他类型节点，省略
  }
}
```

 宿主节点比对: 

```js
function diffHostComponent(fiber: Fiber) {
  // 新增节点
  if (fiber.stateNode == null) {
    fiber.stateNode = createHostComponent(fiber)
  } else {
    updateHostComponent(fiber)
  }

  const newChildren = fiber.pendingProps.children;

  // 比对子节点
  diffChildren(fiber, newChildren);
}
```

 类组件节点对比:

```js
function diffClassComponent(fiber: Fiber) {
  // 创建组件实例
  if (fiber.stateNode == null) {
    fiber.stateNode = createInstance(fiber);
  }

  if (fiber.hasMounted) {
    // 调用更新前生命周期钩子
    applybeforeUpdateHooks(fiber)
  } else {
    // 调用挂载前生命周期钩子
    applybeforeMountHooks(fiber)
  }

  // 渲染新节点
  const newChildren = fiber.stateNode.render();
  // 比对子节点
  diffChildren(fiber, newChildren);

  fiber.memoizedState = fiber.stateNode.state
}
```

### 调用class组件声明周期
1. **[[Class声明周期#getDerivedStateFromProps|getDerivedStateFromProps]]**
获取最新的state

2. **判断是否要更新函数，`shouldComponentUpdate` 或 `PureComponent`。**
 如果存在 `shouldComponentUpdate` 函数则调用，并根据其返回的boolean值来决定是否更新。
否则判断当前组件是否继承自`PureComponent` ，是则浅比较前后的`props`和`state`得出结果。

3. **如果需要更新，则处理 `componentDidUpdate` 及 `getSnapshotBeforeUpdate` 函数**
调和阶段并不会调用以上两个函数，而是打上 tag 以便将来使用位运算知晓是否需要使用它们。`effectTag` 这个属性在整个更新的流程中都是至关重要的一员，凡是涉及到**函数的延迟调用、devTool 的处理、DOM 的更新**都可能会使用到它。

```js
if (typeof instance.componentDidUpdate === 'function') {
    workInProgress.effectTag |= Update;
}
if (typeof instance.getSnapshotBeforeUpdate === 'function') {
    workInProgress.effectTag |= Snapshot;
}
```

##  reconcileChildren
https://react.jokcy.me/book/flow/reconcile-children/array.html

### 第一轮遍历：复用和当前节点索引一致的老节点
 
 -   新旧节点都为文本节点，可以直接复用，因为文本节点不需要 key
 -   其他类型节点一律通过判断 key 是否相同来复用或创建节点（可能类型不同但 key 相同）

一旦出现不能复用的情况就跳出遍历.

```js
for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
  // 找到下一个老的子节点
  nextOldFiber = oldFiber.sibling;
  // 通过 oldFiber 和 newChildren[newIdx] 判断是否可以复用
  // 并给复用出来的节点的 return 属性赋值 returnFiber
  const newFiber = reuse(
    returnFiber,
    oldFiber,
    newChildren[newIdx]
  );
  // 不能复用，跳出
  if (newFiber === null) {
    break;
  }
}

```
 
 ### 第二轮遍历：三种情况
 
第一轮遍历完之后，会出现三种情况
1. **新节点已遍历完**
	这个时候删除剩余老节点即可，设置 `effectTag` 为 `Deletion`
2. **老节点已遍历完**
	把剩余的新节点全部创建完毕即可
3. **新老节点都有剩余**
	那就进入第三轮遍历
 
 ### 第三轮遍历：把老节点存入map，通过key来进行复用
 
 ```js
 // 节点的 key 作为 map 的 key
// 如果节点不存在 key，那么 index 为 key
const map = {
    1: {},
    2: {}
}
```
 
 在遍历的过程中会寻找新节点的 key 是否存在于这个 `map` 中，存在即把这个节点移动位置，并给他的 `effectTag` 赋值为 `Placement`。 然后把 `key` 从 `map` 中删掉。

如果不存在，则创建一个新的。
 
 此轮遍历结束后，就把还存在于 `map` 中的所有老节点删除。
 
 子节点比对: 

```js
function diffChildren(fiber: Fiber, newChildren: React.ReactNode) {
  let oldFiber = fiber.alternate ? fiber.alternate.child : null;
  // 全新节点，直接挂载
  if (oldFiber == null) {
    mountChildFibers(fiber, newChildren)
    return
  }

  let index = 0;
  let newFiber = null;
  // 新子节点
  const elements = extraElements(newChildren)

  // 比对子元素
  while (index < elements.length || oldFiber != null) {
    const prevFiber = newFiber;
    const element = elements[index]
    const sameType = isSameType(element, oldFiber)
    if (sameType) {
      newFiber = cloneFiber(oldFiber, element)
      // 更新关系
      newFiber.alternate = oldFiber
      // 打上Tag
      newFiber.effectTag = UPDATE
      newFiber.return = fiber
    }

    // 新节点
    if (element && !sameType) {
      newFiber = createFiber(element)
      newFiber.effectTag = PLACEMENT
      newFiber.return = fiber
    }

    // 删除旧节点
    if (oldFiber && !sameType) {
      oldFiber.effectTag = DELETION;
      oldFiber.nextEffect = fiber.nextEffect
      fiber.nextEffect = oldFiber
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index == 0) {
      fiber.child = newFiber;
    } else if (prevFiber && element) {
      prevFiber.sibling = newFiber;
    }

    index++
  }
}
```



 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecce3162b355?imageslim) 

 对于需要变更的节点，都打上了'标签'。 在提交阶段，React 就会将这些打上标签的节点应用变更。 



## 副作用的收集和提交

 接下来就是将所有打了 Effect 标记的节点串联起来，这个可以在`completeWork`中做, 例如: 

```js
function completeWork(fiber) {
  const parent = fiber.return

  // 到达顶端
  if (parent == null || fiber === topWork) {
    pendingCommit = fiber
    return
  }

  if (fiber.effectTag != null) {
    if (parent.nextEffect) {
      parent.nextEffect.nextEffect = fiber
    } else {
      parent.nextEffect = fiber
    }
  } else if (fiber.nextEffect) {
    parent.nextEffect = fiber.nextEffect
  }
}
```



 最后了，将所有副作用提交: 

```js
function commitAllWork(fiber) {
  let next = fiber
  while(next) {
    if (fiber.effectTag) {
      // 提交，偷一下懒，这里就不展开了
      commitWork(fiber)
    }
    next = fiber.nextEffect
  }

  // 清理现场
  pendingCommit = nextUnitOfWork = topWork = null
}
```

