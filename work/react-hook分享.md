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



**setState 可获取上次state的值**

```js
setCount(count => count + 1)
```

在异步函数中，遇到环境闭包未更新的情况下，使用此方式可获取最新的state。



#### useEffect

- 在完成渲染**之后**，异步执行，使用`requestIdleCallback`，在浏览器空闲时间执行传入的callback，16毫秒一帧。

- 回调函数的返回值，不仅仅是在组件卸载前执行的，还会在下一个`useEffect`触发前执行。

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

- render && useLayoutEffect1
- useEffect1
- render && clean useLayoutEffect1 && useLayoutEffect2
- clean useEffect1
- useEffect2



#### useRef

> react文档：
>
> `useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变。



react-hooks 是 `immutable`的，而`useRef`则是`mutable`，相当于走了个后门，搭配使用。



##### 使用场景

1. **保存一个字面量的引用类型的数据**



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

> **每一个希望深入hook实践的开发者都必须记住这个结论，无法自如地使用useRef会让你失去hook将近一半的能力。**



### 函数闭包

函数式语言公式：

```text
函数 + 参数 + 环境（闭包） => 返回值 + 环境（闭包）
 ↑    ↑    ↑ 
静态   动态   动态
```



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

