## fiber架构
`React Fiber`可以理解为：

`React`内部实现的一套状态更新机制。支持任务不同`优先级`，可中断与恢复，并且恢复后可以复用之前的`中间状态`。

### 模仿函数调用栈

React Fiber 也被称为虚拟栈帧(Virtual Stack Frame), 你可以拿它和函数调用栈类比一下, 两者结构非常像:

|          | 函数调用栈   | Fiber            |
| -------- | ------------ | ---------------- |
| 基本单位 | 函数         | Virtual DOM 节点 |
| 输入     | 函数参数     | Props            |
| 本地状态 | 本地变量     | State            |
| 输出     | 函数返回值   | React Element    |
| 下级     | 嵌套函数调用 | 子节点(child)    |
| 上级引用 | 返回地址     | 父节点(return)   |



 它其实就是一个深度优先的遍历： 

```js
/**
 * @params fiber 当前需要处理的节点
 * @params topWork 本次更新的根节点
 */
function performUnitOfWork(fiber: Fiber, topWork: Fiber) {
  // 对该节点进行处理
  beginWork(fiber);

  // 如果存在子节点，那么下一个待处理的就是子节点
  if (fiber.child) {
    return fiber.child;
  }

  // 没有子节点了，上溯查找兄弟节点
  let temp = fiber;
  while (temp) {
    completeWork(temp);

    // 到顶层节点了, 退出
    if (temp === topWork) {
      break
    }

    // 找到，下一个要处理的就是兄弟节点
    if (temp.sibling) {
      return temp.sibling;
    }

    // 没有, 继续上溯
    temp = temp.return;
  }
}

```

 ![img](https://user-gold-cdn.xitu.io/2019/10/21/16deecca7850a24d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 







### 数据结构

每个fiber节点：

```js
interface Fiber {
  // 1. Instance 类型信息
  // 标记 Fiber 类型, 例如函数组件、类组件、宿主组件
  tag: WorkTag,
  // class、function的构造函数，或者dom组件的标签名。
  type: any,
	// class、function的构造函数，或者dom组件的标签名。
	elementType: any,
	// DOM节点 | Class实例
	// 函数组件则为空
  stateNode: any,
	
	key: key,
	ref: ref,
	
  // 2. Fiber 结构信息
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,
	// 指向旧树中的节点
  alternate: Fiber | null,

  // 3. Fiber节点的状态
  // 本次更新的props
  pendingProps: any,
  // 上一次渲染的props
  memoizedProps: any, 
  // 如果是class组件，会保存上一次渲染的state
  // 如果是hooks组件，会保存所有hooks组成的链表
  memoizedState: any,
	// 如果是class，将保存setState产生的update链表
	// 如果是hooks，这个地方会存放effect链表
  updateQueue: UpdateQueue<any> | null, 

  // 4. 副作用
  // 当前节点的副作用类型，例如节点更新、删除、移动
  flags: Flags,
  // 和节点关系一样，React 同样使用链表来将所有有副作用的Fiber连接起来
  nextEffect: Fiber | null,
  firstEffect: Fiber | null, // 第一个需要进行 DOM 操作的节点
  lastEffect: Fiber | null, // 最后一个需要进行 DOM 操作的节点，同时也可用于恢复任务
	
	// 5. 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

}
```


