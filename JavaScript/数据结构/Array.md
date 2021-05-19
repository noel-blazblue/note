

## API
### Array.from()
`Array.from`方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）

**参数：**

- `arrayLike`: 想要转换成数组的伪数组对象或可迭代对象。
- `mapFn` 可选,如果指定了该参数，新数组中的每个元素会执行该回调函数。
- `thisArg` 可选参数，执行回调函数 `mapFn` 时 `this` 对象


```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

### Array.of()
`Array.of()`总是返回参数值组成的数组。如果没有参数，就返回一个空数组。

```javascript
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

### Array.prototype.every()
`**every()**` 方法测试一个数组内的所有元素是否都能通过某个指定函数的测试。它返回一个布尔值。
```js
const isBelowThreshold = (currentValue) => currentValue < 40;

const array1 = [1, 30, 39, 29, 10, 13];

console.log(array1.every(isBelowThreshold));
// expected output: true
```

**参数：**

- `callback`：用来测试每个元素的函数，它可以接收三个参数：
	- `element`
		- 用于测试的当前值。
	- `index`
		- 可选，用于测试的当前值的索引。
	- `array`
		- 可选，调用 `every` 的当前数组。
- `thisArg`
	- 执行 `callback` 时使用的 `this` 值。

### copyWithin()
在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。

使用这个方法，会修改当前数组。

**参数：**

-   target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
-   start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
-   end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。


### fill()
`fill`方法使用给定值，填充一个数组。

```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```


## 数组的空位
数组的空位指，数组的某一个位置没有任何值。比如，`Array`构造函数返回的数组都是空位。
-   `forEach()`, `filter()`, `reduce()`, `every()` 和`some()`都会跳过空位。
-   `map()`会跳过空位，但会保留这个值
-   `join()`和`toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串。

`Array.from`方法会将数组的空位，转为`undefined`，也就是说，这个方法不会忽略空位。
扩展运算符（`...`）也会将空位转为`undefined`。
`copyWithin()`会连空位一起拷贝。
`fill()`会将空位视为正常的数组位置。
`for...of`循环也会遍历空位。
`entries()`、`keys()`、`values()`、`find()`和`findIndex()`会将空位处理成`undefined`。