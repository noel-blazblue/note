## 一、webpack 中安装vue依赖

在cmd中安装：

```
"vue": "^2.5.17",

"vue-html-loader": "^1.2.4",

"vue-loader": "^15.4.2",

"vue-router": "^3.0.2",

"vue-style-loader": "^4.1.2",

"vue-template-compiler": "^2.5.17",



"vue-preview": "^1.1.3",

"vue-resource": "^1.5.1",

"vue2-preview": "^1.0.2"
```



在webpack.config.js中配置匹配规则：

```
 {test:/\.vue$/,use:'vue-loader'},
```



## 二、vue功能不完整问题

#### 问题：

在 webpack 中， 使用 `import Vue from 'vue' `导入的 Vue 构造函数，功能不完整，只提供了 runtime-only 的方式 。



#### 导入包的查找规则：

1. 找 项目根目录中有没有 node_modules 的文件夹
2. 在 node_modules 中 根据包名，找对应的 vue 文件夹
3. 在 vue 文件夹中，找 一个叫做 package.json 的包配置文件
4. 在 package.json 文件中，查找 一个 main 属性【main属性指定了这个包在被加载时候，的入口文件】



#### 解决办法1

在导入时指定路径为：

`import Vue from '../node_modules/vue/dist/vue.js' `



#### 解决办法2

在webpack.config.js添加

```
  resolve: {
    alias: { 
        "vue$": "vue/dist/vue.js"
    }
  }
```



## 三、导入和导出

### 1.导出

#### vue后缀文件

```
<template>
  <div>
    <h1>这是登录组件，使用 .vue 文件定义出来的 --- {{msg}}</h1>
  </div>
</template>
<script>
</script>
<style>
</style>
```

- template为模板
- script为执行语句
- style为样式



#### 通过export default导出

```
<script>
export default {
  data() {
    return {
      msg: "123"
    };
  },
  methods: {
    show() {
      console.log("调用了 login.vue 中的 show 方法");
    }
  }
};
</script>
```



#### 通过export 导出

```
export var title = '小星星'
export var content = '哈哈哈'
```

即会将变量导出，可多次导出

### 2.导入

#### 通过import导入export default

```
import login from './login.vue'
```

```
var vm = new Vue({
  el: '#app',
  data: {
    msg: '123'
  },
  render: c => c(login)
})
```

即会将组件渲染到#app页面中



#### 通过import导入export 

```
import ｛｛title｝｝ from './login.vue'
```

导入的接收名必须与导出的文件中的变量名相同



## 四、路由渲染组件

### 1.配置路由

#### 导入依赖包

```
import Vue from 'vue'
import VueRouter from 'vue-router'
```

#### 手动安装路由

```
Vue.use(VueRouter)
```

#### 导入组件

```
import app from './App.vue'
import account from './main/Account.vue'
import goodslist from './main/GoodsList.vue'
```

#### 创建路由对象

```
var router = new VueRouter({
  routes: [
    { path: '/account', component: account },
    { path: '/goodslist', component: goodslist }
  ]
})
```

挂载路由并用render渲染

```
var vm = new Vue({
  el: '#app',
  render: c => c(app), // render 会把 el 指定的容器中，所有的内容都清空覆盖，所以 不要 把 路由的 router-view 和 router-link 直接写到 el 所控制的元素中
  router // 4. 将路由对象挂载到 vm 上
})
```



### 2.配置组件

#### 在模板中创建路由导航和路由视图

```
<template>
  <div>
    <h1>这是 App 组件</h1>
    <router-link to="/account">Account</router-link>
    <router-link to="/goodslist">Goodslist</router-link>
    <router-view></router-view>
  </div>
</template>
```



## 五、路由嵌套

路由配置单独放到一个router.js文件中



### 1.配置路由

#### 导入依赖

```
import VueRouter from 'vue-router'
```

#### 导入组件

```
import account from './main/Account.vue'
import goodslist from './main/GoodsList.vue'
import login from './subcom/login.vue'
import register from './subcom/register.vue'
```

#### 创建路由对象

```
var router = new VueRouter({
  routes: [
    {
      path: '/account',
      component: account,
      children: [
        { path: 'login', component: login },
        { path: 'register', component: register }
      ]
    },
    { path: '/goodslist', component: goodslist }
  ]
})
```

注：路由嵌套的路径中不带/

#### 把路由导出出去

```
export default router
```



### 2.配置组件

一级组件设置路由导航和视图

```
    <router-link to="/account">Account</router-link>
    <router-link to="/goodslist">Goodslist</router-link>
    <router-view></router-view>
```

二级组件设置路由嵌套中的子组件导航

```
    <router-link to="/account/login">登录</router-link>
    <router-link to="/account/register">注册</router-link>
    <router-view></router-view>
```

