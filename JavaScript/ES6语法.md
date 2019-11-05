## let和const

#### 1.块级作用域内奏效



#### 2.不存在变量提升



#### 3.暂时性死区

只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。 同名的变量都以let为主

#### 4.不允许重复声明





#### 1.函数允许在块级作用域内声明

- 允许在块级作用域内声明函数。

- 函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。

- 同时，函数声明还会提升到所在的块级作用域的头部。

  

  `let`命令、`const`命令、`class`命令声明的全局变量，不属于顶层对象的属性。 

### const命令

`const`声明一个只读的常量。一旦声明，常量的值就不能改变。 

```
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

- `const`的作用域与`let`命令相同：只在声明所在的块级作用域内有效。 
- `const`命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。 
- `const`声明的常量，也与`let`一样不可重复声明。 



`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。 

但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，`const`只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了 



还有一些跟let命令不一样的地方：const必须在声明的时候赋值，不然就会报错。const声明的常量不能更改。

这里的常量指的是：数值、字符串、布尔值，对于引用类型（数组和对象），const只能保证指针是固定的，至于数组和对象内部有没有改变就是const不能控制的地方

 



 



## 四、解构

#### 1.数组的解构赋值

这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。 

```
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```



不完全结构

```
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```

#### 2.对象的解构赋值 

变量必须与属性同名，才能取到正确的值。 

```
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
```

#### 3.字符串的解构赋值

```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

类似数组的对象都有一个`length`属性，因此还可以对这个属性解构赋值。 

```
let {length : len} = 'hello';
len // 5
```

#### 4.数值和布尔值的解构赋值

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于`undefined`和`null`无法转为对象，所以对它们进行解构赋值，都会报错。 

```
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

```
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```



## 五、字符串的拓展

#### 1.字符串的遍历器接口

```
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
```

#### 2.新增方法

- **includes(str,n)**：返回布尔值，表示是否找到了参数字符串。

- **startsWith(str,n)**：返回布尔值，表示参数字符串是否在原字符串的头部。

- **endsWithstr,n()**：返回布尔值，表示参数字符串是否在原字符串的尾部。

- **repeat：**返回一个新字符串，表示将原字符串重复`n`次。 

  

## 六、函数的拓展

#### 1.参数设置默认值

```
function Point(x = 0, y = 0) {
  this.x = x;
  this.y = y;
}

const p = new Point();
p // { x: 0, y: 0 }
```

参数变量`x`是默认声明的，在函数体中，不能用`let`或`const`再次声明 

使用参数默认值时，函数不能有同名参数。 

一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的。 

#### 2.箭头函数

() => {}

箭头函数没有自己的this，箭头函数中的this继承于外层代码库中的this.

内部的`this`是词法作用域，由上下文确定。 



1. 箭头函数没有arguments（建议使用更好的语法，剩余运算符替代）
2. 箭头函数没有prototype属性，没有constructor，即不能用作与构造函数（不能用new关键字调用）
3. 箭头函数没有自己this，它的this是词法的，引用的是上下文的this，即在你写这行代码的时候就箭头函数的this就已经和外层执行上下文的this绑定了(这里个人认为并不代表完全是静态的,因为外层的上下文仍是动态的
4. 可以使用call,apply,bind修改,这里只是说明了箭头函数的this始终等于它上层上下文中的this)



箭头函数是ES6中新增的，它和普通函数有一些区别，箭头函数没有自己的this，它的this继承于外层代码库中的this。箭头函数在使用时，需要注意以下几点:

（1）函数体内的this对象，继承的是外层代码块的this。

（2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

（3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

（4）不可以使用yield命令，因此箭头函数不能用作 Generator 函数。

（5）箭头函数没有自己的this，所以不能用call()、apply()、bind()这些方法去改变this的指向.

 

## 七、数组的拓展

### entries()，keys()，values()

`keys()`是对键名的遍历、`values()`是对键值的遍历，`entries()`是对键值对的遍历。 



### find() ， findIndex()

**find**

方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回`undefined`。 

- value 当前的值
- index 当前的索引
- arr 当前的数组

**findIndex** 

返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回`-1`。 

参数与上相同



### includes 

arr.includes ()  返回一个布尔值，表示某个数组是否包含给定的值 

参数：

- element  要搜索的值
- index  起始位置，默认为0



### map

 arr.map(callback)

`map`按顺序为数组中的**每个元素**调用一次传入的`callback`函数，并从结果中构造一个新数组。 

传入一个函数，依次对数组遍历执行这个函数，返回一个新的数组

```
function pow(x) {
    return x * x;
}
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
var results = arr.map(pow); // [1, 4, 9, 16, 25, 36, 49, 64, 81]
```

 

### reduce

`reduce` 来说，它接受两个参数，分别是回调函数和初始值 

回调参数：

- acc 累加器
- cur 当前值
- idx  索引
- src  源数组

 arr.reduce()

