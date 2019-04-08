this总是指向调用该方法的对象！



情况1：如果一个函数中有this，但是它没有被上一级的对象所调用，那么this指向的就是window，这里需要说明的是在js的严格版中this指向的不是window，但是我们这里不探讨严格版的问题，你想了解可以自行上网查找。

情况2：**如果一个函数中有this，这个函数有被上一级的对象所调用，那么this指向的就是上一级的对象。**

情况3：**如果一个函数中有this，这个函数中包含多个对象，尽管这个函数是被最外层的对象所调用，this指向的也只是它上一级的对象，**如果不相信，那么接下来我们继续看几个例子。



**全局环境的this**

- 非严格模式：this先指向undefined,然后再自动指向Window。

- 严格模式：this指向undefined




## 

## this五种绑定



this是一个指针，指向调用函数的对象



1. 默认绑定
2. 隐式绑定
3. 硬绑定
4. new绑定



### 默认绑定

默认绑定，在不能应用其它绑定规则时使用的默认规则 

```
function sayHi(){
    console.log('Hello,', this.name);
}
var name = 'YvetteLau';
sayHi();
```



### 隐式绑定

调用位置存在上下文对象

```
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
person.sayHi();
```

隐式绑定存在丢失情况，通常如果有回调函数，会产生一个函数作用域

```
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = person.sayHi;
Hi();
```



### 显式绑定

就是通过call,apply,bind的方式 ，显式的指定this所指向的对象 

```
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = person.sayHi;
Hi.call(person); 
```



### new绑定

> 使用new来调用函数，会自动执行下面的操作：

1. 创建一个新对象
2. 将构造函数的作用域赋值给新对象，即this指向这个新对象
3. 执行构造函数中的代码
4. 返回新对象



因此，我们使用new来调用函数的时候，就会新对象绑定到这个函数的this上。 

```
function sayHi(name){
    this.name = name;
	
}
var Hi = new sayHi('Yevtte');
console.log('Hello,', Hi.name);
```



## 绑定优先级

new绑定 > 显式绑定 > 隐式绑定 > 默认绑定 



例外：

如果我们将null或者是undefined作为this的绑定对象传入call、apply或者是bind,这些值在调用时会被忽略，实际应用的是默认绑定规则。 



## 箭头函数

箭头函数没有自己的this，箭头函数中的this继承于外层代码库中的this. 



（1）函数体内的this对象，继承的是外层代码块的this。

（2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

（3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

（4）不可以使用yield命令，因此箭头函数不能用作 Generator 函数。

（5）箭头函数没有自己的this，所以不能用call()、apply()、bind()这些方法去改变this的指向.

 

```
var obj = {
    hi: function(){
        console.log(this);
        return ()=>{
            console.log(this);
        }
    },
    sayHi: function(){
        return function() {
            console.log(this);
            return ()=>{
                console.log(this);
            }
        }
    },
    say: ()=>{
        console.log(this);
    }
}
let sayHi = obj.sayHi();
let fun1 = sayHi(); //输出window
fun1();             //输出window

let fun2 = sayHi.bind(obj)();//输出obj
fun2();                      //输出obj
```





> #### 1. 如何准确判断this指向的是什么？

1. 函数是否在new中调用(new绑定)，如果是，那么this绑定的是新创建的对象。
2. 函数是否通过call,apply调用，或者使用了bind(即硬绑定)，如果是，那么this绑定的就是指定的对象。
3. 函数是否在某个上下文对象中调用(隐式绑定)，如果是的话，this绑定的是那个上下文对象。一般是obj.foo()
4. 如果以上都不是，那么使用默认绑定。如果在严格模式下，则绑定到undefined，否则绑定到全局对象。
5. 如果把Null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。
6. 如果是箭头函数，箭头函数的this继承的是外层代码块的this。

 

 

##  call，apply，bind

## 

```
obj.myFun.call(db,'成都','上海')；　　　　 // 德玛 年龄 99 来自 成都去往上海

obj.myFun.apply(db,['成都','上海']);		 // 德玛 年龄 99 来自 成都去往上海

obj.myFun.bind(db,'成都','上海')(); 		// 德玛 年龄 99 来自 成都去往上海
```



call（）：第一个参数是this值没有变化，变化的是**其余参数都直接传递给函数**。在使用call（）方法时，**传递给函数的参数必须逐个列举出来**。

apply（）：传递给函数的是**参数数组**

bind（）：返回的是一个函数，最好直接调用（）。



参数为3个（包含3）以内时，优先使用 call 方法进行事件的处理。而当参数过多（多余3个）时，才考虑使用 apply 方法。 如果不考虑性能，代码层面用apply来写会简洁很多。 

call要比apply性能快：

- 由于 apply 中定义的参数格式（数组），使得被调用之后需要做更多的事，需要将给定的参数格式改变（步骤8）。
- 同时也有一些对参数的检查（步骤2），在 call 中却是不必要的。
- 另外一个很重要的点：在 apply 中不管有多少个参数，都会执行循环，也就是步骤 6-8，在 call 中也就是对应步骤3 ，是有需要才会被执行。

 

 不管我们给函数 `bind` 几次，`fn` 中的 `this` 永远由第一次 `bind` 决定 



![img](https://user-gold-cdn.xitu.io/2018/11/15/16717eaf3383aae8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 