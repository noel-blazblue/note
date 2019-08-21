## 一、动画

#### 1.过渡类名动画



1.html结构:

```
    <transition>
      <div v-show="isshow">动画</div>
    </transition>
```

​	要使用transition包裹

2.定义两组类样式：

​	定义 开始状态 和 结束状态

```
.fade-enter,
.fade-leave-to {
    opacity: 0;
    transform: translateX(100px);
}
```

​	定义进入和离开时候的过渡状态

```
 .fade-enter-active,
 .fade-leave-active {
      transition: all 0.2s ease;
      position: absolute;
 }
```



#### 2.第三方CSS动画库动画

1.导入动画类库：

```
<link rel="stylesheet" type="text/css" href="./lib/animate.css">
```

2.定义 transition 及属性：

```
<transition
	enter-active-class="fadeInRight"					//进入状态
    leave-active-class="fadeOutRight"					//离开状态
    :duration="{ enter: 500, leave: 800 }">				//持续时间
  	<div class="animated" v-show="isshow">动画哦</div>	  //要添加class=“animated”
</transition>
```

在transition标签内定义：开始状态、结束状态、持续时间    的第三方类名

#### 3.钩子函数动画

1.定义methods 钩子方法：

```
methods: {
        beforeEnter(el) { 				// 动画进入之前的回调
          el.style.transform = 'translateX(500px)';
        },
        enter(el, done) {				 // 动画进入完成时候的回调
          el.offsetWidth;
          el.style.transform = 'translateX(0px)';
          done();
        },
        afterEnter(el) { 				// 动画进入完成之后的回调
          this.isshow = !this.isshow;
        }
      }
```

2.定义动画过渡时长和样式：

```
.show{
      transition: all 0.4s ease;
}
```

3.使用transition组件，绑定钩子函数：

```

    <transition
    @before-enter="beforeEnter"
    @enter="enter"
    @after-enter="afterEnter"
    >
      <div v-if="isshow" class="show">OK</div>
    </transition>
```



#### 4.transition-group 组件

如要用v-for循环 ，需要用transition-group 组件 包裹起来

```
<transition-group tag="ul" name="list">
   	<li v-for="(item, i) in list" :key="i">{{item}}</li>
</transition-group>
```

tag="ul"  可以定义组件为什么html结构





`<transition-group>` 组件还有一个特殊之处。不仅可以进入和离开动画，**还可以改变定位**。

`v-move` 和 `v-leave-active` 结合使用，能够让列表的过渡更加平缓柔和：

```
.v-move{
  transition: all 0.8s ease;
}
.v-leave-active{
  position: absolute;
}
```



## 二、定义Vue组件

组件的出现，就是为了拆分Vue实例的代码量的，能够让我们以不同的组件，来划分不同的功能模块，将来我们需要什么样的功能，就可以去调用对应的组件即可；
组件化和模块化的不同：

- 模块化： 是从代码逻辑的角度进行划分的；方便代码分层开发，保证每个功能模块的职能单一；
- 组件化： 是从UI界面的角度进行划分的；前端的组件化，方便UI组件的重用；



#### 1.全局组件

+ 使用 Vue.extend 配合 Vue.component 方法：

```
var login = Vue.extend({
      template: '<h1>登录</h1>'
    });
Vue.component('login', login);
```

- 直接使用 Vue.component 方法：

```
Vue.component('register', {
      template: '<h1>注册</h1>'
});
```

- 将模板字符串，定义到script标签种：

```
<script id="tmpl" type="x-template">
      <div><a href="#">登录</a> | <a href="#">注册</a></div>
 </script>
```

同时，需要使用 Vue.component 来定义组件：

```
Vue.component('account', {
      template: '#tmpl'
});
```

> 注意： 组件中的DOM结构，有且只能有唯一的根元素（Root Element）来进行包裹！



#### 2.局部组件

1.定义子组件：

```
<script>
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {},
      components: { 			// 定义子组件
        account: { 				// account 组件
          template: '<div><h1>这是Account组件{{name}}</h1><login></login></div>', // 在这里使用定义的子组件
          components: { // 定义子组件的子组件
            login: { // login 组件
              template: "<h3>这是登录组件</h3>"
            }
          }
        }
      }
    });
</script>
```

2.引用组件

```
<div id="app">
    <account></account>
</div>
```

account标签为组件的名称



#### 3.切换组件



##### 3.1使用`flag`标识符结合`v-if`和`v-else`切换组件



页面结构：

```
<div id="app">
    <input type="button" value="toggle" @click="flag=!flag">
    <my-com1 v-if="flag"></my-com1>
    <my-com2 v-else="flag"></my-com2>
  </div>
```

Vue实例定义：

```
<script>
    Vue.component('myCom1', {
      template: '<h3>奔波霸</h3>'
    })

    Vue.component('myCom2', {
      template: '<h3>霸波奔</h3>'
    })

    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {
        flag: true
      },
      methods: {}
    });
  </script>
```



##### 4.使用`:is`属性来切换不同的子组件,并添加切换动画

1.定义组件：

```
  // 登录组件
    const login = Vue.extend({
      template: `<div>
        <h3>登录组件</h3>
      </div>`
    });
    Vue.component('login', login);
```

​	

    // 注册组件
    const register = Vue.extend({
      template: `<div>
        <h3>注册组件</h3>
      </div>`
    });
    Vue.component('register', register);


```
var vm = new Vue({
  el: '#app',
  data: { comName: 'login' },
  methods: {}
});
```



2.使用`component`标签，来引用组件，并通过`:is`属性来指定要加载的组件：



通过两个a标签给变量comName传值

```
<a href="#" @click.prevent="comName='login'">登录</a>
<a href="#" @click.prevent="comName='register'">注册</a>
<component :is="comName"></component>
```