一般只需要传入前两个值

```
[x1, x2, x3, x4].reduce(f) = f(f(f(x1, x2), x3), x4)
```

```
var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x + y;
}); // 25
```



### filter

`filter()`把传入的函数依次作用于数组的每个元素，然后根据返回值是`true`还是`false`决定保留还是丢弃该元素，最终返回一个新数组。

```
var arr = [1, 2, 4, 5, 6, 9, 10, 15];
var r = arr.filter(function (x) {
    return x % 2 !== 0;
});
r; // [1, 5, 9, 15]
```

参数：

- element 数组的元素
- index 数组的索引
- self 数组本身

通常只用到第一个参数



### sort

可传入一个执行函数

参数：

- x
- y

对于两个元素`x`和`y`，如果认为`x < y`，则返回`-1`，如果认为`x == y`，则返回`0`，如果认为`x > y`，则返回`1`

`Array`的`sort()`方法默认把所有元素先转换为String再排序 



## 八、对象的拓展

### keys()，values()，entries() 

Object.keys()，Object.values()，Object.entries()

参数是一个对象



## 八、Iterator与for ... of循环

它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作 

遍历器对象本质上，就是一个指针对象。

**遍历过程：**

- 创建一个指针对象，指向当前数据结构的起始位置。
- 第一次调用指针对象的`next`方法，可以将指针指向数据结构的第一个成员。以后每次调用都往后移动一位成员

**返回：**

每次调用next（），都会返回一个对象，是遍历成员的信息

- value     成员的值
- done true/false   是否遍历完成



由于 Iterator 只是把接口规格加到数据结构之上，所以，遍历器与它所遍历的那个数据结构，实际上是分开的 



默认的 Iterator 接口部署在数据结构的`Symbol.iterator`属性，`Symbol.iterator`属性本身是一个函数，就是当前数据结构默认的遍历器生成函数，执行这个函数，就会返回一个遍历器。 



原生具备 Iterator 接口的数据结构如下：

- Array
- Map
- Set
- String
- TypedArray
- 函数的 arguments 对象
- NodeList 对象



```
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```



对于原生部署 Iterator 接口的数据结构，不用自己写遍历器生成函数，`for...of`循环会自动遍历它们。除此之外，其他数据结构（主要是对象）的 Iterator 接口，都需要自己在`Symbol.iterator`属性上面部署，这样才会被`for...of`循环遍历。 



```
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"
var iterator = someString[Symbol.iterator]();
iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```



### 默认调用iterator

- **数组、Set 的结构赋值**，会默认调用`Symbol.iterator`方法。 

  - ```
    let set = new Set().add('a').add('b').add('c');
    
    let [x,y] = set;
    // x='a'; y='b'
    ```

- **扩展运算符**，只要某个数据结构部署了 Iterator 接口，就可以对它使用扩展运算符，将其转为数组。 

  - ```
    [...str] //  ['h','e','l','l','o']
    ```

- **yield***  后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。 

- 任何接受数组作为参数的场合，都调用了遍历器接口。 

  - for...of
  - Array.from()
  - Map(), Set(), WeakMap(), WeakSet()（比如`new Map([['a',1],['b',2]])`）
  - Promise.all()
  - Promise.race()



###  return()，throw()

- `return`方法的使用场合是，如果`for...of`循环提前退出（通常是因为出错，或者有`break`语句），就会调用`return`方法。
- 如果一个对象在完成遍历前，需要清理或释放资源，就可以部署`return`方法。 



`throw`方法主要是配合 Generator 函数使用，一般的遍历器对象用不到这个方法。 



```
for (var n of fibonacci) {
  if (n > 1000)
    break;
  console.log(n);
}
```



### for……of

一个数据结构只要部署了`Symbol.iterator`属性，就被视为具有 iterator 接口，就可以用`for...of`循环遍历它的成员。 `for...of`循环内部调用的是数据结构的Symbol.iterator方法。 



#### 数组

```
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
} 
```

`for...of`循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。 



#### Set和Map

```
var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

- 遍历的顺序是按照各个成员被添加进数据结构的顺序。
- Set 结构遍历时，返回的是一个值
- Map 结构遍历时，返回的是一个数组，该数组的两个成员分别为当前 Map 成员的键名和键值。 

```
let map = new Map().set('a', 1).set('b', 2);
for (let pair of map) {
  console.log(pair);
}
// ['a', 1]
// ['b', 2]

