## 1数据类型

var obj = //		输出一个正则



var a=b=3 相当于 var a = 3； b = 3； 



null === undefined

//false

null == undefined

//true



## 2优先级

加号优先级高于 三目运算。低于括号 



'1' === true

//false

'1' == true

//true



## 概念

dtd  文档类型声明/定义 



不支持冒泡的事件：scroll,blur,mouseleave



## 模块化

模块化优点：

- 解决命名冲突
- 提供复用性
- 提高代码可维护性



### AMD 和 CMD



### CommonJS

```
// a.js
module.exports = {
    a: 1
}
// or 
exports.a = 1

// b.js
var module = require('./a.js')
module.a // -> log 1
```



### ES Module



## API

### typeof 和 instanceof 的区别

typeof 是判断底层的二进制来确定数据类型，因此在判断null和引用数据类型的时候都会得出object这个结果

- **000：对象**
- 1：整数
- 010：浮点数
- 100：字符串
- 110：布尔



instanceof 是在原型链中寻找，所以可以判断出引用数据类型



## 线程



```
let a = () => {
  setTimeout(() => {
    console.log("任务队列函数1");
  }, 0);
  for (let i = 0; i < 5000; i++) {
    console.log("a的for循环");
  }
  console.log("a事件执行完");
};

let b = () => {
  setTimeout(() => {
    console.log("任务队列函数2");
  }, 0);
  for (let i = 0; i < 5000; i++) {
    console.log("b的for循环");
  }
  console.log("b事件执行完");
};
let c = () => {
  setTimeout(() => {
    console.log("任务队列函数3");
  }, 0);
  for (let i = 0; i < 5000; i++) {
    console.log("c的for循环");
  }
  console.log("c事件执行完");
};
a();
b();
c();

//打印结果
5000 a的for循环
 a事件执行完
5000 b的for循环
 b事件执行完
5000 c的for循环
 c事件执行完
undefined
 任务队列函数1
 任务队列函数2
 任务队列函数3
```







#### 缓存

对于一个数据请求来说，可以分为发起网络请求、后端处理、浏览器响应三个步骤。浏览器缓存可以帮助我们在第一和第三步骤中优化性能。比如说直接使用缓存而不发起请求，或者发起了请求但后端存储的数据和前端一致，那么就没有必要再将数据回传回来，这样就减少了响应数据。 



#### 请给出异步加载js方案，不少于两种？

async、defer 



#### 比较 HTML XML XHTML 和 JSON

- 我们最熟悉的就是 HTML（HyperText Markup Language / 超文本标记语言），用来描述和定义 网络内容的标记语言，超文本的意思是说，除了能标记本文，还能标记 图片，视频，链接 等其他内容
- XML（Extensible Markup Language / 可扩展标记语言），表现就是给一堆文档加上标签，说明里面的数据是什么意思，方便存储、传输、分享数据。和 HTML 的区别是 HTML 的标签是预定义的，XML 是可扩展的 
- XHTML: Extensible Hypertext Markup Language / 可扩展超文本标记语，其实就是 HTML 的严格语法形式，约定了 属性名和元素名小写，属性名必需加引号，空元素必需关闭，布尔类型必需加属性值
- JSON（Javascript Object Notation）轻量级的数据交换格式，由键值对组成，数据格式比较简单, 易于读写, 格式都是压缩的, 占用带宽小



#### 面向对象和面向过程的区别



- 面向过程就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了。
- 面向对象是把构成问题事务分解成各个对象，建立对象的目的不是为了完成一个步骤，而是为了描叙某个事物在整个解决问题的步骤中的行为。



## 4数据结构

#### null和undefined的差异

- 在 if判断语句中,值都默认为 false
- 大体上两者都是代表无,具体看差异 差异:
- null转为数字类型值为0,而undefined转为数字类型为 NaN(Not a Number)
- undefined是代表调用一个值而该值却没有赋值,这时候默认则为undefined
- null是一个很特殊的对象,最为常见的一个用法就是作为参数传入(说明该参数不是对象)
- 设置为null的变量或者对象会被内存收集器回收



null

（1） 作为函数的参数，表示该函数的参数不是对象。

（2） 作为对象原型链的终点。



#### require和import的区别

`require`与`import`的区别

- `require`支持 **动态导入**，`import`不支持，正在提案 (babel 下可支持)
- `require`是 **同步** 导入，`import`属于 **异步** 导入
- `require`是 **值拷贝**，导出值变化不会影响导入值；`import`指向 **内存地址**，导入值会随导出值而变化

 

####  闭包的作用

1. 封装私有变量
2. 模仿块级作用域(ES5中没有块级作用域)
3. 实现JS的模块



#### 堆栈溢出

每次执行代码时，都会分配一定尺寸的栈空间（Windows系统中为1M），每次方法调用时都会在栈里储存一定信息（如参数、局部变量、返回值等等） ，成千上万个此类空间累积起来，自然就超过线程的栈空间了。 

解决办法：

- 异步
  - 利用异步的特性，把执行函数放置到事件队列中，就会先执行同步代码，于是函数就可以退出，从而清空堆栈。

1. 闭包





## 5编程题

#### 1.判断一个字符串中出现次数最多的字符，统计这个次数。

```
        var str = 'fafwefaejkhvlajk'
        var obj = {}
        for(let i = 0; i < str.length; i ++){
            var key = str[i]
            typeof obj[key] === 'undefined' ? obj[key] = 1 : obj[key]++
        }
        var max = -1
        var max_key = null
        for(let key in obj){
            if(max < obj[key]){
                max = obj[key]
                max_key = key
            }
        }
        document.write('字符是:' + max_key + '出现次数是:' + max)
```



#### 2.判断字符串是否是这样组成的，第一个必须是字母，后面可以是字母、数字、下划线，总长度为5-20

