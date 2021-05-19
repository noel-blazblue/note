

## 一、创建过程：



1. 创建一个 XMLHttpRequest 类型的对象 —— 相当于打开了一个浏览器

```
 var xhr = new XMLHttpRequest()
```

2. 打开与一个网址之间的连接 —— 相当于在地址栏输入访问地址 

```
xhr.open('GET', './time.php')
```

3. 通过连接发送一次请求 —— 相当于回车或者点击访问发送请求 

```
xhr.send(null)
```

4. 指定 xhr 状态变化事件处理函数 —— 相当于处理网页呈现后的操作

```
 xhr.onreadystatechange = function () {
```

   通过 xhr 的 readyState 判断此次请求的响应是否接收完成 

```
if (this.readyState === 4) { 
```

   通过 xhr 的 responseText 获取到响应的响应体 console.log(this)

 

## 二、readyState    

![1551949051303](C:\Users\ADMINI~1\AppData\Local\Temp\1551949051303.png)



## 三、概念

### AJAX是什么

Ajax是多种技术组合起来的一种浏览器和服务器交互技术，基本思想是允许一个互联网浏览器向一个远程页面/服务做异步的http调用，并且用收到的数据来更新一个当前web页面而不必刷新整个页面。该技术能够改进客户端的体验。 



### AJAX的交互模型

- 同步：脚本会停留并等待服务器发送回复然后再继续 
- 异步：脚本允许页面继续其进程并处理可能的回复 