for (let [key, value] of map) {
  console.log(key + ' : ' + value);
}
// a : 1
// b : 2
```



- `entries()` 返回一个遍历器对象，用来遍历`[键名, 键值]`组成的数组。对于数组，键名就是索引值；对于 Set，键名与键值相同。Map 结构的 Iterator 接口，默认就是调用`entries`方法。
- `keys()` 返回一个遍历器对象，用来遍历所有的键名。
- `values()` 返回一个遍历器对象，用来遍历所有的键值。



#### 对象

对于普通的对象，`for...of`结构不能直接使用，会报错，必须部署了 Iterator 接口后才能使用。 

一种解决方法是，使用`Object.keys`方法将对象的键名生成一个数组，然后遍历这个数组。 

```
for (var key of Object.keys(someObject)) {
  console.log(key + ': ' + someObject[key]);
}
```



另一个方法是使用 Generator 函数将对象重新包装一下。 

```
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```



`for...in`循环有几个缺点。

- 数组的键名是数字，但是`for...in`循环是以字符串作为键名“0”、“1”、“2”等等。
- `for...in`循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键。
- 某些情况下，`for...in`循环会以任意顺序遍历键名。

总之，`for...in`循环主要是为遍历对象而设计的，不适用于遍历数组。





## 九、剩余/扩展运算符

扩展运算符：

将一个数组转为用逗号分隔的参数序列。 

```
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5
```





## 十、对象属性/方法简写



## Set数据结构

 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

`Set`本身是一个构造函数，用来生成 Set 数据结构。



`Set`函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。 

```
// 例一
const set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 例二
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size // 5

// 例三
const set = new Set(document.querySelectorAll('div'));
set.size // 56

// 类似于
const set = new Set();
document
 .querySelectorAll('div')
 .forEach(div => set.add(div));
set.size // 56
```



数组或字符串去重

```
// 去除数组的重复成员
[...new Set(array)]
```

```
[...new Set('ababbc')].join('')
// "abc"
```



向 Set 加入值的时候，不会发生类型转换，所以`5`和`"5"`是两个不同的值。 



### Set 实例的属性和方法

#### size,add(),delete(),has(),clear()

Set 结构的实例有以下属性。

- `Set.prototype.constructor`：构造函数，默认就是`Set`函数。
- `Set.prototype.size`：返回`Set`实例的成员总数。

Set 实例的方法分为两大类：操作方法（用于操作数据）和遍历方法（用于遍历成员）。下面先介绍四个操作方法。

- `add(value)`：添加某个值，返回 Set 结构本身。
- `delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功。
- `has(value)`：返回一个布尔值，表示该值是否为`Set`的成员。
- `clear()`：清除所有成员，没有返回值。

```
s.add(1).add(2).add(2);
// 注意2被加入了两次

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2);
s.has(2) // false
```



`Array.from`方法可以将 Set 结构转为数组。

```
const items = new Set([1, 2, 3, 4, 5]);
const array = Array.from(items);
```

这就提供了去除数组重复成员的另一种方法。

```
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```



### Set遍历操作

Set 结构的实例有四个遍历方法，可以用于遍历成员。

- `keys()`：返回键名的遍历器
- `values()`：返回键值的遍历器
- `entries()`：返回键值对的遍历器
- `forEach()`：使用回调函数遍历每个成员



#### **keys()，values()，entries()** 

`keys`方法、`values`方法、`entries`方法返回的都是遍历器对象。由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以`keys`方法和`values`方法的行为完全一致。 

```
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```



Set 结构的实例默认可遍历，它的默认遍历器生成函数就是它的`values`方法。 

```
Set.prototype[Symbol.iterator] === Set.prototype.values
// true
```

这意味着，可以省略`values`方法，直接用`for...of`循环遍历 Set。 

```
let set = new Set(['red', 'green', 'blue']);

for (let x of set) {
  console.log(x);
}
// red
// green
// blue
```



#### **forEach()** 

Set 结构的实例与数组一样，也拥有`forEach`方法，用于对每个成员执行某种操作，没有返回值。 



扩展运算符（`...`）内部使用`for...of`循环，所以也可以用于 Set 结构。 

```
let set = new Set(['red', 'green', 'blue']);
let arr = [...set];
// ['red', 'green', 'blue']
```

使用 Set 可以很容易地实现并集（Union）、交集（Intersect）和差集（Difference）。 

```
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```



如果想在遍历操作中，同步改变原来的 Set 结构，目前没有直接的方法，但有两种变通方法。一种是利用原 Set 结构映射出一个新的结构，然后赋值给原来的 Set 结构；另一种是利用`Array.from`方法。 

```
// 方法一
let set = new Set([1, 2, 3]);
set = new Set([...set].map(val => val * 2));
// set的值是2, 4, 6

// 方法二
let set = new Set([1, 2, 3]);
set = new Set(Array.from(set, val => val * 2));
// set的值是2, 4, 6
```



### WeakSet 

WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。

- 首先，WeakSet 的成员只能是对象，而不能是其他类型的值。
- 其次，WeakSet 中的对象都是弱引用，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。 

因此，WeakSet 适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在 WeakSet 里面的引用就会自动消失。 

由于上面这个特点，WeakSet 的成员是不适合引用的，因为它会随时消失。另外，由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此 ES6 规定 WeakSet 不可遍历。 

WeakSet 的一个用处，是储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏。 





