## 一、配置

基于node.js

#### 下载：

```
npm i --save axios
npm i --save vue-axios
```



#### 引入：

```
import axios from 'axios'
import VueAxios from 'vue-axios'
```



#### 注册：

```
Vue.use(VueAxios,axios)
```



## 二、跨域

在根目录新建vue.config.js文件

#### 配置：

```
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'url',
        ws: true,
        changeOrigin: true,
        pathRewrite: {
          '^/api': '/', // rewrite path
        }
      }
    }
  }
}
```

`url`为要跨域的地址

`pathRewrite` 自动把所有以api开头的地址转换为target中的url



#### 设置根url

在main.js文件中配置

```
axios.defaults.baseURL = '/api';
```

为所有请求添加上/api，不用手动添加



 Axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';

设置所有post的请求头



## 三、API

### 拦截器

在请求或响应被 `then` 或 `catch` 处理前拦截它们。

```
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```

如果你想在稍后移除拦截器，可以这样：

```
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

