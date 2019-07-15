![img](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)



## redux

**整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。**



### Store

存放数据，整个应用只有一个store

```javascript
import { createStore } from 'redux';
const store = createStore(reducer);
```

`createStore`函数接受reducer作为参数，返回新生成的 Store 对象。



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



### store.dispatch()

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

`store.dispatch`方法会触发 Reducer 的自动执行。



### store.subscribe()

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

```javascript
store.subscribe(listener);
```

把render函数放入，就会实现view的自动渲染



