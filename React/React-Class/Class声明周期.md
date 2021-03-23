https://react.jokcy.me/book/flow/begin-work/update-class-component.html

### `getDerivedStateFromProps`
`getDerivedStateFromProps` 会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 `null` 则不更新任何内容。

### `getSnapshotBeforeUpdate`
`getSnapshotBeforeUpdate()` 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给 `componentDidUpdate()`。

### componentDidUpdate
`componentDidUpdate()` 会在更新后会被立即调用。首次渲染不会执行此方法。
如果`showComponentUpdate`返回false，则不会调用这个函数