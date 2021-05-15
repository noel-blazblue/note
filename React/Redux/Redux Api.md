## createStore(reducer, [preloadedState], enhancer)
创建一个 Redux [store](https://www.redux.org.cn/docs/api/Store.html) 来以存放应用中所有的 state。  
应用中应有且仅有一个 store。

### 参数

1.  `reducer` _(Function)_: 接收两个参数，分别是当前的 state 树和要处理的 [action](https://www.redux.org.cn/docs/Glossary.html#action)，返回新的 [state 树](https://www.redux.org.cn/docs/Glossary.html#state)。
    
2.  \[`preloadedState`\] _(any)_: 初始时的 state。 在同构应用中，你可以决定是否把服务端传来的 state 水合（hydrate）后传给它，或者从之前保存的用户会话中恢复一个传给它。如果你使用 [`combineReducers`](https://www.redux.org.cn/docs/api/combineReducers.html) 创建 `reducer`，它必须是一个普通对象，与传入的 keys 保持同样的结构。否则，你可以自由传入任何 `reducer` 可理解的内容。
    
3.  `enhancer` _(Function)_: Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator。这与 middleware 相似，它也允许你通过复合函数改变 store 接口。


### 返回值

([_`Store`_](https://www.redux.org.cn/docs/api/Store.html)): 保存了应用所有 state 的对象。改变 state 的惟一方法是 [dispatch](https://www.redux.org.cn/docs/api/Store.html#dispatch) action。你也可以 [subscribe 监听](https://www.redux.org.cn/docs/api/Store.html#subscribe) state 的变化，然后更新 UI。

## Store
Store 就是用来维持应用所有的 [state 树](https://www.redux.org.cn/docs/Glossary.html#state) 的一个对象。 改变 store 内 state 的惟一途径是对它 dispatch 一个 [action](https://www.redux.org.cn/docs/Glossary.html#action)。

Store 不是类。它只是有几个方法的对象。 要创建它，只需要把根部的 [reducing 函数](https://www.redux.org.cn/docs/Glossary.html#reducer) 传递给 [`createStore`](https://www.redux.org.cn/docs/api/createStore.html)。

### getState()
返回应用当前的 state 树。  
它与 store 的最后一个 reducer 返回值相同。

### dispatch(action)

分发 action。这是触发 state 变化的惟一途径。

会使用当前 [`getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 的结果和传入的 `action` 以同步方式的调用 store 的 reduce 函数。返回值会被作为下一个 state。从现在开始，这就成为了 [`getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 的返回值，同时变化监听器(change listener)会被触发。

**参数：**

1.  `action` (_Object_†): 描述应用变化的普通对象。Action 是把数据传入 store 的惟一途径，所以任何数据，无论来自 UI 事件，网络回调或者是其它资源如 WebSockets，最终都应该以 action 的形式被 dispatch。按照约定，action 具有 `type` 字段来表示它的类型。type 也可被定义为常量或者是从其它模块引入。最好使用字符串，而不是 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 作为 action，因为字符串是可以被序列化的。除了 `type` 字段外，action 对象的结构完全取决于你。参照 [Flux 标准 Action](https://github.com/acdlite/flux-standard-action) 获取如何组织 action 的建议。

**返回值：**
(Object†): 要 dispatch 的 action。

† 使用 [`createStore`](https://www.redux.org.cn/docs/api/createStore.html) 创建的 “纯正” store 只支持普通对象类型的 action，而且会立即传到 reducer 来执行。

但是，如果你用 [`applyMiddleware`](https://www.redux.org.cn/docs/api/applyMiddleware.html) 来套住 [`createStore`](https://www.redux.org.cn/docs/api/createStore.html) 时，middleware 可以修改 action 的执行，并支持执行 dispatch [intent（意图）](https://www.redux.org.cn/docs/Glossary.html#intent)。Intent 一般是异步操作如 Promise、Observable 或者 Thunk。


### subscribe(listener)
添加一个变化监听器。每当 dispatch action 的时候就会执行，state 树中的一部分可能已经变化。

#### 参数

1.  `listener` (_Function_): 每当 dispatch action 的时候都会执行的回调。state 树中的一部分可能已经变化。你可以在回调函数里调用 [`getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 来拿到当前 state。store 的 reducer 应该是纯函数，因此你可能需要对 state 树中的引用做深度比较来确定它的值是否有变化。

#### 返回值

(_Function_): 一个可以解绑变化监听器的函数。

```js
function select(state) {
  return state.some.deep.property
}

let currentValue
function handleChange() {
  let previousValue = currentValue
  currentValue = select(store.getState())

  if (previousValue !== currentValue) {
    console.log('Some deep nested property changed from', previousValue, 'to', currentValue)
  }
}

let unsubscribe = store.subscribe(handleChange)
unsubscribe()
```


### replaceReducer(nextReducer)
替换 store 当前用来计算 state 的 reducer。

这是一个高级 API。只有在你需要实现代码分隔，而且需要立即加载一些 reducer 的时候才可能会用到它。在实现 Redux 热加载机制的时候也可能会用到。

#### 参数

1.  `reducer` (_Function_) store 会使用的下一个 reducer。


## combineReducers(reducers)
`combineReducers` 辅助函数的作用是，把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用`createStore`方法。

合并后的 reducer 可以调用各个子 reducer，并把它们返回的结果合并成一个 state 对象。 **由 `combineReducers()` 返回的 state 对象，会将传入的每个 reducer 返回的 state 按其传递给 `combineReducers()` 时对应的 key 进行命名**。

```js
rootReducer = combineReducers({potato: potatoReducer, tomato: tomatoReducer})
// rootReducer 将返回如下的 state 对象
{
  potato: {
    // ... potatoes, 和一些其他由 potatoReducer 管理的 state 对象 ... 
  },
  tomato: {
    // ... tomatoes, 和一些其他由 tomatoReducer 管理的 state 对象，比如说 sauce 属性 ...
  }
}
```

### 参数

1.  `reducers` (_Object_): 一个对象，它的值（value）对应不同的 reducer 函数，这些 reducer 函数后面会被合并成一个。下面会介绍传入 reducer 函数需要满足的规则。

### 返回值

(_Function_)：一个调用 `reducers` 对象里所有 reducer 的 reducer，并且构造一个与 `reducers` 对象结构相同的 state 对象。


## applyMiddleware(...middlewares)
Middleware 可以让你包装 store 的 [`dispatch`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法来达到你想要的目的。同时， middleware 还拥有“可组合”这一关键特性。多个 middleware 可以被组合到一起使用，形成 middleware 链。

Middleware 并不需要和 [`createStore`](https://www.redux.org.cn/docs/api/createStore.html) 绑在一起使用，也不是 Redux 架构的基础组成部分，但它带来的益处让我们认为有必要在 Redux 核心中包含对它的支持。因此，虽然不同的 middleware 可能在易用性和用法上有所不同，它仍被作为扩展 [`dispatch`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 的唯一标准的方式。

### 参数

-   `...middlewares` (_arguments_): 遵循 Redux _middleware API_ 的函数。每个 middleware 接受 [`Store`](https://www.redux.org.cn/docs/api/Store.html) 的 [`dispatch`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 和 [`getState`](https://www.redux.org.cn/docs/api/Store.html#getState) 函数作为命名参数，并返回一个函数。该函数会被传入 被称为 `next` 的下一个 middleware 的 dispatch 方法，并返回一个接收 action 的新函数，这个函数可以直接调用 `next(action)`，或者在其他需要的时刻调用，甚至根本不去调用它。调用链中最后一个 middleware 会接受真实的 store 的 [`dispatch`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法作为 `next` 参数，并借此结束调用链。所以，middleware 的函数签名是 `({ getState, dispatch }) => next => action`。


## bindActionCreators(actionCreators, dispatch)
把一个 value 为不同 action creator 的对象，转成拥有同名 key 的对象。同时使用 `dispatch`对每个 action creator 进行包装，在调用这个 action creator 时，在内部会直接进行`dispatch`。

### 参数

1.  `actionCreators` (_Function_ or _Object_): 一个 [action creator](https://www.redux.org.cn/docs/Glossary.html#action-creator)，或者一个 value 是 action creator 的对象。
    
2.  `dispatch` (_Function_): 一个由 [`Store`](https://www.redux.org.cn/docs/api/Store.html) 实例提供的 [`dispatch`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 函数。
    

### 返回值

(_Function_ or _Object_): 一个与原对象类似的对象，只不过这个对象的 value 都是会直接 dispatch 原 action creator 返回的结果的函数。如果传入一个单独的函数作为 `actionCreators`，那么返回的结果也是一个单独的函数。


## compose(...functions)
从右到左来组合多个函数。

#### 参数

1.  (_arguments_): 需要合成的多个函数。预计每个函数都接收一个参数。它的返回值将作为一个参数提供给它左边的函数，以此类推。例外是最右边的参数可以接受多个参数，因为它将为由此产生的函数提供签名。（译者注：`compose(funcA, funcB, funcC)` 形象为 `compose(funcA(funcB(funcC())))`）

#### 返回值

(_Function_): 从右到左把接收到的函数合成后的最终函数。

```js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import DevTools from './containers/DevTools'
import reducer from '../reducers/index'

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunk),
    DevTools.instrument()
  )
)
```