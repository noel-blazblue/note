## c3新增样式

1、box-shadow（阴影效果）

2、border-color（为边框设置多种颜色）

3、border-image（图片边框）

4、text-shadow（文本阴影）

5、text-overflow（文本截断）

6、word-wrap（自动换行）

7、border-radius（圆角边框）

8、opacity（透明度）

9、box-sizing（控制盒模型的组成模式）

10、resize（元素缩放）

11、outline（外边框）

12、background-size（指定背景图片尺寸）

13、background-origin（指定背景图片从哪里开始显示）

14、background-clip（指定背景图片从什么位置开始裁剪）

15、background（为一个元素指定多个背景）

16、hsl（通过色调、饱和度、亮度来指定颜色颜色值）

17、hsla（在hsl的基础上增加透明度设置）

18、rgba（基于rgb设置颜色，a设置透明度）



- transition：过渡

- transform：旋转、缩放、移动或者倾斜

- animation：动画

- gradient：渐变

- shadow：阴影

- border-radius：圆角

  

body,form,img 有默认的padding值



清除浮动的方式：

- 在浮动元素末尾加一个空的标签如

  ```
  <div style="clear:both"></div>
  ```

- 设置父元素为`overflow:hidden`

- 给父元素添加`clearfix`类



## 优先级

!important -> 行内样式 -> #id -> .class -> 元素和伪元素 -> * -> 继承 -> 默认 



## BFC

BFC 就是 块级格式上下文，它是一个独立的渲染区域，让处于 BFC 内部的元素和外部的元素相互隔离，使内外元素的定位不会相互影响。

一定的 CSS 声明可以生成 BFC，浏览器对生成的 BFC 有一系列的渲染规则，利用这些渲染规则可以达到一定的布局效果。

- 为什么需要 BFC 呢？

1. 它可以防止 margin 元素重叠（div 中包含 ul，而 div 与 ul 之间的垂直距离，取决于 div、ul、li 三者之间的最大外边距，这时候给 ul 一个 display:inline-block 即可解决这个问题）
2. 清除内部浮动（div 中包含 ul，而 ul 采用 float:left，那么 div 将变成一长条，这时候给 div 加上规则使其变成 BFC 即可）

- 如何产生 BFC？

1. display: inline-block
2. position: absolute/fixed

- 工作中一般可能不会顾及这个：

1. float 很少使用了，尽可能使用 flex
2. css reset 一般会清除掉一些问题，减少 BFC 的使用。



## 什么是外边距合并？

当两个垂直外边距相遇时，它们将形成一个外边距，合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。



## 清除浮动

#### :after+clear:both



#### 父元素设置over-flow: hidden





## 布局

### 垂直居中

#### 定位

margin： -

```
#child {
    width: 150px;
    height: 100px;
    background: orange;
    position: absolute;
    top: 50%;
    margin: -50px 0 0 0; 
}
```



transform

```
#box {
    width: 300px;
    height: 300px;
    background: #ddd;
    position: relative;
}
#child {
    background: orange;
    position: absolute;
    top: 50%;
    transform: translate(0, -50%);
}
```



margin： %

```
#box {
    width: 300px;
    height: 300px;
    background: #ddd;
    position: relative;
}
#child {
    width: 50%;
    height: 30%;
    background: orange;
    position: absolute;
    top: 50%;
    margin: -15% 0 0 0;
}
```



margin：auto

```
#child {
    width: 200px;
    height: 100px;
    background: orange;
    position: absolute;
    top: 0;
    bottom: 0;
    margin: auto;
    line-height: 100px;
}
```



