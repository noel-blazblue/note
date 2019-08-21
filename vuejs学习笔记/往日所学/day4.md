## 一、Vue组件



#### 1.用props属性传值



- 使用v-bind将父组件的数据传递到子组件中：

```
<div id="app">
    <son :finfo="msg"></son>
</div>
```



- 在子组件使用`props`属性，定义父组件传递过来的数据

```
 var vm = new Vue({
      el: '#app',
      data: {
        msg: '这是父组件中的消息'
      },
      components: {
        son: {
          template: '<h1>这是子组件 --- {{finfo}}</h1>',
          props: ['finfo']
        }
      }
 });
```



#### 2.用this.$emit 方式来调用方法

把父组件的方法传入子组件	：			

```
<son @func="getMsg"></son>					//getMsg为父组件的一个方法，func为子组件接收的方法
```

子组件调用方法：

```
 Vue.component('son', {
      template: '#son', 
      methods: {
        sendMsg() {
          this.$emit('func', 'OK'); 			// 调用父组件传递过来的方法，同时把数据传递出去
        }
      }
    });
```



#### 3.使用 `this.$refs` 来获取元素和组件

用ref给元素定义一个名称：

```
 	  <!-- 使用 ref 获取元素 -->
      <h1 ref="myh1">这是一个大大的H1</h1>

      <hr>
      <!-- 使用 ref 获取子组件 -->
      <my-com ref="mycom"></my-com>
```

在vue实例中通过名称获取元素：

```
 var vm = new Vue({
      el: '#app',
      data: {},
      methods: {
        getElement() {
         
          console.log(this.$refs.myh1.innerText);		 // 通过 this.$refs 来获取元素
          
          console.log(this.$refs.mycom.name);			// 通过 this.$refs 来获取组件
        }
      }
 });
```



## 二、路由

#### 1.什么是路由

1. 对于普通的网站，所有的超链接都是URL地址，所有的URL地址都对应服务器上对应的资源；
2. 对于单页面应用程序来说，主要通过URL中的hash(#号)来实现不同页面之间的切换，同时，hash有一个特点：HTTP请求中不会包含hash相关的内容；所以，单页面程序中的页面跳转主要用hash实现；
3. 在单页面应用程序中，这种通过hash改变来切换页面的方式，称作前端路由（区别于后端路由）；



#### 2.使用 vue-router

需要导入 vue-router 组件类库：

```
<script src="./lib/vue-router-2.7.0.js"></script>
```

1.创建路由 router 实例，通过 routers 属性来定义路由匹配规则

```
 var router = new VueRouter({
      routes: [
        { path: '/login', component: login },					//component：组件名称
        { path: '/register', component: register }
      ]
 });
```

2.在vue实例中加载路由

```
    var vm = new Vue({
      el: '#app',
      router: router 			// 使用 router 属性来使用路由规则
    });
```

3.使用 router-link 组件来导航

```
<router-link to="/login">登录</router-link>
<router-link to="/register">注册</router-link>
```

4.使用 router-view 组件来显示匹配到的组件

```
<router-view></router-view>
```



#### 3.在路由规则中定义参数

1. 在规则中定义参数：

```
{ path: '/register/:id', component: register }
```

2. 在组件模版中，通过 `this.$route.params`来获取路由中的参数：

```
var register = Vue.extend({
      template: '<h1>	{{	this.$route.params.id	}}	</h1>'
    });
```



#### 4.使用 `children` 属性实现路由嵌套



```
 var router = new VueRouter({
      routes: [
        { path: '/', redirect: '/account/login' }, // 使用 redirect 实现路由重定向
        {
          path: '/account',
          component: account,
          children: [ 
            { path: 'login', component: login }, 	// 注意，子路由的开头位置，不要加 / 路径符
            { path: 'register', component: register }
          ]
        }
      ]
});
```



5.命名视图

在组件中传入一个对象，定义name对应什么组件

```
var router = new VueRouter({
      routes: [
        {
          path: '/', components: {
            ’default‘: header,
            ’a‘: sidebar,
            ’b‘: mainbox
          }
        }
      ]
});
```



在路由视图标签中，添加name

```
<router-view></router-view>
<router-view name="a"></router-view>
<router-view name="b"></router-view>
```

