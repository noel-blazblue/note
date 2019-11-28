## Vue生命周期钩子

- beforeCreate：此时获取不到prop和data中的数据；
- created：可以获取到prop和data中的数据； 
- beforeMount：获取到了VDOM; 
- mounted：VDOM解析成了真实DOM; 
- beforeUpdate：在更新之前调用； 
- updated：在更新之后调用；
- keep-alive：切换组件之后，组件放进activated，之前的组件放进deactivated；
- beforeDestory：在组件销毁之前调用，可以解决内存泄露的问题，如setTimeout和setInterval造成的问题。
- destory：组件销毁之后调用。

 

### beforeCreate

```
callHook(vm, 'beforeCreate')
// 初始化 inject
// 初始化 props、methods、data、computed 和 watch
// 初始化 provide
callHook(vm, 'created')
// 挂载实例 vm.$mount(vm.$options.el)
```



v-show和v-if指令的共同点和不同点？

```
  1.v-if是删除/添加Dom标签，不占据文档位置,v-show切换css的display属性，控制显示隐藏，还会占据文档位置。
  2.v-if会删除dom标签所以v-if性能消耗会高一些，需要频繁切换的话，使用v-show会好一点。
```

 

`<keep-alive></keep-alice>`的作用的是什么？

```
  `<keep-alive>`是Vue的内置组件，能在组件切换过程中将状态保留在内存中，防止重复渲染DOM。
```



## Vuex

