`Reflect`对象与`Proxy`对象一样，也是 ES6 为了操作对象而提供的新 API。

`Reflect`对象一共有 13 个静态方法。

-   Reflect.apply(target, thisArg, args)
-   Reflect.construct(target, args)
-   Reflect.get(target, name, receiver)
-   Reflect.set(target, name, value, receiver)
-   Reflect.defineProperty(target, name, desc)
-   Reflect.deleteProperty(target, name)
-   Reflect.has(target, name)
-   Reflect.ownKeys(target)
-   Reflect.isExtensible(target)
-   Reflect.preventExtensions(target)
-   Reflect.getOwnPropertyDescriptor(target, name)
-   Reflect.getPrototypeOf(target)
-   Reflect.setPrototypeOf(target, prototype)