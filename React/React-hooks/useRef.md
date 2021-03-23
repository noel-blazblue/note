
useRef是一个可变数据，可以直接修改.current属性，一般用来保存一些数据或者dom节点引用。修改current并不会一起组件的渲染

内部实现原理也是创建一个hook节点，把ref对象挂载到memoizedState属性上，并push到hooks链表中，最终也会挂载到fiber节点上。但区别在于往后的每一次更新，都只是取出这个hook节点的memoizedState，也就是他的ref对象。因此，在整个生命周期，这个hook节点都不会再改变。

```js
function useRef<T\>(initialValue: T): {|current: T|} {

	currentlyRenderingComponent = resolveCurrentlyRenderingComponent();

	workInProgressHook = createWorkInProgressHook();

	const previousRef = workInProgressHook.memoizedState;

	if (previousRef === null) {

		const ref = {current: initialValue};

		workInProgressHook.memoizedState = ref;

		return ref;

	} else {

		return previousRef;

	}
}
```

## 用法

### 控制按钮

由于hook function 实际上会调用非常多次，在某一次调用时，他里面的方法获取到的是旧的state值。所以如果用state来控制按钮是否允许点击的话，在频繁点击时是控制不到的。

因此可以用ref来作为保存和控制，他不用通过更新调度就能直接修改数据，随时就能获取到最新的值。

### 防抖节流

由于传入的fn引用值会在每一次更新的时候都会改变，所以如果单纯的用useEffect去监听的话会造成死循环调用的问题，因此需要用ref来作为保存。

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

### 配合 `useImperativeHandle`使子组件暴露出去属性和方法