![vuexçå·¥ä½æµç¨](https://user-gold-cdn.xitu.io/2017/12/9/1603ae858f7da6cd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

Vuex 是用于 Vue.js 应用的状态管理库，为应用中的所有组件提供集中式的状态存储与操作，保证了所有状态以可预测的方式进行修改。 



1. 在vue组件里面，通过dispatch来触发actions提交修改数据的操作。
2. 然后再通过actions的commit来触发mutations来修改数据。
3. mutations接收到commit的请求，就会自动通过Mutate来修改state（数据中心里面的数据状态）里面的数据。
4. 最后由store触发每一个调用它的组件的更新

Vuex的作用：项目数据状态的集中管理，复杂组件的数据通信问题。



```
const store = new Vuex.Store({
  state: {
  },
  actions: {
  },
  mutations: {
  },
  getters: {
  },  
  modules: {
    
  }
})
```



- state
  - state 定义了应用状态的数据结构，同样可以在这里设置默认的初始状态。
- actions
  - Actions 即是定义提交触发更改信息的描述 
- mutations:
  - 更新应用状态的地方。
- getters
  -  Getters 允许组件从 Store 中获取数据 
- modules
  -  modules 对象允许将单一的 Store 拆分为多个 Store 的同时保存在单一的状态树中。 



对于 `$` 前缀来说，其在 Vue 生态系统中的目的是暴露给用户的一个特殊的实例属性，所以把它用于*私有*属性并不合适。 

## Vue路由

### 路由模式

"hash" | "history" | "abstract" 

- `hash`: 使用 URL hash 值来作路由。支持所有浏览器，包括不支持 HTML5 History Api 的浏览器。
- `history`: 依赖 HTML5 History API 和服务器配置。查看 [HTML5 History 模式](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。
- `abstract`: 支持所有 JavaScript 运行环境，如 Node.js 服务器端。**如果发现没有浏览器的 API，路由会自动强制进入这个模式。**



#### hash

hash模式仅改变hash部分的内容，而hash部分是不会包含在http请求中的。所以hash模式下遇到根据url请求页面不会有问题



#### history

而history模式则将url修改的就和正常请求后端的url一样(history不带#)

[oursite.com/user/id](https://link.juejin.im/?target=http%3A%2F%2Foursite.com%2Fuser%2Fid)

如果这种向后端发送请求的话，后端没有配置对应/user/id的get路由处理,会返回404错误。



优势：

- pushState设置的新url可以是与当前url同源的任意url,而hash只可修改#后面的部分，故只可设置与当前同文档的url
- pushState设置的新url可以与当前url一模一样，这样也会把记录添加到栈中，而hash设置的新值必须与原来不一样才会触发记录添加到栈中
- pushState通过stateObject可以添加任意类型的数据记录中，而hash只可添加短字符串 pushState可额外设置title属性供后续使用



#### abstract

不涉及和浏览器地址的相关记录 

abstract模式是使用一个不依赖于浏览器的浏览历史虚拟管理后端，其原理是通过数组模拟浏览器历史记录栈的功能 



### 路由导航守卫

导航守卫主要用来通过跳转或取消的方式守卫导航。



**当点击切换路由时：beforeRouterLeave-->beforeEach-->beforeEnter-->beforeRouteEnter-->beforeResolve-->afterEach-->beforeCreate-->created-->beforeMount-->mounted-->beforeRouteEnter的next的回调** 



#### 钩子

每个守卫方法接收三个参数：

- **to: Route**: 即将要进入的目标 [路由对象](https://router.vuejs.org/zh/api/#%E8%B7%AF%E7%94%B1%E5%AF%B9%E8%B1%A1)
- **from: Route**: 当前导航正要离开的路由
- **next: Function**: 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。
  - **next()**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
  - **next(false)**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
  - **next('/') 或者 next({ path: '/' })**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 [`router-link` 的 `to` prop](https://router.vuejs.org/zh/api/#to) 或 [`router.push`](https://router.vuejs.org/zh/api/#router-push) 中的选项。
  - **next(error)**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 [`router.onError()`](https://router.vuejs.org/zh/api/#router-onerror) 注册过的回调。

**确保要调用 next 方法，否则钩子就不会被 resolved。**



#### 全局守卫

##### router.beforeEach 全局前置守卫

​	当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 **等待中**。 



##### router.beforeResolve 全局解析守卫

​	这和 `router.beforeEach` 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。 



##### router.afterEach  全局后置钩子

这些钩子不会接受 `next` 函数也不会改变导航本身： 



#### 组件内守卫



##### beforeRouteEnter  进入组件之前

进入该组件之前被调用，组件实例还没有被创建，不能使用 this关键字

通过传一个回调给 next来访问组件实例，在导航被确认的时候执行回调 ，用next函数的 vm 参数充当 this

    export default {
        name: "Admin",
        data(){
          return{
            infor:'hw'
          }
        },
      beforeRouteEnter:(to,from,next)=>{
          //此时该组件还没被实例化
          alert(this.infor);       //弹出消息框信息为 undefined
        next(vm =>{
          //此时该组件被实例化了
          alert(vm.infor);         //弹出消息框信息为 hw
        })
      }



注意 `beforeRouteEnter` 是支持给 `next` 传递回调的唯一守卫。对于 `beforeRouteUpdate` 和 `beforeRouteLeave` 来说，`this` 已经可用了，所以**不支持**传递回调，因为没有必要了。 



##### beforeRouteLeave  离开组件之后

 **离开组件之后调用，可以调用 this 关键字** 

这个离开守卫通常用来禁止用户在还未保存修改前突然离开。该导航可以通过 `next(false)` 来取消。 



##### **beforeRouteUpdate**     组件被复用的时候被调用

```
 beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 this
  }
```





1. router.match 可以根据请求的 path 和 method 筛选出匹配的 route



### $route 

**$route 是“路由信息对象”，包括 path，params，hash，query，fullPath，matched，name 等路由信息参数。**

**① $route.path** 字符串，对应当前路由的路径，总是解析为绝对路径，如 "/order"。

**② $route.params** 一个 key/value 对象，包含了 动态片段 和 全匹配片段， 如果没有路由参数，就是一个空对象。

**③ $route.query** 一个 key/value 对象，表示 URL 查询参数。 例如，对于路径 /foo?user=1，则有 $route.query.user为1， 如果没有查询参数，则是个空对象。

**④ $route.hash** 当前路由的 hash 值 (不带 #) ，如果没有 hash 值，则为空字符串。

**⑤ $route.fullPath** 完成解析后的 URL，包含查询参数和 hash 的完整路径。

**⑥ $route.matched** 数组，包含当前匹配的路径中所包含的所有片段所对应的配置参数对象。

**⑦ $route.name   当前路径名字**

 

##  Vue组件

### keep-alive

keep-alive是 Vue 内置的一个组件，可以使被包含的组件保留状态，或避免重新渲染。

> <keep-alive> 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 <transition> 相似，<keep-alive> 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。 当组件在 <keep-alive> 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。 在 2.2.0 及其更高版本中，activated 和 deactivated 将会在 <keep-alive> 树内的所有嵌套组件中触发。 主要用于保留组件状态或避免重新渲染。

exclude和include属性

#### keep-alive`的生命周期

- 初次进入时： 
  1. `created` > `mounted` > `activated`
  2. 退出后触发 `deactivated`
- 再次进入： 
  1. 只会触发 `activated`
- 事件挂载的方法等，只执行一次的放在 `mounted` 中；组件每次进去执行的方法放在 `activated` 中




##  Vue数据

vue2.0用的是object.defineProperty

vue3.0用的是Proxy



### Object.defineProperty 

 getter和setter,劫持用户对对象属性的取值和赋值。

```
Object.defineProperty(obj, 'name', {
  get() {
    console.log('劫持了你的取值操作啦');
    return val;
  },
  set(newVal) {
    console.log('劫持了你的赋值操作啦');
    val = newVal;
  }
});
```



1. vue.js首先通过Object.defineProperty来对要监听的数据进行getter和setter劫持，当数据的属性被赋值/取值的时候，vue.js就可以察觉到并做相应的处理。

2. 通过订阅发布模式，我们可以为data对象的每个属性都创建一个发布者，

   当有其他订阅者依赖于这个属性的时候，则将订阅者加入到发布者的队列中。

   利用Object.defineProperty的数据劫持，在属性的setter调用的时候，该属性的发布者通知所有订阅者更新内容。





### vue.set

**如果在实例创建之后添加新的属性到实例上，它不会触发视图更新**。 

```
Vue.set(data,'sex', '男')
```



### methods

methods是方法，要有一定的触发条件才能执行 

而对于method ，只要发生重新渲染，method 调用总会执行该函数。 



### computed

computed是计算属性， 在DOM加载后和依赖变化后马上执行

**computed**计算属性是基于它们的依赖进行缓存的,只有在它的相关依赖发生改变时才会重新求值。 



### watch

侦听器watch是侦听一个特定的值，当该值变化时执行特定的函数。 

watch是命令式和重复式的，假如监听涉及到多个属性，代码就会变得繁杂，不如computed直接



## vue响应式

1. 在 Vue 中，从性能/体验的性价比考虑，就弃用了这个特性。Object.defineProperty无法监控到数组下标的变化，导致通过数组下标添加元素，不能实时响应；
2. Object.defineProperty只能劫持对象的属性，从而需要对每个对象，每个属性进行遍历，如果，属性值是对象，还需要深度遍历。Proxy可以劫持整个对象，并返回一个新的对象。
3. Proxy不仅可以代理对象，还可以代理数组。还可以代理动态增加的属性。





## 组件间通讯

### props

父组件传递给子组件

```js
// 数组:不建议使用
props:[]

// 对象
props:{
 inpVal:{
  type:Number, //传入值限定类型
  // type 值可为String,Number,Boolean,Array,Object,Date,Function,Symbol
  // type 还可以是一个自定义的构造函数，并且通过 instanceof 来进行检查确认
  required: true, //是否必传
  default:200,  //默认值,对象或数组默认值必须从一个工厂函数获取如 default:()=>[]
  validator:(value) {
    // 这个值必须匹配下列字符串中的一个
    return ['success', 'warning', 'danger'].indexOf(value) !== -1
  }
 }
}

```



### $emit

子组件传递给父组件

```js
// 父组件
<home @title="title">
// 子组件
this.$emit('title',[{title:'这是title'}])
```



### attrs

获取子传父中未在 props 定义的值

```js
// 父组件
<home title="这是标题" width="80" height="80" imgUrl="imgUrl"/>

// 子组件
mounted() {
  console.log(this.$attrs) //{title: "这是标题", width: "80", height: "80", imgUrl: "imgUrl"}
},
```



### listeners

listeners" 传入内部组件——在创建更高层次的组件时非常有用

```js
// 父组件
<home @change="change"/>

// 子组件
mounted() {
  console.log(this.$listeners) //即可拿到 change 事件
}
```



###  provide和inject

描述: provide 和 inject 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中; 并且这对选项需要一起使用; 以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。

```js
//父组件:
provide: { //provide 是一个对象,提供一个属性或方法
  foo: '这是 foo',
  fooMethod:()=>{
    console.log('父组件 fooMethod 被调用')
  }
},

// 子或者孙子组件
inject: ['foo','fooMethod'], //数组或者对象,注入到子组件
mounted() {
  this.fooMethod()
  console.log(this.foo)
}
//在父组件下面所有的子组件都可以利用inject

```



### parent和children

```js
//父组件
mounted(){
  console.log(this.$children) 
  //可以拿到 一级子组件的属性和方法
  //所以就可以直接改变 data,或者调用 methods 方法
}

//子组件
mounted(){
  console.log(this.$parent) //可以拿到 parent 的属性和方法
}
```



### $refs

```j&#39;s
// 父组件
<home ref="home"/>

mounted(){
  console.log(this.$refs.home) //即可拿到子组件的实例,就可以直接操作 data 和 methods
}

```



### $root

```
// 父组件
mounted(){
  console.log(this.$root) //获取根实例,最后所有组件都是挂载到根实例上
  console.log(this.$root.$children[0]) //获取根实例的一级子组件
  console.log(this.$root.$children[0].$children[0]) //获取根实例的二级子组件
}

```



## vueX



```
state:定义存贮数据的仓库 ,可通过this.$store.state 或mapState访问
getter:获取 store 值,可认为是 store 的计算属性,可通过this.$store.getter 或
       mapGetters访问
mutation:同步改变 store 值,为什么会设计成同步,因为mutation是直接改变 store 值,
         vue 对操作进行了记录,如果是异步无法追踪改变.可通过mapMutations调用
action:异步调用函数执行mutation,进而改变 store 值,可通过 this.$dispatch或mapActions
       访问
modules:模块,如果状态过多,可以拆分成模块,最后在入口通过...解构引入
```



### 