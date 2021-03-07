## react-hooks 基础知识分享



### 什么是React-hooks？

react-hook 是一种基于 immutable 方式的编程方法和编程思想。是一种声明式的心智模型



**编程方法：**useState等api，声明式钩子。

**编程思想：**以model为主导，低耦合、高复用的函数式的编写方式。



### API简介

#### useState

1. 赋予初始值

```js
const [state, setState] = useState(initialState)

// ts泛型可以手动指定,也可以自动读取
useState<number>(1) === useState(1)

// 如果state是要通过异步获取，想要有一个空的初始值，并且有类型提示，那就可以用类型断言。
interface Company {
  shuzhikeji: '数智科技';
}

const [state, setState] = useState<Company>({}) // ts error
const [state, setState] = useState({} as Company) // ts true

state.shuzhikeji // ts true
```



2. 赋予初始化函数

```js
const [operation, setOperation] = useState(() => {
  return '编辑'
});
```

简易计算建议使用初始化函数，而不要放在useEffect中去做，尤其是在其他地方有依赖于初始值的情况下，就会导致前后`render`状态不一。



**setState 的更新以最后一次调用为准**

```js
setCount(1)
setCount(2)
setCount(3)

// output count === 3
```

每个hooks，内部都有一个队列去维护每一次setState，在同一次render中，只会执行队列链表中的尾部。



**setState 可获取上次state的值**

```js
setCount(count => count + 1)
```

在异步函数中，遇到环境闭包未更新的情况下，使用此方式可获取最新的state。



#### useEffect

- 在完成渲染**之后**，异步执行，使用`requestIdleCallback`，在浏览器空闲时间执行传入的callback，16毫秒一帧。

- 与`componentWillUnmount`不同，回调函数的返回值，不仅仅是在组件卸载前执行的，还会在下一个`useEffect`触发前执行。

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```



##### useEffect清除订阅的问题

```js
  const [lapse, setLapse] = React.useState(0);
  const [running, setRunning] = React.useState(false);

  useEffect(() => {
    if (running) {
      const startTime = Date.now() - lapse;
      const intervalId = setInterval(() => {
        setLapse(Date.now() - startTime);
      }, 0);
      return () => clearInterval(intervalId);
    }
  }, [running]);

  function handleRunClick() {
    setRunning((r) => !r);
  }

  function handleClearClick() {
    setRunning(false);
    setLapse(0);
  }

  return (
    <div>
      <label style={labelStyles}>{lapse}ms</label>
      <button onClick={handleRunClick} style={buttonStyles}>
        {running ? 'Stop' : 'Start'}
      </button>
      <button onClick={handleClearClick} style={buttonStyles}>
        Clear
      </button>
    </div>
  )
```



开启定时之后，点击`Clear`，时间不会清空。



**执行流程**

开启定时：

- setRunning(true)
- render渲染
- useEffect，Interval开启。



结束定时：

- setRunning(false) setLapse(0)
- render渲染
- 未清除的Interval，执行setLapse(Date.now() - startTime)
- clearInterval。(下个useEffect调用之前，执行上个useEffect的返回值)
- useEffect
- render渲染



#### useLayoutEffect

> react文档：它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，`useLayoutEffect` 内部的更新计划将被同步刷新。



useLayoutEffect与useEffect的执行流程：

- setRunning(true)
- render && useLayoutEffect1
- useEffect1
- setRunning(false)
- render && clean useLayoutEffect1 && useLayoutEffect2
- clean useEffect1
- useEffect2



#### useRef

> react文档：
>
> `useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变。



react-hooks 是 `immutable`的，而`useRef`则是`mutable`，相当于走了个后门，搭配使用。



##### 使用场景

1. **保存一个引用类型的数据**



```js
const useTimeout = (fn, time) => useEffect(
    () => {
        const tick = setTimeout(fn);
        return () => clearTimeout(tick);
    },
    [fn, time]
);
```

这是一个简略的函数防抖，如果这么使用：

```js
const fun = () => {}
fun1
fun2
fun3
fun4
useInterval(() => setCounter(counter => counter + 1));
```



他就会无限调用。因为每render一次，函数对象就会重新创建，useEffect监听到依赖值发生改变，就会重新调用。



此时可以用useRef解决引用问题。

```js
const useTimeout = (fn, time) => {
    const callback = useRef(fn);
    callback.current = fn;
    useEffect(
        () => {
            const tick = setTimeout(callback.current);
            return () => clearTimeout(tick);
        },
        [time]
    );
};
```



2. **获取前一次的state值**

class组件：

```js
class Foo extends Component {
 	 	state
    componentDidUpdate(prevProps, prevState, snapshot) {}
    componentWillReceiveProps(nextProps) {}
}
```



hook实现：

```js
const usePreviousValue = value => {
    const previous = useRef(undefined);
    const previousValue = previous.current;
    previous.current = value;
    return previousValue;
};
```



