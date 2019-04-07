## 一、Promise



### 概念

1. Promise 是一个 构造函数，既然是构造函数， 那么，我们就可以  new Promise() 得到一个 Promise 的实例；

2. 在 Promise 上，有两个函数，分别叫做 resolve（成功之后的回调函数） 和 reject（失败之后的回调函数）

3. 在 Promise 构造函数的 Prototype 属性上，有一个 .then() 方法，也就说，只要是 Promise 构造函数创建的实例，都可以访问到 .then() 方法

4. Promise 表示一个 异步操作；每当我们 new 一个 Promise 的实例，这个实例，就表示一个具体的异步操作；

5. 既然 Promise 创建的实例，是一个异步操作，那么，这个 异步操作的结果，只能有两种状态：

     5.1 状态1： 异步执行成功了，需要在内部调用 成功的回调函数 resolve 把结果返回给调用者；

     5.2 状态2： 异步执行失败了，需要在内部调用 失败的回调函数 reject 把结果返回给调用者；

     5.3 由于 Promise 的实例，是一个异步操作，所以，内部拿到 操作的结果后，无法使用 return 把操作的结果返回给调用者； 这时候，只能使用回调函数的形式，来把 成功 或 失败的结果，返回给调用者；

6. 我们可以在 new 出来的 Promise 实例上，调用 .then() 方法，【预先】 为 这个 Promise 异步操作，指定 成功（resolve） 和 失败（reject） 回调函数；

 

### 用法

#### 1.定义一个Promise函数

```
function getFileByPath(fpath) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fpath, 'utf-8', (err, dataStr) => {
      if (err) return reject(err)
      resolve(dataStr)
    })
  })
}
```

- `new Promise` 被创建之后，就会立即执行，所以需要放在一个函数中
- 在函数中，把Promise对象`return` 
- 在Promise对象中，执行操作。
- `resolve ` 成功回调函数
- `reject`失败回调函数



#### 2.调用

```
getFileByPath('./files/1.txt')
  .then(function (data) {
    console.log(data)
    return getFileByPath('./files/22.txt')
  })
  .then(function (data) {
    console.log(data)
    return getFileByPath('./files/3.txt')
  })
  .then(function (data) {
    console.log(data)
  })
```

- `.then`可接收Promise对象
- 在`then`中，可操作Promise对象的成功回调和失败回调
- 在执行末尾，`return`一个Promise实例的函数，就能再次执行



通过`.catch`捕获异常，终止回调

```
  .catch(function (err) { // catch 的作用： 如果前面有任何的 Promise 执行失败，则立即终止所有 promise 的执行，并 马上进入 catch 去处理 Promise中 抛出的异常；
    console.log('这是自己的处理方式：' + err.message)
  })
```



#### Jquery中使用Promise方法

```
      $('#btn').on('click', function () {
        $.ajax({
          url: './data.json',
          type: 'get',
          dataType: 'json'
        })
          .then(function (data) {
            console.log(data)
          })
      })
```



## 二、新闻列表

### 提前渲染数据

#### 1.定义一个接收数据的数组

```
  data() {
    return {
      newslist: [] // 新闻列表
    };
  },
```

#### 2.定义一个获取数据的方法

​	在其中指定获取数据后，保存到变量内

```
    getNewsList() {
      // 获取新闻列表
      this.$http.get("api/getnewslist").then(result => {
        if (result.body.status === 0) {
          // 如果没有失败，应该把数据保存到 data 上
          this.newslist = result.body.message;
        } else {
          Toast("获取新闻列表失败！");
        }
      });
    }
```

#### 3.在Vue实例创始之后调用方法

```
  created() {
    this.getNewsList();
  }
```

#### 4.渲染到组件模版中

```
<template>
     <span>发表时间：{{ item.add_time | dateFormat }}</span>
     <span>点击：{{item.click}}次</span>
</template>
```



### 根据ID传值

#### 1.父组件中把ID值绑定到路由导航中

```
<router-link :to="'/home/newsinfo/' + item.id">
/router-link>
```

#### 2.在路由匹配中通过属性绑定形式接收ID值

​	`:id`可接收路由中传过来的值

```
{ path: '/home/newsinfo/:id', component: NewsInfo }
```

3. #### 在子组件中通过`props`接收参数

```
props: ["id"]
```

#### 4.即可通过`this.id`访问

```
.get("api/getcomments/" + this.id + "?pageindex=" + this.pageIndex)
```



### 点击加载下一页

1.定义一个页数变量

```
  data() {
    return {
      pageIndex: 1, // 默认展示第一页数据
    };
  }
```

2.定义一个增加页数并调用数据渲染的方法

```
    getMore() {
      this.pageIndex++;
      this.getComments();
    }
```

3.在`getComments()`方法中，将新数据追加到变量中

```
this.comments = this.comments.concat(result.body.message);
```

3.绑定事件到组件模板中	

```
<mt-button type="danger" size="large" plain @click="getMore">加载更多</mt-button>
```

