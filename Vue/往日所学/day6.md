## 一、安装第三方库



## 二、路由实现页面跳转

在根组件构建以下结构



### 顶部：

设置固定定位

```
    <mt-header fixed title="黑马程序员·Vue项目"></mt-header>
```



### 中间：

放置router-view视图，显示组件

```
			<router-view></router-view>
```



### 底部：

放置路由导航

```
  <nav class="mui-bar mui-bar-tab">
			<router-link class="mui-tab-item" to="/home">
				<span class="mui-icon mui-icon-home"></span>
				<span class="mui-tab-label">首页</span>
			</router-link>
			<router-link class="mui-tab-item" to="/member">
				<span class="mui-icon mui-icon-contact"></span>
				<span class="mui-tab-label">会员</span>
			</router-link>
			<router-link class="mui-tab-item" to="/shopcar">
				<span class="mui-icon mui-icon-extra mui-icon-extra-cart">
					<span class="mui-badge">0</span>
				</span>
				<span class="mui-tab-label">购物车</span>
			</router-link>
			<router-link class="mui-tab-item" to="/search">
				<span class="mui-icon mui-icon-search"></span>
				<span class="mui-tab-label">搜索</span>
			</router-link>
		</nav>
```

用router-link 代替a标签去进行导航，实现中间区域跳转页面，上下导航不变



### 配置路由

```
var router = new VueRouter({
    routes: [
        { path: '/', redirect: '/home' },
        { path: '/home', component: HomeContainer },
        { path: '/member', component: MemberContainer },
        { path: '/shopcar', component: ShopcarContainer },
        { path: '/search', component: SearchContainer }
    ],
    linkActiveClass: 'mui-active'
})
```



- { path: '/', redirect: '/home' },

  可重定向到home页面

- linkActiveClass: 'mui-active' 

  在路由跳转时切换样式



## 三、页面滑动动画

#### 用transition标签包裹router-view

```
  <transition>
        <router-view></router-view>
    </transition>
```



#### 设置样式

- 分别为进场和出场动画设置不同的样式

  ```
  .v-enter {
    opacity: 0;
    transform: translateX(100%);
  }
  
  .v-leave-to {
    opacity: 0;
    transform: translateX(-100%);
    position: absolute;
  }
  ```

  离场动画要绝对定位，脱离文档流



- 设置过渡

  ```
  .v-enter-active,
  .v-leave-active {
    transition: all 0.5s ease;
  }
  ```

  

- 隐藏横向溢出

  ```
  .app-container {
    overflow-x: hidden;
  }
  ```

  

## 四、git忽略文件



文件命名：.gitignore

添加内容

```
node_modules
.idea
.vscode
.git
```



上传到仓库时就会忽略这些文件夹或文件