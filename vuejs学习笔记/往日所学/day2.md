## 一、过滤器

#### 1.全局过滤器

引用：

```
{{ item.ctime | dateFormat() }}			
```

item.ctime		是变量
|				是管道符
dateFormat()		是过滤器名称

定义：

```
Vue.filter('dateFormat', function (dateStr, pattern = "") {
  
      var dt = new Date(dateStr)

      var y = dt.getFullYear()
      var m = dt.getMonth() + 1
      var d = dt.getDate()

      if (pattern.toLowerCase() === 'yyyy-mm-dd') {
        return `${y}-${m}-${d}`
      } else {
        var hh = dt.getHours()
        var mm = dt.getMinutes()
        var ss = dt.getSeconds()

        return `${y}-${m}-${d} ${hh}:${mm}:${ss}`
      }
    })
```



#### 2.私有过滤器

```
var vm2 = new Vue({
      el: '#app2',
      data: {
        dt: new Date()
      },
      methods: {},
      filters: { // 定义私有过滤器    过滤器有两个 条件  【过滤器名称 和 处理函数】
        // 过滤器调用的时候，采用的是就近原则，如果私有过滤器和全局过滤器名称一致了，这时候 优先调用私有过滤器
        dateFormat: function (dateStr, pattern = '') {
          // 根据给定的时间字符串，得到特定的时间
          var dt = new Date(dateStr)

          //   yyyy-mm-dd
          var y = dt.getFullYear()
          var m = (dt.getMonth() + 1).toString().padStart(2, '0')
          var d = dt.getDate().toString().padStart(2, '0')

          if (pattern.toLowerCase() === 'yyyy-mm-dd') {
            return `${y}-${m}-${d}`
          } else {
            var hh = dt.getHours().toString().padStart(2, '0')
            var mm = dt.getMinutes().toString().padStart(2, '0')
            var ss = dt.getSeconds().toString().padStart(2, '0')

            return `${y}-${m}-${d} ${hh}:${mm}:${ss} ~~~~~~~`
          }
        }
      }
```



## 二、自定义事件修饰符和指令

#### 1.自定义全局

- 按键修饰符

```
  Vue.config.keyCodes.f2 = 113
```

 

- 指令			

```
 Vue.directive('focus', {
      bind: function (el) { },
      inserted: function (el) { },
      updated: function (el) { }
    })
```

使用：

```
v-focus	
```

bind：每当指令绑定到元素上的时候，会立即执行这个 bind 函数，只执行一次 

inserted：表示元素 插入到DOM中的时候，会执行 inserted 函数【触发1次】 

updated：当VNode更新的时候，会执行 updated， 可能会触发多次 



+ 在每个 函数中，第一个参数，永远是 el ，表示 被绑定了指令的那个元素，这个 el 参数，是一个原生的JS对象 
+ 在元素 刚绑定了指令的时候，还没有 插入到 DOM中去，这时候，调用 focus 方法没有作用 
+ 因为，一个元素，只有插入DOM之后，才能获取焦点 

#### 2.自定义私有

要在new实例中

```
    var vm2 = new Vue({
      el: '#app2',
      data: {
        dt: new Date()
      },
      methods: {},
      directives: { 								// 自定义私有指令
        'fontweight': {								 // 设置字体粗细的
          bind: function (el, binding) {
            el.style.fontWeight = binding.value
          }
        }
      }
    })
```

简写：

```
'fontsize': function (el, binding) { 
    el.style.fontSize = parseInt(binding.value) + 'px'
}
```

注意：这个 function 等同于 把 代码写到了 bind 和 update 中去



## 三、生命周期

#### 1.生命周期：从Vue实例创建、运行、到销毁期间，总是伴随着各种各样的事件，这些事件，统称为生命周期！

- 生命周期钩子：就是生命周期事件的别名而已；

- 生命周期钩子 = 生命周期函数 = 生命周期事件

  

  #### 2.生命周期函数

- 创建期间的生命周期函数：
  - **beforeCreate：**实例刚在内存中被创建出来，此时，还没有初始化好 data 和 methods 属性
  - **created：**实例已经在内存中创建OK，此时 data 和 methods 已经创建OK，此时还没有开始 编译模板+ + 
  - **beforeMount：**此时已经完成了模板的编译，但是还没有挂载到页面中
  - **mounted：**此时，已经将编译好的模板，挂载到了页面指定的容器中显示
- 运行期间的生命周期函数：
  - **beforeUpdate：**状态更新之前执行此函数， 此时 data 中的状态值是最新的，但是界面上显示的 数据还是旧的，因为此时还没有开始重新渲染DOM节点
  - **updated：**实例更新完毕之后调用此函数，此时 data 中的状态值 和 界面上显示的数据，都已经完成了更新，界面已经被重新渲染好了！
- 销毁期间的生命周期函数：
  - **beforeDestroy：**实例销毁之前调用。在这一步，实例仍然完全可用。
  - **destroyed：**Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。



## 四、 get, post, jsonp请求

注：需要引入 `vue-resource` 的脚本文件；

#### 1.发送get请求：

```
this.$http.get('http://127.0.0.1:8899/api/getlunbo').then(res => {
    console.log(res.body);
})
```

#### 2.发送post请求：

```
this.$http.post(url, { name: 'zs' }, { emulateJSON: true }).then(res => {
    console.log(res.body);
});
```

- 参数1： 要请求的URL地址
- 参数2： 要发送的数据对象
- 参数3： 指定post提交的编码类型为 application/x-www-form-urlencoded

#### 3,发送JSONP请求获取数据：

```
this.$http.jsonp(url).then(res => {
    console.log(res.body);
 });
```

