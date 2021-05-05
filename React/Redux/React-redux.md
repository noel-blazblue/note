
## react-redux

### Provider

它接受store作为props，然后通过context往下传，这样react中任何组件都可以通过context获取store。

```html
<Provider store={store}>
    <Component />
</Provider>
```



### connect()

生成容器组件，嵌套UI组件，并把数据和逻辑通过`props`传入UI组件

接收四个参数，并返回一个函数：**wrapWithConnect（component）**



**mapStateToProps(state, [ownProps])：**

- **state** ： 接收store的state

- **[ownProps]：** 可选，自定义的props

定义该参数，组件将会监听 Redux store 的变化，在发生改变时重新调用该函数。

通过state参数，获取你所需要的数据，作为对象返回，这个对象会作为props的一部分传入UI组件。



**mapDispatchToProps(dispatch, [ownProps])：**

如果传递的是一个对象（由各个action组成），那么对象中每个属性中的函数都被当做 `Redux action creator ` ，返回新的函数，绑定dispatch和action。合并到props中，调用属性名就可以自动执行dispatch。

如果传递的是一个函数，该函数将接收一个 `dispatch` 函数，然后由你来决定如何返回一个对象，这个对象通过 `dispatch` 函数与 action creator 以某种方式绑定在一起。



**mergeProps(stateProps, dispatchProps, ownProps)：**

可以在其中对 mapStateToProps， mapDispatchToProps的结果进一步处理

如果指定了这个参数，`mapStateToProps()` 与 `mapDispatchToProps()` 的执行结果和组件自身的 `props` 将传入到这个回调函数中。该回调函数返回的对象将作为 props 传递到被包装的组件中。

默认情况下返回 `Object.assign({}, ownProps, stateProps, dispatchProps)` 的结果。



**options：**

一些配置项，例如 pure.当设置为true时，会避免不必要的渲染

pure = true 表示Connect容器组件将在shouldComponentUpdate中对store的state和ownProps进行浅对比，判断是否发生变化，优化性能。为false则不对比。



### wrapWithConnect()

接收一个组件作为参数wrapWithConnect(component)，它内部定义一个新组件Connect(容器组件)并将传入的组件(ui组件)作为Connect的子组件然后return出去。

完整写法：connect(mapStateToProps, mapDispatchToProps, mergeProps, options)(component)



### react-redux 工作流程：

1. Provider组件接受redux的store作为props，然后通过context往下传。

2. connect函数
   1. 执行mapDispatchToProps函数，