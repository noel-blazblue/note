Math.ceil();  //向上取整。

Math.floor();  //向下取整。

Math.round();  //四舍五入。

Math.random();  //0.0 ~ 1.0 之间的一个伪随机数。【包含0不包含1】 //比如0.8647578968666494

Math.**ceil**(Math.random()\*10);      // 获取从1到10的随机**整数** ，取0的概率极小。

Math.**round**(Math.random());   //可均衡获取0到1的随机**整数**。

Math.**floor**(Math.random()\*10);  //可均衡获取0到9的随机**整数**。

Math.**round**(Math.random()\*10);  //基本均衡获取0到10的随机**整数**，其中**获取最小值0和最大值10的几率少一半**。

## API
### min()
-   Math.min 的参数是 0 个或者多个，如果多个参数很容易理解，返回参数中最小的。如果没有参数，则返回 Infinity，无穷大。
-   而 Math.max 没有传递参数时返回的是-Infinity.所以输出 false

## 运用

### 生成某个值内的随机数
```js
Math.ceil(num * Math.random())
```

### 生成[n,m]的随机整数
- 取差值 x ： `n - m`
- 生成差值内的随机数 c ：`x * Math.random()`
- 随机数再加上最小值：`c + m`


```js
return parseInt(Math.random() * (maxNum - minNum + 1) + minNum, 10);
```