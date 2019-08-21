### HTML语义化

语义化是指根据内容的结构（内容语义化），去选择合适的标签（代码语义化），便于开发者阅读和写出更优雅的代码的同时，让浏览器的爬虫和机器更好的解析。 

- 有利于SEO，有助于爬虫抓取更多的有效信息，爬虫是依赖于标签来确定上下文和各个关键字的权重。
- 语义化的HTML在没有CSS的情况下也能呈现较好的内容结构与代码结构
- 方便其他设备的解析
- 便于团队开发和维护



## 列表





- ul	无序列表
  + li
- ol     有序列表
  - li
- dl     自定义列表
  - dt		标题
  - dd           内容



SVG 即 Scalable Vector Graphics，是一种用来绘制 [矢量图](https://www.baidu.com/s?wd=%E7%9F%A2%E9%87%8F%E5%9B%BE&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YvmHc3njPbmHwBmWn4Pyu90ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHmYnWf3nj6YPWDzPj6vnHT3n0) 的 HTML5 标签。你只需定义好XML属性，就能获得一致的图像元素。 



img标签不用成对出现 



## title和alt的区别

```
1.<img src="#" alt="alt信息" />
//1.当图片不输出信息的时候，会显示alt信息 鼠标放上去没有信息，当图片正常读取，不会出现alt信息

2.<img src="#" alt="alt信息" title="title信息" />
// 2.当图片不输出信息的时候，会显示alt信息 鼠标放上去会出现title信息

//当图片正常输出的时候，不会出现alt信息，鼠标放上去会出现title信息

```

另外还有一些关于title属性的知识：

```
title属性可以用在除了base，basefont，head，html，meta，param，script和title之外的所有标签
title属性的功能是提示。额外的说明信息和非本质的信息请使用title属性。title属性值可以比alt属性值设置的更长
title属性有一个很好的用途，即为链接添加描述性文字，特别是当连接本身并不是十分清楚的表达了链接的目的。
```

 

 

 

 