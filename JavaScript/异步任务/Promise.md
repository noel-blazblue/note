Promise本身是一个状态机，具有pending（等待），fulfilled，rejected 这3个状态，

`Promise`对象有以下两个特点。

（1）对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

## 原型链方法

### Promise.prototype.then()
Promise 实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数是`rejected`状态的回调函数，它们都是可选的。

```javascript
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
```

### Promise.prototype.catch()
用于指定发生错误时的回调函数。
```javascript
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```
Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获。

### Promise.prototype.finally()
`finally()`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。
```javascript
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

## 静态方法

### Promise.all()
`Promise.all()`方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
只有当所有实例的状态都变为`fulfilled`之后，整个`Promise`实例才会变为`fulfilled`。
只要有一个成员变为`rejected`，那么整个`Promise`都会变为`rejected`。

### Promise.allSettled()
`Promise.allSettled()`方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。
只有等到所有这些参数实例都返回结果，不管是`fulfilled`还是`rejected`，包装实例才会结束。一旦结束，状态总是`fulfilled`，不会变成`rejected`。并且返回一个数组对应着`Promise`实例成员。
```javascript
const promises = [
  fetch('/api-1'),
  fetch('/api-2'),
  fetch('/api-3'),
];

await Promise.allSettled(promises);
removeLoadingIndicator();
```

### Promise.race()
`Promise.race()`方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。
当其中任何一个实例率先改变状态，总的`Promise`的状态就会跟着改，并且传递给回调函数。
```javascript
const p = Promise.race([p1, p2, p3]);
```

### Promise.any()
ES2021 引入了[`Promise.any()`方法](https://github.com/tc39/proposal-promise-any)。该方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。
只要参数实例有一个变成`fulfilled`状态，包装实例就会变成`fulfilled`状态；
如果所有参数实例都变成`rejected`状态，包装实例就会变成`rejected`状态。
```javascript
const promises = [
  fetch('/endpoint-a').then(() => 'a'),
  fetch('/endpoint-b').then(() => 'b'),
  fetch('/endpoint-c').then(() => 'c'),
];
try {
  const first = await Promise.any(promises);
  console.log(first);
} catch (error) {
  console.log(error);
}
```
上面代码中，`Promise.any()`方法的参数数组包含三个 Promise 操作。其中只要有一个变成`fulfilled`，`Promise.any()`返回的 Promise 对象就变成`fulfilled`。如果所有三个操作都变成`rejected`，那么`await`命令就会抛出错误。

`Promise.any()`抛出的错误，不是一个一般的错误，而是一个 AggregateError 实例。它相当于一个数组，每个成员对应一个被`rejected`的操作所抛出的错误。下面是 AggregateError 的实现示例。

### Promise.resolve()
有时需要将现有对象转为 Promise 对象，`Promise.resolve()`方法就起到这个作用。
```javascript
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

### Promise.reject()
`Promise.reject(reason)`方法也会返回一个新的 Promise 实例，该实例的状态为`rejected`。
```javascript
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```


## Promise 缺点

-   1、无法取消Promise，一旦新建它就会立即执行，无法中途取消。
-   2、如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
-   3、当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
-   4、Promise 真正执行回调的时候，定义 Promise 那部分实际上已经走完了，所以 Promise 的报错堆栈上下文不太友好。


## 实现 Promise
简易版
```js
class Promise{
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
      }
    };
    try {
      // 立即执行函数
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onRejected) {
    if (this.state === 'fulfilled') {
      let x = onFulfilled(this.value);
    };
    if (this.state === 'rejected') {
      let x = onRejected(this.reason);
    };
  }
}
```