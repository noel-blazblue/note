## 前言

本文主要解释 hooks 这部分的源码，让大家在用hooks开发业务中能有一个清晰的认识。对于fiber架构和任务调度，只会略微提及。不过我建议大家最好先了解一下，尽管不懂这部分流程，仍然可以看懂hook。

社区上的一些优秀文章：

## 基础架构

### Fiber 简介

fiber 是 React 架构中的一个基本工作单元，里面会存放组件的实例、状态、结构关系、副作用。react中的大部分逻辑都依托于这个结构来实现。

```js
interface Fiber {
  /**
   * ⚛️ 节点的类型信息
   */
  // 标记 Fiber 类型, 例如函数组件、类组件、宿主组件
  tag: WorkTag,
  // 节点元素类型, 是具体的类组件、函数组件、宿主组件(字符串)
  type: any,

  /**
   * ⚛️ 结构信息
   */ 
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,
  // 子节点的唯一键, 即我们渲染列表传入的key属性
  key: null | string,
	// 指向旧树中的节点
  alternate: Fiber | null,

  /**
   * ⚛️ 节点的状态
   */
  // 节点实例(状态)：
  //        对于宿主组件，这里保存宿主组件的实例, 例如DOM节点。
  //        对于类组件来说，这里保存类组件的实例
  //        对于函数组件说，这里为空，因为函数组件没有实例
  stateNode: any,
  // 新的、待处理的props
  pendingProps: any,
  // 上一次渲染的props
  memoizedProps: any, 
  // 上一次渲染的组件状态
  memoizedState: any,
	// 如果是class，setState时会把action enqueue进去。
	// 如果是hooks，这个地方会存放effect对象
  updateQueue: UpdateQueue<any> | null, 

  /**
   * ⚛️ 副作用
   */
  // 当前节点的副作用类型，例如节点更新、删除、移动
  effectTag: SideEffectTag,
  // 和节点关系一样，React 同样使用链表来将所有有副作用的Fiber连接起来
  nextEffect: Fiber | null,
  firstEffect: Fiber | null, // 第一个需要进行 DOM 操作的节点
  lastEffect: Fiber | null, // 最后一个需要进行 DOM 操作的节点，同时也可用于恢复任务
}
```

对于class组件，stateNode 属性会存放它的实例。这样就能通过stateNode.render 等方法来获取jsx、调用生命周期，以及更新组件的状态。

但是函数组件是没有实例的，如果在里面写hooks方法，要怎样才能拿到这些信息并加以操作呢？

**答案是通过链表**

每一个hook方法，实际上都会创建一个节点，在内部保存类似于fiber结构这样的信息。然后链接起来，组成一个单链表。最终挂载到当前的 fiber 节点上。

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
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```