hooks以一个单链表的形式保存在[[Fiber]]节点的`memoizedState`属性中

Function Component 不像 Class Component那样可以将私有状态挂载到类实例中并通过对应的key来指向对应的状态，而是每次组件的重新渲染都会使得 Function 重新执行一遍。

Mount 阶段和Update阶段调用的是不同的hooks

```js
// react-reconciler/src/ReactFiberHooks.js
// Mount 阶段Hooks的定义
const HooksDispatcherOnMount: Dispatcher = {
  useEffect: mountEffect,
  useReducer: mountReducer,
  useState: mountState,
 // 其他Hooks
};

// Update阶段Hooks的定义
const HooksDispatcherOnUpdate: Dispatcher = {
  useEffect: updateEffect,
  useReducer: updateReducer,
  useState: updateState,
  // 其他Hooks
};

```

- [[useState]]