WeakSet 是一个构造函数，可以使用`new`命令，创建 WeakSet 数据结构。 接受一个数组或类似数组的对象作为参数。（实际上，任何具有 Iterable 接口的对象，都可以作为 WeakSet 的参数。）该数组的所有成员，都会自动成为 WeakSet 实例对象的成员。 

```
const a = [[1, 2], [3, 4]];
const ws = new WeakSet(a);
// WeakSet {[1, 2], [3, 4]}
```



#### WeakSet 方法

- **WeakSet.prototype.add(value)**：向 WeakSet 实例添加一个新成员。
- **WeakSet.prototype.delete(value)**：清除 WeakSet 实例的指定成员。
- **WeakSet.prototype.has(value)**：返回一个布尔值，表示某个值是否在 WeakSet 实例之中。

WeakSet 没有`size`属性，没有办法遍历它的成员。 

```
const ws = new WeakSet();
const obj = {};
const foo = {};

ws.add(window);
ws.add(obj);

ws.has(window); // true
ws.has(foo);    // false

ws.delete(window);
ws.has(window);    // false
```



## Map数据结构

### 键值对

Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。 

Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。



```
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

Map可用Set的方法，添加成员，查询成员



Map 也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。 

```
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"
```

任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构（详见《Iterator》一章）都可以当作`Map`构造函数的参数。 



注意，只有对同一个对象的引用，Map 结构才将其视为同一个键。 

```
const map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```

Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题

```
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```



如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键，比如`0`和`-0`就是一个键，布尔值`true`和字符串`true`则是两个不同的键。另外，`undefined`和`null`也是两个不同的键。虽然`NaN`不严格相等于自身，但 Map 将其视为同一个键。 



### 实例的属性和操作方法

#### size,set(),delete(),has(),clear()

- **size 属性** 
  - 返回 Map 结构的成员总数。 
- **set(key, value)** 
  - `set`方法设置键名`key`对应的键值为`value`，然后返回整个 Map 结构。如果`key`已经有值，则键值会被更新，否则就新生成该键。 `set`方法返回的是当前的`Map`对象，因此可以采用链式写法。 
- **get(key)** 
  - `get`方法读取`key`对应的键值，如果找不到`key`，返回`undefined`。 
- **has(key)** 
  - `has`方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。 
- **delete(key)** 
  - 删除某个键，返回`true`。如果删除失败，返回`false`。 
- **clear()** 
  - 清除所有成员，没有返回值。 



### 遍历方法

#### keys() values() entries() forEach() 

Map 结构原生提供三个遍历器生成函数和一个遍历方法。 

- `keys()`：返回键名的遍历器。
- `values()`：返回键值的遍历器。
- `entries()`：返回所有成员的遍历器。
- `forEach()`：遍历 Map 的所有成员。



**Map 结构转为数组结构 ：**

```
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```

**遍历和过滤：**

```
const map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

const map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 产生 Map 结构 {1 => 'a', 2 => 'b'}

const map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```

**`forEach`方法还可以接受第二个参数，用来绑定`this`：**

```
const reporter = {
  report: function(key, value) {
    console.log("Key: %s, Value: %s", key, value);
  }
};

map.forEach(function(value, key, map) {
  this.report(key, value);
}, reporter);
```



### 与其他数据结构的互相转换

**Map 转为数组** :

```
const myMap = new Map()
  .set(true, 7)
  .set({foo: 3}, ['abc']);
[...myMap]
// [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

**数组 转为 Map** :

```
new Map([
  [true, 7],
  [{foo: 3}, ['abc']]
])
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
```

**Map 转为对象** :

如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名。 

```
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```

**对象转为 Map** :

```
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}

objToStrMap({yes: true, no: false})
// Map {"yes" => true, "no" => false}
```

**Map 转为 JSON** :

- 如果Map 的键名都是字符串，这时可以选择转为对象，再转为 JSON。 

- 如果Map 的键名有非字符串，这时可以选择转为数组，再转为 JSON。 

**JSON 转为 Map** ：

- 如果所有键名都是字符串，则先转为对象，再转为Map

- 如果整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。 这时，它可以一一对应地转为 Map。 

  - ```
    function jsonToMap(jsonStr) {
      return new Map(JSON.parse(jsonStr));
    }
    
    jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
    // Map {true => 7, Object {foo: 3} => ['abc']}
    ```



### WeakMap

`WeakMap`结构与`Map`结构类似，也是用于生成键值对的集合。 

`WeakMap`与`Map`的区别有两点。

- 首先，`WeakMap`只接受对象作为键名（`null`除外），不接受其他类型的值作为键名。
- 其次，`WeakMap`的键名所指向的对象，不计入垃圾回收机制。 
  - 只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。 

在网页的 DOM 元素上添加数据，就可以使用`WeakMap`结构。当该 DOM 元素被清除，其所对应的`WeakMap`记录就会自动被移除。 

