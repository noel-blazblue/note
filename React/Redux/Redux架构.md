![img](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)

**整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。**



### Store

存放数据，整个应用只有一个store

```js
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```

它有以下职责：
-   维持应用的 state；
-   提供 [`getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 方法获取 state；
-   提供 [`dispatch(action)`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法更新 state；
-   通过 [`subscribe(listener)`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 注册监听器;
-   通过 [`subscribe(listener)`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 返回的函数注销监听器。



### state

`Store`对象包含所有数据，当前时刻的 State，可以通过`store.getState()`拿到。一个 State 对应一个 View。只要 State 相同，View 就相同。

```js
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
```

```js
console.log(store.getState())
```

state是只读的，唯一改变state的方法就是调用action



### action

描述当前发生的事情，表示 State 应该要发生变化了。改变 State 的唯一办法，就是使用 Action。

```js
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

创建`action`的函数：
```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

```js
dispatch(addTodo(text))
```

#### 异步action
通过使用`redux-thunk`这个 middleware，可以`dispatch`一个函数，在这个函数内可以进行一些异步操作。

它接收两个参数:`(dispatch, getState)`

它可以有以下行为：
- 允许`promise`链式调用，进行一些副作用行为
- 允许在函数内部中进行多个`dispatch`
- 允许`dispatch`另一个`thunk`
- 但是在最终，必须`dispatch`一个`action`对象，回归到同步

需要引入：
```js
import thunkMiddleware from 'redux-thunk'
import { createLogger } from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // 允许我们 dispatch() 函数
    loggerMiddleware // 一个很便捷的 middleware，用来打印 action 日志
  )
)
```

异步`action`:

```js
import fetch from 'cross-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'
export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))
    return fetch(`http://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {

  // 注意这个函数也接收了 getState() 方法
  // 它让你选择接下来 dispatch 什么。

  // 当缓存的值是可用时，
  // 减少网络请求很有用。

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // 在 thunk 里 dispatch 另一个 thunk！
      return dispatch(fetchPosts(subreddit))
    } else {
      // 告诉调用代码不需要再等待。
      return Promise.resolve()
    }
  }
}
```

调用：

```js
store
  .dispatch(fetchPostsIfNeeded('reactjs'))
  .then(() => console.log(store.getState())
)
```

### dispatch()

 View 发出 Action 的唯一方法。

```javascript
store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

`store.dispatch`接受一个 Action 对象作为参数，将它发送出去。store收到action后，就会调用reducer



### reducer

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。

接收当前 state 和 action作为参数，并返回一个新的 state 

```js
function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([{ text: action.text, completed: false }]);
  case 'TOGGLE_TODO':
    return state.map((todo, index) =>
      action.index === index ?
        { text: todo.text, completed: !todo.completed } :
        todo
   )
  default:
    return state;
  }
}
```

**拆分`reducer`**
```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

或者使用`combineReudcer`

```js
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```


### store.subscribe()

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

```javascript
store.subscribe(listener);
```

把render函数放入，就会实现view的自动渲染





## 中间件（middleware）

中间件就是一个函数，对`store.dispatch`方法进行了改造，在发出 Action 和执行 Reducer 这两步之间，添加了其他功能。



### applyMiddlewares()

作用是将所有中间件组成一个数组，依次执行。

每个中间件都会以科里化的形式传入`getState,dispatch,action`，然后在`applyMiddleware`中遍历执行所有的中间件，每个中间件只执行到`dispatch`，然后把返回的`dispatch`传递给下个中间件，这样就实现了对`dispatch`的封装，再把最后的`dispatch`传递给`store`。

```javascript
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```

简易实现：
- 所有中间件依次执行，对`dispatch`进行一些逻辑处理，并返回这个dispatch，传递给下个中间件

```js
// 警告：这只是一种“单纯”的实现方式！
// 这 *并不是* Redux 的 API.

function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  let dispatch = store.dispatch
  middlewares.forEach(middleware =>
    dispatch = middleware(getState)(dispatch)
  )

  return Object.assign({}, store, { dispatch })
}
```


### redux-thunk 中间件

使用`redux-thunk`中间件，改造`store.dispatch`，使得后者可以接受函数作为参数。

异步操作的第一种解决方案就是，写出一个返回函数的 Action Creator，在其中进行`promise`链式调用。

简易实现：

```js
/**
 * 让你可以发起一个函数来替代 action。
 * 这个函数接收 `dispatch` 和 `getState` 作为参数。
 *
 * 对于（根据 `getState()` 的情况）提前退出，或者异步控制流（ `dispatch()` 一些其他东西）来说，这非常有用。
 *
 * `dispatch` 会返回被发起函数的返回值。
 */
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action)
```


### redux-promise 中间件

使得`store.dispatch`方法可以接受 Promise 对象作为参数。

这时，Action Creator 有两种写法。写法一，返回值是一个 Promise 对象。

```javascript
const fetchPosts = 
  (dispatch, postTitle) => new Promise(function (resolve, reject) {
     dispatch(requestPosts(postTitle));
     return fetch(`/some/API/${postTitle}.json`)
       .then(response => {
         type: 'FETCH_POSTS',
         payload: response.json()
       });
});
```



写法二，Action 对象的`payload`属性是一个 Promise 对象。这需要从[`redux-actions`](https://github.com/acdlite/redux-actions)模块引入`createAction`方法，并且写法也要变成下面这样。

```javascript
import { createAction } from 'redux-actions';

class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    // 发出同步 Action
    dispatch(requestPosts(selectedPost));
    // 发出异步 Action
    dispatch(createAction(
      'FETCH_POSTS', 
      fetch(`/some/API/${postTitle}.json`)
        .then(response => response.json())
    ));
  }
```


## 数据流
**1. 调用** [`store.dispatch(action)`](https://www.redux.org.cn/docs/api/Store.html#dispatch)。

**2. Redux store 调用传入的 reducer 函数。**

**3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。**

**4. Redux store 保存了根 reducer 返回的完整 state 树。**

