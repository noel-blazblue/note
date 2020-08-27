### Taro 当前架构

> 由于jsx需要经过多端适配，所以可能会有部分兼容不到的问题，需通过规范react写法避免。

Taro 当前的架构主要分为：**编译时** 和 **运行时**。

其中编译时主要是将 Taro 代码通过 **Babel**[11] 转换成 小程序的代码，如：`JS`、`WXML`、`WXSS`、`JSON`。

运行时主要是进行一些：生命周期、事件、data 等部分的处理和对接。

https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVVWoQKnZN53dyPkAkHC2gSreyxqaROsxcDtFicficXicpA7PTCtIOldfzzicjjaM6cT8c7tmfUvMdrIyw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1

https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVVWoQKnZN53dyPkAkHC2gSrETibsrCkMhdfAFQL5zxod8a5ic6rCFd3HxT6At907RYQcVicYdGrA7ZsA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1

**Taro 当前架构只是在开发时遵循了 React 的语法，在代码编译之后实际运行时，和 React 并没有关系**。



### RN转换到taro的办法

#### api自动转换

无法精确转换，侵入性强，转换出来的一般不能使用

```
$ taro convert
```



#### 手动转换

eslint-config-taro 的eslint插件帮助检查各种不符合Taro要求的代码风格

Animation, 原生平台组件和第三方组件Taro是不支持的，需要寻找方法规避转化问题



##### 路由

路由系统要修改为Taro自带的路由系统 和 API

**跳转方法**

```
import Taro from '@tarojs/taro'
Taro.navigateTo(params).then(...)
```

**路由配置**

```
// app.config.js
export default {
  pages: [
    'pages/index/index',
    'pages/logs/logs'
  ],
  window: {
    navigationBarBackgroundColor: '#ffffff',
    navigationBarTextStyle: 'black',
    navigationBarTitleText: '微信接口功能演示',
    backgroundColor: '#eeeeee',
    backgroundTextStyle: 'light'
  }
}
```



##### 设计稿及单位尺寸

在 Taro 中尺寸单位建议使用 `px`、 `百分比 %`，Taro 默认会对所有单位进行转换。在 Taro 中书写尺寸按照 1:1 的关系来进行书写，即从设计稿上量的长度 `100px`，那么尺寸书写就是 `100px`，当转成微信小程序的时候，尺寸将默认转换为 `100rpx`，当转成 H5 时将默认转换为以 `rem` 为单位的值。

- 设计稿尺寸匹配问题，Taro默认是根据750的设计稿匹配的，可以在配置文件的designWidth属性中进行修改
- 如果是行内长度样式，那么要做手动转换：Taro.pxTransform(10)



##### 样式

在编译时，Taro 会帮你对样式做尺寸转换操作，但是如果是在 JS 中书写了行内样式，那么编译时就无法做替换了

- RN是通过向style中导入对象的方式引入样式，而Taro是通过className结合import样式文件的方式引入样式
- RN的属性命名方法是驼峰，而Taro是短横线
- 部分RN样式属性值Taro是没有的,而且部分样式属性的默认值RN和Taro不一致，例如marginVertical，paddingVertical等等，RN有，但是Taro没有

Taro的样式编码方式(支持sass)

```
// index.js
import "index.css"
class App extends React.Component {
  render () {
    return ()
  }
}
// index.css
.bar {
  height: 10Px;
  background-color:'10px'
}
```



##### 网络请求

fetch/Ajax 等原生的要改成Taro的Taro.request这一API

```
import Taro from '@tarojs/taro'
 
Taro.request({
 url: 'http://localhost:8080/test',
 data: {  foo: 'foo' },
 header: { 'content-type': 'application/json' }
}).then(
 res => console.log(res.data)
)
```

aync/await的使用要通过导入taro的包来开启

```
import '@tarojs/async-await' 
```



##### **引用图片、音频、字体等文件的方式** 

- RN用的是<Image source={...} />和<ImageBackground source />
- Taro用的是<Image src={...} />

taro

```
// 引用文件
import namedPng from '../../images/path/named.png'
// 使用
<View>
 <Image src={namedPng} />
</View>
```

