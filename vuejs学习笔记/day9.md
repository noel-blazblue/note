## 一、编程式导航

### 1.创建组件



### 2.在路由文件中引入组件

```
import GoodsComment from './components/goods/GoodsComment.vue'
```



### 3.定义路由规则

```
 { path: '/home/goodsinfo/:id', component: GoodsInfo, name: 'goodsinfo' }
```

- :id  可接收url中传递的参数
- name  可定义路由名称



### 4.添加方法实现导航

```
this.$router.push({ name: "goodsinfo", params: { id } });
```

- name  对应路由规则中的name
- params  为传递的参数，是一个对象



### 5.接收参数

```
this.$route.params.id
```



注： `$router`为导航系列的功能，`$route`为参数系列的功能



## 二、轮播图





### 实现脱离轮播图组件

父组件获取数据，传给子组件，子组件渲染数据



1.获取图片数据

​	保存到data中

```
    getLunbotu() {
      this.$http.get("api/getlunbo").then(result => {
        if (result.body.status === 0) {
          this.lunbotuList = result.body.message;
        }
      });
    }
```

2.将数据传送给轮播图子组件

```
<swiper :lunbotuList="lunbotuList"></swiper>
```

3.轮播图子组件接收数据

```
 props: ["lunbotuList", "isfull"]
```

4.根据数据渲染页面

```
    <mt-swipe :auto="4000">
      <mt-swipe-item v-for="item in lunbotuList" :key="item.url">
        <img :src="item.img" alt="" :class="{'full': isfull}">
      </mt-swipe-item>
    </mt-swipe>
```



### 轮播图宽度问题

需求：要对不同的组件设置不同的`img`宽度

办法：给轮播图组件添加一个boolean参数，真则添加样式，假则删除样式



1.父组件传递一个参数

​	:isfull

```
<swiper :lunbotuList="lunbotuList" :isfull="true"></swiper>
```

2.子组件接收

```
props: ["lunbotuList", "isfull"]
```

3.根据参数的boolean值判断是否添加样式

```
:class="{'full': isfull}"
```



## 三、加入购物车的小球动画



### 1.设置样式用绝对定位把小球定位到起点

```
  .ball {
    width: 15px;
    height: 15px;
    border-radius: 50%;
    background-color: red;
    position: absolute;
    z-index: 99;
    top: 390px;
    left: 146px;
  }
```



### 2.定义一个flag值决定小球显示或隐藏

```
ballFlag: false, // 控制小球的隐藏和显示的标识符
```

```
<div class="ball" v-show="ballFlag" ref="ball"></div>
```



### 3.定义钩子动画

```
    <transition
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter">
      <div class="ball" v-show="ballFlag" ref="ball"></div>
    </transition>
```



### 4.beforeEnter 

```
      beforeEnter(el) {
        el.style.transform = "translate(0, 0)";
      },
```



### 5.enter 

- 获取小球和购物车的横纵坐标

  ```
  const ballPosition = this.$refs.ball.getBoundingClientRect();
  const badgePosition = document.getElementById("badge").getBoundingClientRect();
  ```

  getBoundingClientRect（），是js原生函数，能够获取left,top,bottom,right

  

- 计算他们的差值，得出移动距离

  ```
        const xDist = badgePosition.left - ballPosition.left;
        const yDist = badgePosition.top - ballPosition.top;
  ```

  

- 定义到tansform中

  ```
       el.style.transform = `translate(${xDist}px, ${yDist}px)`;
        el.style.transition = "all 0.5s cubic-bezier(.4,-0.3,1,.68)";
  ```

  

### 6.after

```
    afterEnter(el) {
      this.ballFlag = !this.ballFlag;
    },
```



## 四、监听值的改变

### 传值

1.父组件传递方法，改变值

```
@getcount="getSelectedCount"
```

```
this.selectedCount = count;
```



2.子组件用change监听值的变化

```
@change="countChanged"
```

传递给父组件

```
    countChanged() {
      this.$emit("getcount", parseInt(this.$refs.numbox.value));
    }
```



### 最大值

1.父组件动态渲染值，传递给自组件



2.自组件用watch监听值的变化

```
props: ["max"],
```

```
  watch: {
    max: function(newVal, oldVal) {
      mui(".mui-numbox")
        .numbox()
        .setOption("max", newVal);
    }
  }
```



## 五、vuex

### Store的创建



1.导入vuex

```
// 入口文件
import Vue from 'vue'
// 配置vuex的步骤
// 1. 运行 cnpm i vuex -S 
// 2. 导入包
import Vuex from 'vuex'
// 3. 注册vuex到vue中
Vue.use(Vuex)
```



2.创建实例

```
var store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    },
    subtract(state, obj) {
      console.log(obj)
      state.count -= (obj.c + obj.d)
    }
  },
  getters: {
    optCount: function (state) {
      return '当前最新的count值是：' + state.count
    }
  }
```

- state  用作存放数据
- mutations  操作state中数据的方法
- getters  包装数据



3.注册

```
const vm = new Vue({
  el: '#app',
  render: c => c(App),
  store // 5. 将 vuex 创建的 store 挂载到 VM 实例上， 只要挂载到了 vm 上，任何组件都能使用 store 来存取数据
})
```



### Store的使用

- 差值表达式

```
{{ $store.state.count }}
{{ $store.getters.optCount }}
```

- 调用mutation

```
this.$store.commit("increment");
this.$store.commit("subtract", { c: 3, d: 1 });
```

​		只能设置两个参数，第二个传值的参数可设置为一个对象