```
var reg = /^[a-zA-Z][a-zA-Z_0-9]{4,19}$/
console.log(reg.test("11a__a1a__a1a__a1a__"))
```



#### 3.编写一个JavaScript函数，实时显示当前时间，格式‘年-月-日 时：分：秒’?

```
    function getTime(){
      var nowDate = new Date();
      var year = nowDate.getFullYear();
      var month = (nowDate.getMonth() + 1) > 10 ? nowDate.getMonth() + 1 : '0' + (nowDate.getMonth() + 1);
      var day = nowDate.getDate() > 10 ? nowDate.getDate() : '0' + nowDate.getDate();
      var hour = nowDate.getHours() > 10 ? nowDate.getHours() : (nowDate.getHours() == 0 ? 24 : '0' + nowDate.getHours());
      var minutes = nowDate.getMinutes() > 10 ? nowDate.getMinutes() : '0' + nowDate.getMinutes();
      var seconds = nowDate.getSeconds() > 10 ? nowDate.getSeconds() : '0' + nowDate.getSeconds();
      var str= year +"-" + month + "-" + day + " " + hour + ":" + minutes + ":" + seconds;
      document.getElementById("show").value = str;
    }
    window.setInterval("getTime()", 1000);
```



#### 4.编写一个JavaScript函数parseQueryString，他的用途是把URL参数解析为一个对象，如：var url=“[witmax.cn/index.php?k…](https://link.juejin.im/?target=http%3A%2F%2Fwitmax.cn%2Findex.php%3Fkey0%3D0%26key1%3D1%26key2%3D2%25E2%2580%259D%25EF%25BC%259B)

```
        function parseQueryString(url) {
            var result = {}
            var arr = url.split('?')
            if(arr.length <= 1) return result
            arr = arr[1].split('&')
            arr.forEach(item => {
                let a = item.split('=')
                result[a[0]] = a[1]
            })
            return result
        }
        var url = "http://witmax.cn/index.php?key0=0&key1=1&key2=2"
        var obj = parseQueryString(url)
        console.log(obj)
```



#### 5.在IE6.0下面是不支持position：fixed的，请写一个JS使用固定在页面的右下角。

```
    window.onscroll = window.onresize = window.onload = () => {
        var getDiv = document.getElementById('box');
        var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
        getDiv.style.left = document.documentElement.clientWidth - getDiv.offsetWidth + 'px';
        getDiv.style.top = document.documentElement.clientHeight - getDiv.offsetHeight + 			scrollTop + 'px';
    }
```



#### 6.请实现，鼠标移到页面中的任意标签，显示出这个标签的基本矩形轮廓。

```
function mouseBorder(t) {
    var c = t.childNodes
    for(let i = 0; i < c.length; i ++){
        var d = c[i]
        if(d.nodeType == 1) {
            d.onmouseover = function() {
                this.style.border = '1px solid red'
            }
            d.onmouseout = function() {
                this.style.border = ''
            }
            mouseBorder(d)
        }
    }
}
mouseBorder(document.body);
```



#### 8.代码实现instanceof

```
function new_instance_of(leftVaule, rightVaule) { 
    let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
    leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
    while (true) {
    	if (leftVaule === null) {
            return false;	
        }
        if (leftVaule === rightProto) {
            return true;	
        } 
        leftVaule = leftVaule.__proto__ 
    }
}
```



#### 9.如何判断一个变量是对象还是数组

```
function isObjArr(value){
     if (Object.prototype.toString.call(value) === "[object Array]") {
            console.log('value是数组');
       }else if(Object.prototype.toString.call(value)==='[object Object]'){
            console.log('value是对象');
      }else{
          console.log('value不是数组也不是对象')
      }
}

```



#### 10.数组去重的方法

- 1.遍历添加到一个新的数组

```
function unique(arr) {
  var newArr = []
  arr.forEach(item => {
    if(newArr.indexOf(item) < 0){
      newArr.push(item)
    }
  });
  return newArr
}
```



- 2.set结构去重

```
let unique= [...new Set(array)];
//es6 Set数据结构类似于数组，成员值是唯一的，有重复的值会自动去重。
//Set内部使用===来判断是否相等，类似'1'和1会两个都保存，NaN和NaN只会保存一个
```



- 3、遍历，将数组的值添加到一个对象的属性名里 

```
function unique(arr){
  var obj = {}
  arr.forEach(item => {
    obj[item] = 0
  });
  return Object.keys(obj)
}
```

**注意：**这个方法会将 number,NaN,undefined,null，**变为字符串形式，因为对象的属性名就是一个字符串**，根据需求来吧，想想还是Set去重最简单也最有效。



#### 11.翻转一个字符串

```
let b=[...str].reverse().join("");//drow olleh
```



#### 12.不借助中间变量交换两个数的值

```
let a = 1
let b = 2
a = a + b
b = a - b
a = a - b

void switch(int* p1, int* p2)
{
    *p1 = *p1 + *p2;   // 改变存储方式，使得*p1保存两个信息
    *p2 = *p1 - *p2;   // 取出 原来的 *p1 信息，存入 *p2 中
    *p1 = *p1 - *p2;   // 取出原来的 *p2 信息，存入 *p1 中
}

```



#### 13.【正则】请从`2017-05-15T09:10:23 Europe/Paris`提取出结果`["2017","05","15","09","10","23"]`

```
let str = '2017-05-15T09:10:23 Europe/Paris';
let arr = str.match(  /\d{1,}/g); 
//match会返回一个数组，
// \d 查找数字  
// {1,} 表示至少重复几次 
// /g表示全局搜索
```



#### 14.判断一个数是否为整数

```
    function isIntefer(x){
        return x%1===0; //返回布尔
    }
```