`WeakMap`的专用场合就是，它的键所对应的对象，可能会在将来消失。`WeakMap`结构有助于防止内存泄漏。 

WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。 



#### `get()`、`set()`、`has()`、`delete()`

WeakMap 与 Map 在 API 上的区别

- 一是没有遍历操作（即没有keys()、values()和entries()方法），
- 二是没有size属性。因为没有办法列出所有键名，某个键名是否存在完全不可预测，跟垃圾回收机制是否运行相关。这一刻可以取到键名，下一刻垃圾回收机制突然运行了，这个键名就没了，为了防止出现不确定性，就统一规定不能取到键名。
- 三是无法清空，即不支持clear方法。 



## class

class改写了ES5的继承方式，让对象原型的写法更加清晰、更像面向对象编程的语法。

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

类的数据类型就是函数，类本身就指向构造函数。

使用的时候，也是直接对类使用`new`命令，跟构造函数的用法完全一致。

```
var b = new Point();
```



构造函数的`prototype`属性，在 ES6 的“类”上面继续存在。事实上，类的所有方法都定义在类的`prototype`属性上面。 

类的内部所有定义的方法，都是不可枚举的（non-enumerable）。 

```
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```



`Object.assign`方法可以很方便地一次向类添加多个方法 

```
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```



**constructor 方法**

`constructor`方法是类的默认方法，通过`new`命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个空的`constructor`方法会被默认添加。 

`constructor`方法默认返回实例对象（即`this`），完全可以指定返回另外一个对象。 



类的属性名，可以采用表达式。 

```
let methodName = 'getArea';

class Square {
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

类也可以使用表达式的形式定义。 

```
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
```





类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。考虑到未来所有的代码，其实都是运行在模块之中，所以 ES6 实际上把整个语言升级到了严格模式。 

类不存在变量提升（hoist），这一点与 ES5 完全不同。 



### 静态方法

所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上`static`关键字，就表示该方法不会被实例继承，而是直接通过类来调用 

```
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

如果静态方法包含`this`关键字，这个`this`指的是类，而不是实例。 



父类的静态方法，可以被子类继承。 

```
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod() // 'hello'
```



实例属性除了定义在`constructor()`方法里面的`this`上面，也可以定义在类的最顶层。 

```
class foo {
  bar = 'hello';
  baz = 'world';

  constructor() {
    // ...
  }
}
```



### 静态属性

静态属性指的是 Class 本身的属性，即`Class.propName`，而不是定义在实例对象（`this`）上的属性。 

```
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```



### 私有方法和私有属性

私有方法和私有属性，是只能在类的内部访问的方法和属性，外部不能访问。 



利用`Symbol`值的唯一性，将私有方法的名字命名为一个`Symbol`值。 

```
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```





### new.target 属性

该属性一般用在构造函数之中，返回`new`命令作用于的那个构造函数。如果构造函数不是通过`new`命令或`Reflect.construct()`调用的，`new.target`会返回`undefined` 



### 继承

- ES5 的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面（`Parent.apply(this)`）。
- ES6 的继承机制完全不同，实质是先将父类实例对象的属性和方法，加到`this`上面（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。 



Class 可以通过`extends`关键字实现继承 

```
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }
  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```



如果子类没有定义`constructor`方法，这个方法会被默认添加  

父类的静态方法，也会被子类继承。 



`Object.getPrototypeOf`方法可以用来从子类上获取父类。 



#### super 关键字

`super`关键字，它在这里表示父类的构造函数，用来新建父类的`this`对象。 如果没有调用`super`方法，新建实例时就会报错。

只有调用`super`之后，才可以使用`this`关键字，否则会报错。这是因为子类实例的构建，基于父类实例，只有`super`方法才能调用父类实例。 ES6 要求，子类的构造函数必须执行一次`super`函数。 

```
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

`super()`在这里相当于`A.prototype.constructor.call(this)`。 



`super`作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。 



### 类的 prototype 属性和__proto__属性

Class 作为构造函数的语法糖，同时有`prototype`属性和`__proto__`属性，因此同时存在两条继承链。 









## Proxy 

Proxy 是一种拦截器，在目标对象之前架设一层“拦截” ，对外界的访问进行过滤和改写。

```
var proxy = new Proxy(target, handler);

```

![1554857255337](C:\Users\ADMINI~1\AppData\Local\Temp\1554857255337.png)

![1554857291916](C:\Users\ADMINI~1\AppData\Local\Temp\1554857291916.png)







### Proxy 实例的方法

#### get()

`get`方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和 proxy 实例本身 ，最后一个参数可选。 

```
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误
```

`get`方法可以继承。 



如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。 



#### set()

`set`方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和 Proxy 实例本身，其中最后一个参数可选。 

```
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 对于满足条件的 age 属性以及其他属性，直接保存
    obj[prop] = value;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 报错
