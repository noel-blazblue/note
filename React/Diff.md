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
1. **[[Class生命周期#getDerivedStateFromProps|getDerivedStateFromProps]]**
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

### 单节点diff

老`fiber`节点存在的情况下遍历

- 老`fiber`节点和新的`element`节点`key`相同
  - 其`type`相同，复用老`fiber`节点，并标记删除其兄弟节点
  - 其`type`不同，删除老`fiber`节点及其兄弟节点，并退出循环
- `key`不同，删除老`fiber`节点，指针移到其兄弟节点，继续遍历。看看是否有同级别能复用的节点。

如果老`fiber`节点为null，证明无可复用节点，则直接新建

```js
function reconcileSingleElement(
	returnFiber: Fiber,
	currentFirstChild: Fiber | null,
	element: ReactElement,
  lanes: Lanes,
): Fiber {
  const key = element.key;
  let child = currentFirstChild;
  while (child !== null) {
    // TODO: If key === null and child.key === null, then this only applies to
    // the first item in the list.
    if (child.key === key) {
      const elementType = element.type;
      if (elementType === REACT_FRAGMENT_TYPE) {
        if (child.tag === Fragment) {
          // 因为最新的是个单节点，所以删除老fiber中无用的兄弟节点
          deleteRemainingChildren(returnFiber, child.sibling);
          const existing = useFiber(child, element.props.children);
          existing.return = returnFiber;
          if (__DEV__) {
            existing._debugSource = element._source;
            existing._debugOwner = element._owner;
          }
          return existing;
        }
      } else {
        if (
          child.elementType === elementType ||
          // Keep this check inline so it only runs on the false path:
          (__DEV__
           ? isCompatibleFamilyForHotReloading(child, element)
           : false) ||
          // Lazy types should reconcile their resolved type.
          // We need to do this after the Hot Reloading check above,
          // because hot reloading has different semantics than prod because
          // it doesn't resuspend. So we can't let the call below suspend.
          (enableLazyElements &&
           typeof elementType === 'object' &&
           elementType !== null &&
           elementType.$$typeof === REACT_LAZY_TYPE &&
           resolveLazy(elementType) === child.type)
        ) {
          // 因为最新的element是个单节点，所以删除老fiber中无用的兄弟节点
          deleteRemainingChildren(returnFiber, child.sibling);
          const existing = useFiber(child, element.props);
          existing.ref = coerceRef(returnFiber, child, element);
          existing.return = returnFiber;
          if (__DEV__) {
            existing._debugSource = element._source;
            existing._debugOwner = element._owner;
          }
          return existing;
        }
      }
      // Didn't match.
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }

  // child为null，则老fiber节点已经遍历完，把剩下的element节点新建fiber节点
  if (element.type === REACT_FRAGMENT_TYPE) {
    const created = createFiberFromFragment(
      element.props.children,
      returnFiber.mode,
      lanes,
      element.key,
    );
    created.return = returnFiber;
    return created;
  } else {
    const created = createFiberFromElement(element, returnFiber.mode, lanes);
    created.ref = coerceRef(returnFiber, currentFirstChild, element);
    created.return = returnFiber;
    return created;
  }
}
```



### 多节点diff

#### 第一轮遍历：复用和当前节点索引一致的老节点

 -   新旧节点都为文本节点，可以直接复用，因为文本节点不需要 `key`
 -   其他类型节点一律通过 `key`来判断，
      -   `key`相同，`elementType` 相同，则复用老的`fiber`节点
      -   `key`相同，`elementType`不相同，则新建`fiber`节点，但并不终止循环
 -   `key`不相同，则跳出循环。

```js
// 新的fiber链表中的第一个节点
let resultingFirstChild: Fiber | null = null;
// 前一个新的fiber节点
let previousNewFiber: Fiber | null = null;

let oldFiber = currentFirstChild;
// 上个未发生移动的节点索引
let lastPlacedIndex = 0;
let newIdx = 0;
let nextOldFiber = null;
for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
  if (oldFiber.index > newIdx) {
    nextOldFiber = oldFiber;
    oldFiber = null;
  } else {
    nextOldFiber = oldFiber.sibling;
  }
  // 有两种情况，一种key相同，elementType相同，则复用老fiber节点。
  // 一种key相同，elementType不同，则新建节点。
  const newFiber = updateSlot(
    returnFiber,
    oldFiber,
    newChildren[newIdx],
    lanes,
  );
  if (newFiber === null) {
    // key不同，跳出循环
    // TODO: This breaks on empty slots like null children. That's
    // unfortunate because it triggers the slow path all the time. We need
    // a better way to communicate whether this was a miss or null,
    // boolean, undefined, etc.
    if (oldFiber === null) {
      oldFiber = nextOldFiber;
    }
    break;
  }
  if (shouldTrackSideEffects) {
    if (oldFiber && newFiber.alternate === null) {
      // We matched the slot, but we didn't reuse the existing fiber, so we
      // need to delete the existing child.
      // key相同，elementType不同，不能复用，则删除老fiber节点
      deleteChild(returnFiber, oldFiber);
    }
  }
  // 如果复用了老fiber节点，则返回其节点的索引
  lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
  // 将构建好的fiber节点以sibling相互连接起来
  if (previousNewFiber === null) {
    // TODO: Move out of the loop. This only happens for the first run.
    resultingFirstChild = newFiber;
  } else {
    // TODO: Defer siblings if we're not at the right index for this slot.
    // I.e. if we had null values before, then we want to defer this
    // for each null value. However, we also don't want to call updateSlot
    // with the previous one.
    previousNewFiber.sibling = newFiber;
  }
  previousNewFiber = newFiber;
  oldFiber = nextOldFiber;
}
```

##### updateSlot

- key相同，调用对应的节点type复用函数
- key不相同，则返回null

```js
function updateSlot(
	returnFiber: Fiber,
	oldFiber: Fiber | null,
	newChild: any,
	lanes: Lanes,
): Fiber | null {
  // Update the fiber if the keys match, otherwise return null.

  const key = oldFiber !== null ? oldFiber.key : null;

  if (typeof newChild === 'string' || typeof newChild === 'number') {
    // Text nodes don't have keys. If the previous node is implicitly keyed
    // we can continue to replace it without aborting even if it is not a text
    // node.
    if (key !== null) {
      return null;
    }
    // 文本节点，直接复用或者新建
    return updateTextNode(returnFiber, oldFiber, '' + newChild, lanes);
  }

  if (typeof newChild === 'object' && newChild !== null) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE: {
        if (newChild.key === key) {
          // key相同，复用节点。如果elementType不同，则新建fiber节点。
          return updateElement(returnFiber, oldFiber, newChild, lanes);
        } else {
          // key不同，无法复用
          return null;
        }
      }
      case REACT_PORTAL_TYPE: {
        if (newChild.key === key) {
          return updatePortal(returnFiber, oldFiber, newChild, lanes);
        } else {
          return null;
        }
      }
      case REACT_LAZY_TYPE: {
        if (enableLazyElements) {
          const payload = newChild._payload;
          const init = newChild._init;
          return updateSlot(returnFiber, oldFiber, init(payload), lanes);
        }
      }
    }

    if (isArray(newChild) || getIteratorFn(newChild)) {
      if (key !== null) {
        return null;
      }

      return updateFragment(returnFiber, oldFiber, newChild, lanes, null);
    }
  }

  return null;
}
```

##### updateElement

- 老节点存在，判断其type是否相同，相同则复用，否则新建
- 老节点不存在，直接新建

```js
function updateElement(
	returnFiber: Fiber,
	current: Fiber | null,
  element: ReactElement,
	lanes: Lanes,
): Fiber {
  const elementType = element.type;
  if (elementType === REACT_FRAGMENT_TYPE) {
    return updateFragment(
      returnFiber,
      current,
      element.props.children,
      lanes,
      element.key,
    );
  }
  if (current !== null) {
    if (
      // element节点相同，则复用
      current.elementType === elementType ||
      // Keep this check inline so it only runs on the false path:
      (__DEV__
       ? isCompatibleFamilyForHotReloading(current, element)
       : false) ||
      // Lazy types should reconcile their resolved type.
      // We need to do this after the Hot Reloading check above,
      // because hot reloading has different semantics than prod because
      // it doesn't resuspend. So we can't let the call below suspend.
      (enableLazyElements &&
       typeof elementType === 'object' &&
       elementType !== null &&
       elementType.$$typeof === REACT_LAZY_TYPE &&
       resolveLazy(elementType) === current.type)
    ) {
      // Move based on index
      const existing = useFiber(current, element.props);
      existing.ref = coerceRef(returnFiber, current, element);
      existing.return = returnFiber;
      if (__DEV__) {
        existing._debugSource = element._source;
        existing._debugOwner = element._owner;
      }
      return existing;
    }
  }
  // 否则新建fiber节点
  // Insert
  const created = createFiberFromElement(element, returnFiber.mode, lanes);
  created.ref = coerceRef(returnFiber, current, element);
  created.return = returnFiber;
  return created;
}
```



 #### 第二轮遍历：三种情况

第一轮遍历完之后，会出现三种情况
1. **新节点已遍历完**
	这个时候删除剩余老节点即可，设置 `effectTag` 为 `Deletion`
2. **老节点已遍历完**
	把剩余的新节点全部创建完毕即可
3. **新老节点都有剩余**
	那就进入第三轮遍历

```js
if (newIdx === newChildren.length) {
  // We've reached the end of the new children. We can delete the rest.
  // newChildren已遍历完，删除剩余的老的fiber节点
  deleteRemainingChildren(returnFiber, oldFiber);
  return resultingFirstChild;
}

if (oldFiber === null) {
  // If we don't have any more existing children we can choose a fast path
  // since the rest will all be insertions.
  // 老fiber节点已遍历完，新的element节点仍有剩余，则把剩余的全部新建
  for (; newIdx < newChildren.length; newIdx++) {
    const newFiber = createChild(returnFiber, newChildren[newIdx], lanes);
    if (newFiber === null) {
      continue;
    }
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
    if (previousNewFiber === null) {
      // TODO: Move out of the loop. This only happens for the first run.
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
  }
  return resultingFirstChild;
}
```



 #### 第三轮遍历：把老节点存入map，通过key来进行复用

1. 把所有剩余的老`fiber`节点存储到`map`中，`key`为`fiber`的`key`或者`index`
2. 遍历剩余的新`element`节点，如果通过`key`在`map`中找到了老节点，那就复用，并在`map`中删除那个节点。否则新建。
3.  把`map`中剩余的老`fiber`节点执行删除操作。

 ```js
// Add all children to a key map for quick lookups.
// 将剩余的老fiber节点存储到map中，map的key为：如果有key则用key，否则用index。
const existingChildren = mapRemainingChildren(returnFiber, oldFiber);

// Keep scanning and use the map to restore deleted items as moves.
for (; newIdx < newChildren.length; newIdx++) {
  const newFiber = updateFromMap(
    existingChildren,
    returnFiber,
    newIdx,
    newChildren[newIdx]=
    lanes,
  );
  if (newFiber !== null) {
    if (shouldTrackSideEffects) {
      if (newFiber.alternate !== null) {
        // The new fiber is a work in progress, but if there exists a
        // current, that means that we reused the fiber. We need to delete
        // it from the child list so that we don't add it to the deletion
        // list.
        // 匹配到老fiber节点，则把那个节点从map中删除
        existingChildren.delete(
          newFiber.key === null ? newIdx : newFiber.key,
        );
      }
    }
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
  }
}

if (shouldTrackSideEffects) {
  // Any existing children that weren't consumed above were deleted. We need
  // to add them to the deletion list.
  // 删除为匹配到的老fiber节点
  existingChildren.forEach(child => deleteChild(returnFiber, child));
}
 ```

 





 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecce3162b355?imageslim) 

 对于需要变更的节点，都打上了'标签'。 在提交阶段，React 就会将这些打上标签的节点应用变更。 

#### 节点的移动

- 如果老`fiber`节点存在
  - 其`index`小于新的`fiber`	节点，证明这个节点向后移动了，会标记为`Placement`
  - 其`index`大于等于新的`fiber`节点的索引，就返回其`index`
- 老`fiber`节点不存在，证明这个节点是新插入的，也标记为`Placement`

```js
/**
   * 如果老fiber节点的索引小于新fiber节点的索引，证明老fiber节点发生了移动，会标记为Placement
   * 如果current不存在，证明是新插入的节点，也会标记为Placement
   * @param {*} newFiber 
   * @param {number} lastPlacedIndex 上个未移动节点的索引
   * @param {number} newIndex 
   * @returns 
   */
function placeChild(
	newFiber: Fiber,
	lastPlacedIndex: number,
	newIndex: number,
): number {
  newFiber.index = newIndex;
  if (!shouldTrackSideEffects) {
    // Noop.
    return lastPlacedIndex;
  }
  const current = newFiber.alternate;
  if (current !== null) {
    const oldIndex = current.index;
    if (oldIndex < lastPlacedIndex) {
      // This is a move.
      // 老fiber节点的索引小于当前复用节点的索引，证明老节点向后移动了。
      newFiber.flags |= Placement;
      return lastPlacedIndex;
    } else {
      // This item can stay in place.
      return oldIndex;
    }
  } else {
    // This is an insertion.
    newFiber.flags |= Placement;
    return lastPlacedIndex;
  }
}
```



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

