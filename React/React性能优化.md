## PureComponent

如果先前的状态和 props 数据与下一个 props 或状态相同，则组件不会重新渲染。

#### 什么是浅层渲染？

在对比先前的 props 和状态与下一个 props 和状态时，浅层比较将检查它们的基元是否有相同的值（例如：1 等于 1 或真等于真），还会检查更复杂的 JavaScript 值（如对象和数组）之间的引用是否相同。

比较基元和对象引用的开销比更新组件视图要低。



在 React 中 PureComponet 的源码为

```js
if (this._compositeType === CompositeTypes.PureClass) {
  shouldUpdate = !shallowEqual(prevProps, nextProps) || ! shallowEqual(inst.state, nextState);
}

```

`shallowEqual` 是如何实现的。

```js
const hasOwnProperty = Object.prototype.hasOwnProperty;

/**
 * is 方法来判断两个值是否是相等的值，为何这么写可以移步 MDN 的文档
 * https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is
 */
function is(x: mixed, y: mixed): boolean {
  if (x === y) {
    return x !== 0 || y !== 0 || 1 / x === 1 / y;
  } else {
    return x !== x && y !== y;
  }
}

function shallowEqual(objA: mixed, objB: mixed): boolean {
  // 首先对基本类型进行比较
  if (is(objA, objB)) {
    return true;
  }

  if (typeof objA !== 'object' || objA === null ||
      typeof objB !== 'object' || objB === null) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);
	
  // 长度不相等直接返回false
  if (keysA.length !== keysB.length) {
    return false;
  }

  // key相等的情况下，再去循环比较
  for (let i = 0; i < keysA.length; i++) {
    if (
      !hasOwnProperty.call(objB, keysA[i]) ||
      !is(objA[keysA[i]], objB[keysA[i]])
    ) {
      return false;
    }
  }

  return true;
}
```



## 使用 React.memo 进行组件记忆

这里与纯组件类似，如果输入 props 相同则跳过组件渲染，从而提升组件性能。

它会记忆上次某个输入 prop 的执行输出并提升应用性能。即使在这些组件中比较也是浅层的。

你还可以为这个组件传递自定义比较逻辑。



## 使用 shouldComponentUpdate 生命周期事件