> React官方也曾经写过一些说明这一现象的博客，他们称`useRef`为“hook中的作弊器”，我想这个形容是准确的，所谓的“作弊”，其它是指它打破了类似`useCallback`、`useEffect`对闭包的约束，使用一个“可变的容器”让ref不需要成为闭包的依赖也可以在闭包中获得最新的内容。



### 函数闭包

### 函数闭包

函数式语言公式：

```text
函数 + 参数 + 环境（闭包） => 返回值 + 环境（闭包）
 ↑    ↑    ↑ 
静态   动态   动态
```



函数本身是静态编译的，是不会改变的，变的是参数和环境。react-hook 每render一次，是在创建函数对象以及闭包。

对于单个组件而言，只需要创建一次闭包就行了。因此，除非是组件嵌套层级多，否则无需担心性能问题。



编译前：

```js
const s = Math.random()

function createFunction(s) {
  console.log('hello', s)
  
}
createFunction(s)
```



编译后：

```js
const global = createClosure(closure)
global.s = Math.random()

function anonymous(closure) {
  console.log('hello', closure.s)
}
const createFunction = { closure: global, func: anonymous }

callFunction(createFunction)
```



#### 闭包陷阱

react每render一次，就会生成一个新的闭包，函数拿到的不是最新的state，而是当前闭包的state的值。

```js
const [count, setCount] = useState(0);

function handleAlertClick() {
  setTimeout(() => {
    alert('You clicked on: ' + count);
  }, 3000);
}

function changeCount() {
  setCount(count + 1)
}
```



拿到最新的state值的办法：

```js
// 1.通过setState的hook，获取到lastState
setCount(count => count + 1);

// 2.通过useReducer，获取到lastState
const [state, dispatch] = useReducer((state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    default:
  }
}, initialState);

// 3.通过useRef，去记录最新值。
const ref = useRef(count)
ref.current = count
function handleAlertClick() {
  setTimeout(() => {
    alert('You clicked on: ' + ref.current);
  }, 3000);
}
```



### 自定义hooks

#### 前端分层

以前端划分，可分为数据层，逻辑层，视图层。

以框架的设计理念来划分，则是mvvm，一个状态驱动视图的模型。

![img](http://5b0988e595225.cdn.sohucs.com/images/20180409/5e85879b6670487f9ce301c728c3382f.jpg)

```js
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { seconds: 0 };
  }
  tick() {
    this.setState(state => ({
      seconds: state.seconds + 1
    }));
  }
  render() {
    return (
      <div>
        Seconds: {this.state.seconds}
      </div>
    );
  }
}
```

```js
const Person = Vue.extend({
  template: `
    <div>
      <span>{{name}}</span>
      <span>{{age}}</span>
    </div>
  `,
  data() {
    return {
      name: 'some',
      age: 10,
    }
  },
  methods: {
    fecht () {}
    onSubmit () {}
    getName() {
      return this.name
    },
    setAge(age) {
      this.age = age
    },
  },
})
```



react class 组件与vue2.0 组件，其实只是UI框架，状态和视图耦合在一块，一切都是为视图而服务，没有独立的一层model。

很多本该由 service层，model层去做的事情，都放在了view层去做。导致view层在处理前端交互的同时，还要去处理业务逻辑，与接口通信。代码耦合性严重，不利于复用。

redux、vuex可以算做model层，但是其本身是为了解决组件通信的问题，把它定义为处理单个页面的业务，并不是一种很好的实践。数据和逻辑处理本就该由单个模块自己去解决，才能形成高内聚，低耦合的架构。



#### model是什么

```js
class Good {
  type = ''
  price = 0
  count = 0

  up(count) {
    this.count += count
  }
  down(count) {
     this.count -= count
  }
	
}
```

这就是一个极为简单的模型，描述了一个商品应有的属性，以及可以进行什么样的操作。和上面两个例子不同，他只是描述自己本身，而不是为了服务于视图。他的模型是自洽的，形成一个闭环。



如果加上状态驱动视图的模型，那就变成这样：

```js
const useGood = () => {
  const [type, setType] = useState('');
  const [price, setPrice] = useState(0);
  const [count, setCount] = useState(0);

  const up = (num) => {
    setCount(count + num)
  }

  const down = (num) => {
    setCount(count - num)
  }
  
  return {count, up, down}
}
```

这就是一个典型的自定义hooks，有自身的数据和逻辑，不受外部影响，只描述自身。



在编写一个庞大且复杂的业务中，就可以以这种模式，拆分为一个个小的模块，且可以任意的组合、复用，形成一个高内聚、低耦合的架构。

把model层划分出来，视图的归视图，逻辑的归逻辑，彼此互不干扰。从 view 导向，变成model导向，这就是 react hooks 的编程思想。

