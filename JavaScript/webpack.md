四大配置项：

entry / output / module / plugins  



运行webpack-dev-server需要在本地安装webpack



完整版：

```js
var HTMLTemp = require('html-webpack-plugin')

module.exports = {
    devtool: 'eval-source-map',
    entry: __dirname + '/src/app.js',
    output: {
        path: __dirname + '/dist',
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {test: /\.js$/, use: 'babel-loader'},
            {test: /\.css$/, use: ['style-loader', 'css-loader']},
            {test: /\.(png|jpg|git|jpeg|svg)$/},
            {test: /\.less$/,use: ['style-loader', 'css-loader', 'less-loader']}
        ]
    },
    plugins: [
        new HTMLTemp({
            template: __dirname + '/src/index.temp.html'
        })
    ]
}
```





# webpack3.0

## 一、安装

全局安装：

```
npm i -g webpack				//安装最新版本
||
npm i -g webpack@3.x			//安装指定版本
```

`i == install`	安装

`-g == global`	全局



本地安装：

```
npm i -D webpack
```

`-D == --save-dev`



项目初始化：

```
npm init -y
```

`-y` 为默认yes的意思

初始化完毕会创建一个package.json的文件



创建文件夹和文件：

+ src					//源码文件
  - index.js			//主要入口文件
+ dist                                //编译之后的文件
+ webpack.config.js      //配置文件



## 二、配置入口和出口

在webpack.config.js文件中配置：

入口：

```
module.exports = {
    entry: __dirname + '/src/index.js',
```

出口：

```
    output: {
        path: __dirname + '/dist',
        filename: 'bundle.js'
    },
```

`__dirname` 为当前文件夹路径



在本地cmd输入webpack，即会自动生成一个打包文件bundle.js



## 三、模块

### 1.webpack-dev-server

webpack-dev-server 服务器既需要在全局安装，也需要在本地安装

```
 npm install -g webpack-dev-server@2.x
 npm install -D webpack-dev-server@2.x
```



在 package.json 文件中修改：

```
"scripts": {
    "dev": "webpack-dev-server --open --content-base dist --inline --hot"
}
```

即添加了webpack-dev-server的快捷启动方式

+ --open	自动打开网页
+ --content-base dist    指定服务器运行根目录为dist
+ --inline --hot    在线更新
+ --port       修改端口



在本地运行cmd输入

```
npm run dev			
```

即可开启服务器



### 2.loader

 loader是webpack可以通过配置脚本，或者外部依赖来执行一些功能



配置loaders：

+ test：一个匹配loader要做操作的文件的一个正则表达式（必须）
+ loader(use)：loader要执行的任务的名字（必须）
+ options:为loader提供一些外部选项配置(可选项)
+ include/exclude: 手动添加必须处理的文件/文件夹,或屏蔽不需要处理的文件/文件夹（可选）



#### es6 -> es5解析

安装本地依赖：

```
npm i -D babel-core babel-loader babel-preset-es2015
```

配置：

```
{test: /\.js$/, use: 'babel-loader'}
```



#### CSS处理

安装本地依赖：

```
 npm i -D css-loader style-loader
```

配置：

```
{test: /\.css$/, use: ['style-loader', 'css-loader']},
```



#### 图片配置

安装本地依赖：

```
npm i -D file-loader url-loader
```

配置：

```
 {test: /\.(png|jpg|git|jpeg|svg)$/},
```



#### LESS和SASS

安装本地依赖：

```
npm i -D less less-loader
```

配置 ：

```
{test: /\.less$/,use: ['style-loader', 'css-loader', 'less-loader']}
```



## 四、插件

在plugins之下



### 1.HTML模板

安装：

```
npm i -D html-webpack-plugin
```

配置：

```
var HTMLTemp = require('html-webpack-plugin')
 plugins: [
        new HTMLTemp({
            template: __dirname + '/src/index.temp.html'，
            filename: 'index.html'
        })
    ]
```

+ 创建一个 在内存中 生成 HTML  页面的插件 
+ 指定 模板页面，将来会根据指定的页面路径，去生成内存中的 页面 
+ 指定生成的页面的名称 
+ 自动生成dist和bundle.js打包文件



### 2.内置插件省略后缀名

```
resolve:{
       extensions:['.js','.jsx']
    }
```

与module同级



## 五、生产环境的搭建

  安装

```
   npm install -D cross-env
    npm install -D babel-plugin-react-transform
    npm install -D react-transform-hmr
    npm install -D babel-preset-stage-2
```



在根目录创建webpack.production.config.js 文件，加上基本的配置代码

```
var pkg = require('./package.json');
var webpack = require("webpack");
var borwerOpen = require("open-browser-webpack-plugin");
var HTMLTemp = require("html-webpack-plugin")
var path = require("path")

module.exports = {
  entry:{
      app:path.resolve(__dirname, 'src/app.js'),
      vendor: Object.keys(pkg.dependencies)
  },
  output:{
    path:__dirname + "/dist",
    filename:"bundle.js"
  },
  resolve:{
       extensions:['.js','.jsx']
  },
  module:{
    rules:[
      {test:/\.json$/,use:"json-loader"},
      {test:/\.(js|jsx)$/,use:"babel-loader"},
      {test:/\.css$/,use:["style-loader","css-loader"]},
      {test:/\.(png|jpg|gif|jpeg|svg)$/,use:"url-loader?limit=2048"},
      {test:/\.less$/,use:["style-loader","css-loader","less-loader"]}
    ]
  },
  plugins:[
    new webpack.BannerPlugin("@Copyright iwen 2018/7/31"),
    new borwerOpen({
      url:"http://localhost:8080"
    }),
    new webpack.DefinePlugin({
      'process.env':{
        'NODE_ENV': JSON.stringify(process.env.NODE_ENV)
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
        compress: {
          warnings: false
        }
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      filename: '/js/[name].[chunkhash:8].js'
    }),
    new HTMLTemp({
      template:__dirname + "/src/index.temp.html"
    }),
    new webpack.DefinePlugin({
      __DEV__: JSON.stringify(JSON.parse((process.env.NODE_ENV == 'dev') || 'false'))
    })
  ]
}


```



在package.json  文件  scripts  选项 添加build和dev

```
 "build": "cross-env NODE_ENV=production webpack --config ./webpack.production.config.js --progress",
    "dev": "cross-env NODE_ENV=dev webpack-dev-server --content-base dist --progress --inline"
```



在终端中执行 npm run build 命令 



# webpack4.x

## 1.生产模式



新增`--mode development`  生产模式	`--mode production`产品模式

```
    "build": "webpack --mode production",
    "dev": "webpack-dev-server --mode development --open --port 3000 --hot"
```



## 2.解析es6语法

+ babel-loader: 负责es6语法转化 
+ babel-preset-env: 包含es6、7等版本的语法转化规则 
+ babel-polyfill: es6内置方法和函数转化垫片 
+ babel-plugin-transform-runtime: 避免polyfill污染全局变量 



# vue-cli3.0创建项目



全局安装

```
npm install -g @vue/cli
```



创建项目

```
vue create hello-cli3 
```

​	hello-cli3为项目名字



​	