## 一、基本结构代码

1. 导入Vue的包 -

```
  <script src="./lib/vue-2.4.0.js"></script>
```



2. 创建一个Vue的实例

```
<script>

​    // 当我们导入包之后，在浏览器的内存中，就多了一个 Vue 构造函数

​    //  注意：我们 new 出来的这个 vm 对象，就是我们 MVVM中的 VM调度者

​    var vm = new Vue({

​      el: '#app', 	 		// 表示，当前我们 new 的这个 Vue 实例，要控制页面上的哪个区域

​      // 这里的 data 就是 MVVM中的 M，专门用来保存 每个页面的数据的

​      data: { 				// data 属性中，存放的是 el 中要用到的数据

​        msg: '欢迎学习Vue'	 

​      }

​    })

  </script>
```

在 VM 实例中，如果要访问 data 上的数据，或者要访问 methods 中的方法， 必须带 this 

## 二、插值表达式

```
<h4>{{ msg }}</h4> 
```

插值表达式  只会替换自己的这个占位符，不会把 整个元素的内容清空

 VM实例，会监听自己身上 data 中所有数据的改变，只要数据一发生变化，就会自动把 最新的数据，从data 上同步到页面中去；【好处：程序员只需要关心数据，不需要考虑如何重新渲染DOM页面】 



## 三、vue指令

#### 

- **v-cloak**

  ​    <!-- 使用 v-cloak 能够解决 插值表达式闪烁的问题 -->

  ```
  <p v-cloak>++++++++ {{ msg }} ----------</p>
  ```

- **v-text**

  <!-- 默认 v-text 是没有闪烁问题的 -->

  ```
  <h4 v-text="msg">==================</h4>
  ```

  <!-- v-text会覆盖元素中原本的内容  -->

- **v-html**

  ```
  <h4 v-html="msg">==================</h4>
  ```

  会直接输出html的格式

-  **v-bind**

  <!-- v-bind: 是 Vue中，提供的用于绑定属性的指令 -->

  ```
  <input type="button" value="按钮" v-bind:title="mytitle + '123'">
  
   //mytitle 为data中的变量
  ```

  <!-- 注意： v-bind: 指令可以被简写为 :要绑定的属性 -->

  <!-- v-bind 中，可以写合法的JS表达式 -->

- **v-on**

   Vue 中提供了 v-on: 事件绑定机制 

  ```
   <input type="button" value="按钮" :title="mytitle + '123'" v-on:click="alert('hello')">
  ```

  可简写为@click=""，""内为methods中的方法

- **v-model**

  ​    <!-- 使用  v-model 指令，可以实现 表单元素和 Model 中数据的双向数据绑定 -->

  ​    <!-- 注意： v-model 只能运用在 表单元素中 -->

  ```
  <input type="text" v-model="msg">
  ```

- **v-for**

  - 循环普通数组

    ```
    <p v-for="(item, i) in list">索引值：{{i}} --- 每一项：{{item}}</p>
    ```

  - 循环对象数组

    ```
    <p v-for="(user, i) in list">Id：{{ user.id }} --- 名字：{{ user.name }} --- 索引：{{i}}</p>
    ```

  - 循环对象

    ```
    <p v-for="(val, key, i) in user">值是： {{ val }} --- 键是： {{key}} -- 索引： {{i}}</p>
    ```

    <!-- 注意：在遍历对象身上的键值对的时候， 除了 有  val  key  ,在第三个位置还有 一个 索引  --> 

    

    <!-- 注意： key 在使用的时候，必须使用 v-bind 属性绑定的形式，指定 key 的值 -->

    <!-- 在组件中，使用v-for循环的时候，或者在一些特殊情况中，如果 v-for 有问题，必须 在使用 v-for 的同时，指定 唯一的 字符串/数字 类型 :key 值 -->

    ```
    <p v-for="item in list" :key="item.id">
    
       <input type="checkbox">{{item.id}} --- {{item.name}}
    
    </p>
    ```

    

- **v-if**

  <!-- v-if 的特点：每次都会重新删除或创建元素 --> 

  ```
  <h3 v-if="flag">这是用v-if控制的元素</h3>
  ```

- **v-show**

  <!-- v-show 有较高的初始渲染消耗 --> 

  //会设置元素属性为display：none

  ```
  
  ```

## 四、事件修饰符

#### 1	.stop 

​	<!-- 使用  .stop  阻止冒泡 --> 

```
<input type="button" value="戳他" @click.stop="btnHandler">
```

#### 2	.capture 

​	 <!-- 使用  .capture 实现捕获触发事件的机制 --> 

```
<div class="inner" @click.capture="div1Handler">
      <input type="button" value="戳他" @click="btnHandler">
</div> -->
```

#### 3	.prevent 

​	<!-- 使用 .prevent 阻止默认行为 --> 

```
<a href="http://www.baidu.com" @click.prevent="linkClick">有问题，先去百度</a>
```

#### 4	self 

​	<!-- 使用 .self 实现只有点击当前元素时候，才会触发事件处理函数 --> 

```
<div class="inner" @click="div1Handler">
      <input type="button" value="戳他" @click="btnHandler">
</div> 
```

<!-- .self 只会阻止自己身上冒泡行为的触发，并不会真正阻止 冒泡的行为 --> 

#### 5	.once 

​	<!-- 使用 .once 只触发一次事件处理函数 --> 

```

```

## 五、绑定class

 <!-- 第一种使用方式，直接传递一个数组，注意： 这里的 class 需要使用  v-bind 做数据绑定 -->

​    <!-- <h1 :class="['thin', 'italic']">这是一个很大很大的H1，大到你无法想象！！！</h1> -->

 

​    <!-- 在数组中使用三元表达式 -->

​    <!-- <h1 :class="['thin', 'italic', flag?'active':'']">这是一个很大很大的H1，大到你无法想象！！！</h1> -->

 

​    <!-- 在数组中使用 对象来代替三元表达式，提高代码的可读性 -->

​    <!-- <h1 :class="['thin', 'italic', {'active':flag} ]">这是一个很大很大的H1，大到你无法想象！！！</h1> -->

 

​    <!-- 在为 class 使用 v-bind 绑定 对象的时候，对象的属性是类名，由于 对象的属性可带引号，也可不带引号，所以 这里我没写引号；  属性的值 是一个标识符 -->