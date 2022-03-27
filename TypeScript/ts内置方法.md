## Partial
Partial 将属性变为可选属性。
```ts
interface iUser {
  name: string;
  age: number;
}
interface iOptionUser {
  name?: string;
  age?: number;
}
```

实现：
```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

但是 Partial 有个局限性，就是只支持处理第一层的属性

可以看到，第二层以后的就不会处理了，如果要处理多层，就可以自己通过 [Conditional Types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html) 实现一个更强力的 Partial

```ts
export type PowerPartial<T> = {
     // 如果是 object，则递归类型
    [U in keyof T]?: T[U] extends object
      ? PowerPartial<T[U]>
      : T[U]
};
```



## Required

将属性变成必须。

实现：
```ts
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

其中 `-?` 是代表移除 `?` 这个 modifier 的标识。



## Pick

可以将某个类型中的子属性取出
```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```



## Record

根据 `k` 中所有属性名，赋予 `T` 为值

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```



## Exclude

排除某个类型

```ts
// node_modules/typescript/lib/lib.es5.d.ts

type Exclude<T, U> = T extends U ? never : T;
```

```ts
type T00 = Exclude<
	"a" | "b" | "c" | "d", 
	"a" | "c" | "f"
>;  // "b" | "d"
```

可以和 `Pick` 搭配使用，排除某个属性

```ts
// correct.
interface NewChicken extends Pick<Chicken, Exclude<keyof Chicken, 'name'>> {
  name: number;
}
```



## ReturnType

获取函数的返回类型
```ts
// node\_modules/typescript/lib/lib.es5.d.ts

type ReturnType<T extends (...args: any\[\]) \=> any\> \= T extends (...args: any\[\]) \=> infer R ? R : any;
```



```ts
function TestFn() {
  return '123123';
}

type T01 = ReturnType<typeof TestFn>; // string
```



## ThisType

指定 `this` 对象类型

```ts
interface Person {
    name: string;
    age: number;
}

const obj: ThisType<Person> = {
  dosth() {
    this.name // string
  }
}
```



## NonNullable

过滤类型中的 null 及 undefined 类型。

```ts
type T22 = '123' | '222' | null;
type T23 = NonNullable<T22>; // '123' | '222'
```

