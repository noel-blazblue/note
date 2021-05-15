Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截

## handler

-   **get(target, propKey, receiver)**：
	-   拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
-   **set(target, propKey, value, receiver)**：
	-   拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
-   **has(target, propKey)**：
	-   拦截`propKey in proxy`的操作，返回一个布尔值。
-   **deleteProperty(target, propKey)**：
	-   拦截`delete proxy[propKey]`的操作，返回一个布尔值。
-   **ownKeys(target)**：
	-   拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。
-   **getOwnPropertyDescriptor(target, propKey)**：
	-   拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
-   **defineProperty(target, propKey, propDesc)**：
	-   拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
-   **preventExtensions(target)**：
	-   拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
-   **getPrototypeOf(target)**：
	-   拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
-   **setPrototypeOf(target, proto)**：
	-   拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
-   **isExtensible(target)**：
	-   拦截`Object.isExtensible(proxy)`，返回一个布尔值。
-   **apply(target, object, args)**：
	-   拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
-   **construct(target, args)**：
	-   拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。


### get()
`get`方法用于拦截某个属性的读取操作

```js
get: function(target, property, receiver) {}
```

参数：
- `target`： 目标对象。
- `property`： 被获取的属性名。
- `receiver`： Proxy或者继承Proxy的对象

返回值：
get方法可以返回任何值。

拦截：
该方法会拦截目标对象的以下操作:
-   访问属性: `proxy[foo]和` `proxy.bar`
-   访问原型链上的属性: `Object.create(proxy)[foo]`
-   `Reflect.get()`


```javascript
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, propertyKey, receiver) {
    if (propKey in target) {
      return target[propKey];
    } else {
      throw new ReferenceError("Prop name \"" + propKey + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误
```


如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。
```javascript
const target = Object.defineProperties({}, {
  foo: {
    value: 123,
    writable: false,
    configurable: false
  },
});

const handler = {
  get(target, propKey) {
    return 'abc';
  }
};

const proxy = new Proxy(target, handler);

proxy.foo
// TypeError: Invariant check failed
```

### set()
`set`方法用来拦截某个属性的赋值操作

参数：
- `target`： 目标对象。
- `property`： 将被设置的属性名或 `Symbol`
- `value`：新属性值。
- `receiver`：
最初被调用的对象。通常是 proxy 本身，但 handler 的 set 方法也有可能在原型链上，或以其他方式被间接地调用（因此不一定是 proxy 本身）。

**比如：**假设有一段代码执行 `obj.name = "jen"`， `obj` 不是一个 proxy，且自身不含 `name` 属性，但是它的原型链上有一个 proxy，那么，那个 proxy 的 `set()` 处理器会被调用，而此时，`obj` 会作为 receiver 参数传进来。

返回值：
`set()` 方法应当返回一个布尔值。
-   返回 `true` 代表属性设置成功。
-   在严格模式下，如果 `set()` 方法返回 `false`，那么会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常。

### has()
`has()`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是`in`运算符。

```js
const handler1 = {
  has(target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
```

参数：
- `target`： 目标对象.
- `prop`： 需要检查是否存在的属性.

返回值：
 `has` 方法返回一个 boolean 属性的值.

`has()`拦截只对`in`运算符生效，对`for...in`循环不生效，导致不符合要求的属性没有被`for...in`循环所排除。

### ownKeys()
`ownKeys()`方法用来拦截对象自身属性的读取操作。具体来说，拦截以下操作。

拦截：
-   `Object.getOwnPropertyNames()`
-   `Object.getOwnPropertySymbols()`
-   `Object.keys()`
-   `for...in`循环

返回值：
返回一个可枚举对象

```
var p = new Proxy({}, {
  ownKeys: function(target) {
    console.log('called');
    return ['a', 'b', 'c'];
  }
});

console.log(Object.getOwnPropertyNames(p)); // "called"
                                            // [ 'a', 'b', 'c' ]
```

### apply()
`apply`方法拦截函数的调用、`call`和`apply`操作
```js
var p = new Proxy(target, {
  apply: function(target, thisArg, argumentsList) {
  }
});
```

参数:
以下是传递给apply方法的参数，`this`上下文绑定在`handler`对象上.
- `target`： 目标对象（函数）。
- `thisArg`： 被调用时的上下文对象。
- `argumentsList`: 被调用时的参数数组。

返回值:
apply方法可以返回任何值。

```javascript
var twice = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments) * 2;
  }
};
function sum (left, right) {
  return left + right;
};
var proxy = new Proxy(sum, twice);
proxy(1, 2) // 6
proxy.call(null, 5, 6) // 22
proxy.apply(null, [7, 8]) // 30
```


### construct()
`construct()`方法用于拦截`new`命令

```javascript
const handler = {
  construct (target, args, newTarget) {
    return new target(...args);
  }
};
```


