Generator 是一个状态机，内部封装了多个状态。通过`yield`去暂停函数内代码的执行，然后通过`next`迭代来恢复代码执行，属于一个异步操作的容器。

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
```

```javascript
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

`value`为当前遍历器的值，`done`: 为遍历是否结束的标志。

## Generator 语法

### yield 表达式
`yield`代表暂停执行。

调用`next`的时候，会执行到`yield`后面的语句，然后下面的代码就暂停执行。

```js
function *run(){
  let a = null
  a = yield 1
  console.log(a)
  a = yield 2
  console.log(a)
}

let generator = run()
generator.next(3)
generator.next(4) // console.log(4)
generator.next(5) // console.log(5)
```


| 区别   |                    |                          |      |      |
| ------ | ------------------ | ------------------------ | ---- | ---- |
| yield  | 具备记忆位置的功能 | 可执行多次，返回多个值   |      |      |
| return | 不具备             | 只能执行一次，返回一个值 |      |      |

`yield`表达式如果用在另一个表达式之中，必须放在圆括号里面。 

```js
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}
```

`yield`表达式用作函数参数或放在赋值表达式的右边，可以不加括号。 

```js
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```



由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的`Symbol.iterator`属性，从而使得该对象具有 Iterator 接口。 

```js
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```

### yield* 表达式

`yield*`表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。 

`yield*`后面的 Generator 函数（没有`return`语句时），不过是`for...of`的一种简写形式，完全可以用后者替代前者。反之，在有`return`语句时，则需要用`var value = yield* iterator`的形式获取`return`语句的值。 

任何数据结构只要有 Iterator 接口，就可以被`yield*`遍历。 

```js
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}
```


### Generator.prototype.next()

`next()` 方法迭代生成器的状态到下个`yield`指针，并返回一个包含属性 `done` 和 `value` 的对象。该方法也可以通过接受一个参数用以向生成器传值。

**参数：**
`value`： 向生成器传递的值.

**返回值：**
返回的对象包含两个属性:
-   `done` (布尔类型)
    -   如果迭代器超过迭代序列的末尾，则值为 `true`。 在这种情况下，`value`可选地指定迭代器的返回值。
    -   如果迭代器能够生成序列中的下一个值，则值为`false`。 这相当于没有完全指定`done`属性。
-   `value` \- 迭代器`yield`表达式返回的任意的Javascript值。当 `done` 的值为 `true` 时可以忽略该值。


`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值。 

```js
function *run(){
  let a = null
  a = yield 1
  console.log(a)
  a = yield 2
  console.log(a)
}

let generator = run()
generator.next(3)
generator.next(4) // console.log(4)
generator.next(5) // console.log(5)
```


### Generator.prototype.throw()

`throw()` 方法用来向生成器抛出异常，并恢复生成器的执行，返回带有 `done` 及 `value` 两个属性的对象。

异常可以被函数体内或者外面的`try catch()`捕捉，如果没有被捕获，则会终止函数的执行。

`throw`方法可以接受一个参数，该参数会被`catch`语句接收，建议抛出`Error`对象的实例。 

```js
var g = function* () {
  while (true) {
    yield;
    console.log('内部捕获', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 外部捕获 a
```

如果 Generator 函数内部和外部，都没有部署`try...catch`代码块，那么程序将报错，直接中断执行。 

`throw`方法抛出的错误要被内部捕获，前提是必须至少执行过一次`next`方法。 

`next`方法一次都没有执行过。这时，抛出的错误不会被内部捕获，而是直接在外部抛出，导致程序出错。这种行为其实很好理解，因为第一次执行`next`方法，等同于启动执行 Generator 函数的内部代码，否则 Generator 函数还没有开始执行，这时`throw`方法抛错只可能抛出在函数外部。 

`throw`方法被捕获以后，会附带执行下一条`yield`表达式。也就是说，会附带执行一次`next`方法。 

一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。如果此后还调用`next`方法，将返回一个`value`属性等于`undefined`、`done`属性等于`true`的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了。 



### Generator.prototype.return()

`return`方法返回给定的值并结束生成器。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

如果`return`方法调用时，不提供参数，则返回值的`value`属性为`undefined`。 

如果 Generator 函数内部有`try...finally`代码块，且正在执行`try`代码块，那么`return`方法会推迟到`finally`代码块执行完再执行。 



### next()、throw()、return() 的共同点

`next()`、`throw()`、`return()`这三个方法本质上是同一件事，可以放在一起理解。它们的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换`yield`表达式。 

- `next()`是将`yield`表达式替换成一个值。 
  - 无参数，就替换成undefined
  - 有参数，就替换成那个参数
- `throw()`是将`yield`表达式替换成一个`throw`语句。 
- `return()`是将`yield`表达式替换成一个`return`语句。 

### for...of 循环
`for...of`循环可以自动遍历 Generator 函数运行时生成的`Iterator`对象，且此时不再需要调用`next`方法。

遍历到的每一个值，就是 Generator 对象的`value`。

一旦`next`方法的返回对象的`done`属性为`true`，`for...of`循环就会中止，且不包含该返回对象，所以上面代码的`return`语句返回的`6`，不包括在`for...of`循环之中。 

```js
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```



将 Generator 函数加到对象的`Symbol.iterator`属性上面。 

```js
function* objectEntries() {
  let propKeys = Object.keys(this);

  for (let propKey of propKeys) {
    yield [propKey, this[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

jane[Symbol.iterator] = objectEntries;

for (let [key, value] of jane) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```



### 作为对象属性的 Generator 函数

```js
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
等价于
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```



### Generator 函数的this

Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的`prototype`对象上的方法。 

Generator 函数也不能跟`new`命令一起用，会报错。 

把Generator变成构造函数，共享属性和方法的例子：

```js
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```



### Generator 与上下文

Generator 函数不是这样，它执行产生的上下文环境，一旦遇到`yield`命令，就会暂时退出堆栈，但是并不消失，里面的所有变量和对象会冻结在当前状态。等到对它执行`next`命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行。 


## 异步应用