person.age = 300 // 报错
```



- ### apply()

`apply`方法拦截函数的调用、`call`和`apply`操作。 

`apply`方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（`this`）和目标对象的参数数组。 



- ### has()

`has`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是`in`运算符。 

`has`方法可以接受两个参数，分别是目标对象、需查询的属性名。 



- ### construct()

`construct`方法用于拦截`new`命令

- `target`：目标对象
- `args`：构造函数的参数对象
- `newTarget`：创造实例对象时，`new`命令作用的构造函数



- ### deleteProperty()

`deleteProperty`方法用于拦截`delete`操作，如果这个方法抛出错误或者返回`false`，当前属性就无法被`delete`命令删除。 



- ### defineProperty()

`defineProperty`方法拦截了`Object.defineProperty`操作。 









## Symbol

ES5 的对象属性名都是字符串，这容易造成属性名的冲突。 凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。 

Symbol 值通过`Symbol`函数生成。 

注意，`Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。 

如果 Symbol 的参数是一个对象，就会调用该对象的`toString`方法，将其转为字符串，然后才生成一个 Symbol 值。 



Symbol`函数可以接受一个字符串作为参数 

`Symbol`函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的`Symbol`函数的返回值是不相等的。 

```
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)
```



Symbol 值不能与其他类型的值进行运算，会报错。 但是，Symbol 值可以显式转为字符串。 Symbol 值也可以转为布尔值，但是不能转为数值。 



作为一个对象的属性名

```
// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });
```



注意，Symbol 值作为对象属性名时，不能用点运算符。 因为点运算符后面总是字符串，所以不会读取`mySymbol`作为标识名所指代的那个值，导致`a`的属性名实际上是一个字符串，而不是一个 Symbol 值。 

Symbol 值必须放在方括号之中。 

```
let obj = {
  [s]: function (arg) { ... }
};
```

```
let obj = {
  [s](arg) { ... }
};
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};
```



ymbol 值作为属性名时，该属性还是公开属性，不是私有属性。 



`Object.getOwnPropertySymbols`方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。 

`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。 



### Symbol.for()，Symbol.keyFor()

`Symbol.for`接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。 

```
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');
s1 === s2 // true
```

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的 Symbol。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。`Symbol.for()`不会每次调用就返回一个新的 Symbol 类型的值，而是会先检查给定的`key`是否已经存在，如果不存在才会新建一个值。 



`Symbol.keyFor`方法返回一个已登记的 Symbol 类型值的`key`。 

```
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```



除了定义自己使用的 Symbol 值以外，ES6 还提供了 11 个内置的 Symbol 值，指向语言内部使用的方法。：

### Symbol.hasInstance 

对象的`Symbol.hasInstance`属性，指向一个内部方法。当其他对象使用`instanceof`运算符，判断是否为该对象的实例时，会调用这个方法。 



### Symbol.isConcatSpreadable

对象的`Symbol.isConcatSpreadable`属性等于一个布尔值，表示该对象用于`Array.prototype.concat()`时，是否可以展开。 

数组的默认行为是可以展开，`Symbol.isConcatSpreadable`默认等于`undefined`。该属性等于`true`时，也有展开的效果。

类似数组的对象正好相反，默认不展开。它的`Symbol.isConcatSpreadable`属性设为`true`，才可以展开。

```
let arr1 = ['c', 'd'];
['a', 'b'].concat(arr1, 'e') // ['a', 'b', 'c', 'd', 'e']
arr1[Symbol.isConcatSpreadable] // undefined

let arr2 = ['c', 'd'];
arr2[Symbol.isConcatSpreadable] = false;
['a', 'b'].concat(arr2, 'e') // ['a', 'b', ['c','d'], 'e']
```



### Symbol.species

`Symbol.species`的作用在于，实例对象在运行过程中，需要再次调用自身的构造函数时，会调用该属性指定的构造函数。 



### Symbol.match 

对象的`Symbol.match`属性，指向一个函数。当执行`str.match(myObject)`时，如果该属性存在，会调用它，返回该方法的返回值。 



### Symbol.replace 



### Symbol.search

对象的`Symbol.search`属性，指向一个方法，当该对象被`String.prototype.search`方法调用时，会返回该方法的返回值。 



## Promise



 当一个 promise 被 resolve 时，会遍历之前通过 then 给这个 promise 注册的所有回调，将它们依次放入微任务队列中 



### Promise状态

Promise 有三种状态，pedding, resolve, reject

此状态不受外界影响，只能通过以上三个函数调用异步操作才能决定状态

- `resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去； 
- `reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。 

一旦决定了状态，就不可再改变，任何时候都可以得到这个结果 。





### Promise实例

`Promise`对象是一个构造函数，用来生成`Promise`实例 ，接收两个参数resolve, reject

创建promise实例