参数：
-   `target`：目标对象。
-   `args`：构造函数的参数数组。
-   `newTarget`：创造实例对象时，`new`命令作用的构造函数（下面例子的`p`）。

返回值：
`construct` 方法必须返回一个对象。


### deleteProperty()
`deleteProperty`方法用于拦截`delete`操作，如果这个方法抛出错误或者返回`false`，当前属性就无法被`delete`命令删除。

参数：
- `target`：目标对象。
- `property`： 待删除的属性名。

返回值：
`deleteProperty` 必须返回一个 `Boolean`

如果抛出一个错误或者返回一个`false`，那就会拦截删除操作。

### defineProperty()
`defineProperty()`方法拦截了`Object.defineProperty()`操作。

参数：
- `target`： 目标对象。
- `property`： 待检索其描述的属性名。
- `descriptor`： 待定义或修改的属性的描述符。

返回值：
`defineProperty` 方法必须以一个 `Boolean`

### getOwnPropertyDescriptor()
`getOwnPropertyDescriptor()`方法拦截`Object.getOwnPropertyDescriptor()`，返回一个属性描述对象或者`undefined`。

```
var p = new Proxy({ a: 20}, {
  getOwnPropertyDescriptor: function(target, prop) {
    console.log('called: ' + prop);
    return { configurable: true, enumerable: true, value: 10 };
  }
});

console.log(Object.getOwnPropertyDescriptor(p, 'a').value); // "called: a"
                                                            // 10
```

### getPrototypeOf()
`getPrototypeOf()`方法主要用来拦截获取对象原型。具体来说，拦截下面这些操作。
拦截：
-   `Object.prototype.__proto__`
-   `Object.prototype.isPrototypeOf()`
-   `Object.getPrototypeOf()`
-   `Reflect.getPrototypeOf()`
-   `instanceof`

```javascript
var proto = {};
var p = new Proxy({}, {
  getPrototypeOf(target) {
    return proto;
  }
});
Object.getPrototypeOf(p) === proto // true
```

返回值：
`getPrototypeOf()`方法的返回值必须是对象或者`null`，否则报错。另外，如果目标对象不可扩展（non-extensible）， `getPrototypeOf()`方法必须返回目标对象的原型对象。

### setPrototypeOf()
`setPrototypeOf()`方法主要用来拦截`Object.setPrototypeOf()`方法。

返回值：
该方法只能返回布尔值，否则会被自动转为布尔值。另外，如果目标对象不可扩展（non-extensible），`setPrototypeOf()`方法不得改变目标对象的原型。

```javascript
var handler = {
  setPrototypeOf (target, proto) {
    throw new Error('Changing the prototype is forbidden');
  }
};
var proto = {};
var target = function () {};
var proxy = new Proxy(target, handler);
Object.setPrototypeOf(proxy, proto);
// Error: Changing the prototype is forbidden
```

### isExtensible()
`isExtensible()`方法拦截`Object.isExtensible()`操作。

返回值：
该方法只能返回布尔值，否则返回值会被自动转为布尔值。

```javascript
var p = new Proxy({}, {
  isExtensible: function(target) {
    console.log("called");
    return true;
  }
});

Object.isExtensible(p)
// "called"
// true
```

### preventExtensions()
`preventExtensions()`方法拦截`Object.preventExtensions()`。该方法必须返回一个布尔值，否则会被自动转为布尔值。
这个方法有一个限制，只有目标对象不可扩展时（即`Object.isExtensible(proxy)`为`false`），`proxy.preventExtensions`才能返回`true`，否则会报错。

```javascript
var proxy = new Proxy({}, {
  preventExtensions: function(target) {
    console.log('called');
    Object.preventExtensions(target);
    return true;
  }
});

Object.preventExtensions(proxy)
// "called"
// Proxy {}
```

## 方法
### Proxy.revocable()
`Proxy.revocable()`方法返回一个可取消的 Proxy 实例。

```javascript
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

`Proxy.revocable()`方法返回一个对象，该对象的`proxy`属性是`Proxy`实例，`revoke`属性是一个函数，可以取消`Proxy`实例。上面代码中，当执行`revoke`函数之后，再访问`Proxy`实例，就会抛出一个错误。

`Proxy.revocable()`的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

## 实际题
### 使用ES6 的Proxy实现数组负索引
```js
const proxyArray = arr => {
    const length = arr.length;
    return new Proxy(arr, {
        get(target, key) {
            key = +key;
            while (key < 0) {
                key += length;
            }
            return target[key];
        }
    })
};
var a = proxyArray([1, 2, 3, 4, 5, 6, 7, 8, 9]);
console.log(a[1]);  // 2
console.log(a[-10]);  // 9
console.log(a[-20]);  // 8
```