```
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

用then方法制定回调函数

```
promise.then(function(value) {		//resolve参数
  // success
}, function(error) {				//reject参数
  // failure
});
```



### then方法

为 Promise 实例添加状态改变时的回调函数。 

`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态变为`rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受`Promise`对象传出的值作为参数。 

一旦决定状态，就会触发then回调函数

如果一个promise对象的resolve触发了另一个promise对象，那么他本身的状态就会被覆盖，以后者为主

注意，调用`resolve`或`reject`并不会终结 Promise 的函数的执行。 

promise是微任务，会在事件循环的末尾执行 

`then`方法返回的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。 

```
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
```



### catch方法

`Promise.prototype.catch`方法是`.then(null, rejection)`或`.then(undefined, rejection)`的别名，用于指定发生错误时的回调函数。 

如果运行中抛出错误，就会被`catch`方法捕获。 

`reject`方法的作用，等同于抛出错误。 

Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获。 

不要在`then`方法里面定义 Reject 状态的回调函数（即`then`的第二个参数），总是使用`catch`方法。 

`catch`方法返回的还是一个 Promise 对象 

`catch`方法处理错误之后，返回的 Promise 对象 等同于resolve



### finally 方法

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。 

`finally`方法的回调函数不接受任何参数 ，是与状态无关的，不依赖于 Promise 的执行结果。 

`finally`方法总是会返回原来的值。 



### Promise.all方法 

`Promise.all`方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。 

```
const p = Promise.all([p1, p2, p3]);
```

`Promise.all`方法接受一个数组作为参数，`p1`、`p2`、`p3`都是 Promise 实例，如果不是，就会先调用下面讲到的`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。 



`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

（1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。

（2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。



注意，如果作为参数的 Promise 实例，自己定义了`catch`方法，那么它一旦被`rejected`，并不会触发`Promise.all()`的`catch`方法。 



### Promise.race

`Promise.race`方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。 

```
const p = Promise.race([p1, p2, p3]);
```

只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给`p`的回调函数。 



### Promise.resolve

有时需要将现有对象转为 Promise 对象，`Promise.resolve`方法就起到这个作用。 

```
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

```
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```



**（1）参数是一个 Promise 实例**

如果参数是 Promise 实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。

**（2）参数是一个thenable对象**

`Promise.resolve`方法会将这个对象转为 Promise 对象，然后就立即执行`thenable`对象的`then`方法 

**（3）参数不是具有then方法的对象，或根本就不是对象** 

如果参数是一个原始值，或者是一个不具有`then`方法的对象，则`Promise.resolve`方法返回一个新的 Promise 对象，状态为`resolved`。 

**（4）不带有任何参数** 

`Promise.resolve`方法允许调用时不带参数，直接返回一个`resolved`状态的 Promise 对象。

所以，如果希望得到一个 Promise 对象，比较方便的方法就是直接调用`Promise.resolve`方法。



### Promise.reject()

`Promise.reject(reason)`方法也会返回一个新的 Promise 实例，该实例的状态为`rejected`。 









Promise对象用于异步操作，它表示一个尚未完成且预计在未来完成的异步操作。 





回调函数缺点：

1. 多重嵌套，导致回调地狱
2. 代码跳跃，并非人类习惯的思维模式
3. 信任问题，你不能把你的回调完全寄托与第三方库，因为你不知道第三方库到底会怎么执行回调（多次执行）
4. 第三方库可能没有提供错误处理
5. 不清楚回调是否都是异步调用的（可以同步调用ajax，在收到响应前会阻塞整个线程，会陷入假死状态，非常不推荐）

1. `xhr.open("GET","/try/ajax/ajax_info.txt",false); //通过设置第三个async为false可以同步调用ajax`



Promise本身是一个状态机，具有pending（等待），resolve（决议），rejected （拒绝）这3个状态，



当请求发送没有得到响应的时候会pending状态，并且一个Promise实例的状态只能从pending => resolve 或者从 pending => reject，即当一个Promise实例从pending状态改变后，就不会再改变了（不存在resolve => reject 或 reject => resolve）

而Promise实例必须主动调用then方法，才能将值从Promise实例中取出来（前提是Promise不是pending状态），这一个“主动”的操作就是解决这个问题的关键，即第三方库做的只是把改变Promise的状态，而响应的值怎么处理，这是开发者主动控制的，这里就实现了控制反转，将原来第三方库的控制权转移到了开发者上



1. 第三方库可能没有提供错误处理

Promise的then方法会接受2个函数，第一个函数是这个Promise实例被resolve时执行的回调，第二个函数是这个Promise实例被reject时执行的回调，而这个也是开发者主动调用的

使用Promise在异步请求发送错误的时候，即使没有捕获错误，也不会阻塞主线程的代码

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

1. 不清楚回调是否都是异步调用的

Promise在设计的时候保证所有响应的处理回调都是异步调用的，不会阻塞代码的执行，Promise将then方法的回调放入一个叫微任务的队列中（MicroTask），保证这些回调任务都在同步任务执行完再执行，这部分同样也是事件循环的知识点，有兴趣的朋友可以深入研究一下



Promise的使用场景：ajax请求，回调函数，复杂操作判断。

Promise是ES6为了解决异步编程所诞生的。

异步操作解决方案：`Promise`、`Generator`、定时器（不知道算不算）、还有ES7的`async`



## Generator 

Generator 函数是一个状态机，封装了多个内部状态。

执行 Generator 函数会返回一个遍历器对象，返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

形式上，Generator 函数是一个普通函数，但是有两个特征:

- 一是，`function`关键字与函数名之间有一个星号；
- 二是，函数体内部使用`yield`表达式，定义不同的内部状态 

```
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
```

调用generator之后，用next()遍历他自身的每一个状态

```
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

 如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`。 

### yield 表达式

`yield`表达式后面的表达式，只有当调用`next`方法、内部指针指向该语句时才会执行 



| 区别   |                    |                          |      |      |
| ------ | ------------------ | ------------------------ | ---- | ---- |
| yield  | 具备记忆位置的功能 | 可执行多次，返回多个值   |      |      |
| return | 不具备             | 只能执行一次，返回一个值 |      |      |

`yield`表达式如果用在另一个表达式之中，必须放在圆括号里面。 

```
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}
```

`yield`表达式用作函数参数或放在赋值表达式的右边，可以不加括号。 

```
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```



由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的`Symbol.iterator`属性，从而使得该对象具有 Iterator 接口。 

```
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```



### next 方法的参数

`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值。 

```
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

由于`next`方法的参数表示上一个`yield`表达式的返回值，所以在第一次使用`next`方法时，传递参数是无效的。从语义上讲，第一个`next`方法用来启动遍历器对象  



### for...of 循环

一旦`next`方法的返回对象的`done`属性为`true`，`for...of`循环就会中止，且不包含该返回对象，所以上面代码的`return`语句返回的`6`，不包括在`for...of`循环之中。 

```
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

```
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



### Generator.prototype.throw()

`throw`方法可以接受一个参数，该参数会被`catch`语句接收，建议抛出`Error`对象的实例。 

```
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

`return`方法，可以返回给定的值，并且终结遍历 Generator 函数。

```
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



### yield* 表达式

`yield*`表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。 

```
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



`yield*`后面的 Generator 函数（没有`return`语句时），不过是`for...of`的一种简写形式，完全可以用后者替代前者。反之，在有`return`语句时，则需要用`var value = yield* iterator`的形式获取`return`语句的值。 

任何数据结构只要有 Iterator 接口，就可以被`yield*`遍历。 



### 作为对象属性的 Generator 函数

```
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

```
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

#### 异步操作的同步化表达

Generator 函数的暂停执行的效果，意味着可以把异步操作写在`yield`表达式里面，等到调用`next`方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在`yield`表达式下面，反正要等到调用`next`方法时再执行。 

```
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
var loader = loadUI();
// 加载UI
loader.next()

// 卸载UI
loader.next()
```



### 异步应用

Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。 





## async

async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里。 

`async`函数将 Generator 函数的星号（`*`）替换成`async`，将`yield`替换成`await` 

`async`函数自带执行器，返回值是 Promise 对象 ，可以使用`then`方法添加回调函数 

```
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```



`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。 

`async`函数内部抛出错误，会导致返回的 Promise 对象变为`reject`状态。抛出的错误对象会被`catch`方法回调函数接收到。 

`async`函数返回的 Promise 对象，必须等到内部所有`await`命令后面的 Promise 对象执行完，才会发生状态改变 

`await`命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。 

任何一个`await`语句后面的 Promise 对象变为`reject`状态，那么整个`async`函数都会中断执行。 



### 错误处理

如果`await`后面的异步操作出错，那么等同于`async`函数返回的 Promise 对象被`reject`。 

`await`命令后面的`Promise`对象，运行结果可能是`rejected`，所以最好把`await`命令放在`try...catch`代码块中。 



多个`await`命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。 

```
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```



如果确实希望多个请求并发执行，可以使用`Promise.all`方法。 



将 Generator 写法中的自动执行器，改在语言层面提供，不暴露给用户，因此代码量最少。 



### 异步遍历的接口

```
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );
```

调用遍历器的`next`方法，返回的是一个 Promise 对象。 

异步遍历器接口，部署在`Symbol.asyncIterator`属性上面。 



#### for await...of

`for...of`循环用于遍历同步的 Iterator 接口。新引入的`for await...of`循环，则是用于遍历异步的 Iterator 接口。 

```
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b
```

`createAsyncIterable()`返回一个拥有异步遍历器接口的对象，`for...of`循环自动调用这个对象的异步遍历器的`next`方法，会得到一个 Promise 对象。`await`用来处理这个 Promise 对象，一旦`resolve`，就把得到的值（`x`）传入`for...of`的循环体。 





## 十三、ES6 Module



## 

