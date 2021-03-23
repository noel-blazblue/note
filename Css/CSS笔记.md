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



##  CSS3中transition和animation的属性分别有哪些

`transition` 过渡动画：
 (1) `transition-property`：属性名称
 (2) `transition-duration`: 间隔时间
 (3) `transition-timing-function`: 动画曲线
 (4) `transition-delay`: 延迟

`animation` 关键帧动画：
 (1) `animation-name`：动画名称
 (2) `animation-duration`: 间隔时间
 (3) `animation-timing-function`: 动画曲线
 (4) `animation-delay`: 延迟
 (5) `animation-iteration-count`：动画次数
 (6) `animation-direction`: 方向
 (7) `animation-fill-mode`: 禁止模式



## 优先级

!important -> 行内样式 -> #id -> .class -> 元素和伪元素 -> * -> 继承 -> 默认 

- `!important`
- 内联样式（1000）
- ID选择器（0100）
- 类选择器/属性选择器/伪类选择器（0010）
- 元素选择器/关系选择器/伪元素选择器（0001）
- 通配符选择器（0000）



## BFC

> BFC 就是 块级格式上下文，它是一个独立的渲染区域，让处于 BFC 内部的元素和外部的元素相互隔离，使内外元素的定位不会相互影响。
>

CSS 声明可以生成 BFC，浏览器对生成的 BFC 有一系列的渲染规则，利用这些渲染规则可以达到一定的布局效果。



- **BFC可以解决的问题**
  - 垂直外边距重叠问题
  - 去除浮动
  - 自适用两列布局（`float` + `overflow`）



- #### 触发BFC的条件

  - 根元素或其它包含它的元素

  - 浮动元素 (元素的 `float` 不是 `none`)

  - 绝对定位元素 (元素具有 `position` 为 `absolute` 或 `fixed`)

  - 内联块 (元素具有 `display: inline-block`)

  - 表格单元格 (元素具有 `display: table-cell`，HTML表格单元格默认属性)

  - 表格标题 (元素具有 `display: table-caption`, HTML表格标题默认属性)

  - 具有`overflow` 且值不是 `visible` 的块元素

  - 弹性盒（`flex`或`inline-flex`）

  - `display: flow-root`

  - `column-span: all`

    

- **工作中一般可能不会顾及这个：**

  - float 很少使用了，尽可能使用 flex
  - css reset 一般会清除掉一些问题，减少 BFC 的使用。



## 什么是外边距合并？

当两个垂直外边距相遇时，它们将形成一个外边距，合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。



## 清除浮动

#### :after+clear:both



#### 父元素设置over-flow: hidden





## 布局

### 盒子模型
由从内到外的四个部分构成，即内容区（content）、内边距(padding)、边框(border)和外边距(margin)。内容区是盒子模型的中心，呈现盒子的主要信息内容；内边距是内容区和边框之间的空间；边框是环绕内容区和内边距的边界；外边距位于盒子的最外围，是添加在边框外周围的空间。

![](https://user-gold-cdn.xitu.io/2019/11/22/16e930e04e31efa6?imageslim)

根据计算宽高的区域我们可以将其分为`IE盒模型`和`W3C标准盒模型`，可以通过`box-sizing`来进行设置：

-   `content-box`：W3C标准盒模型
-   `border-box`：IE盒模型

区别：  
`W3C标准盒模型`：width(宽度) = content(内容宽度)  
`IE盒模型`：width(宽度) = content(内容宽度) + padding(内边距) + border(边框)

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



## 伪类和伪元素的区别

- **伪类：** 用于当已有元素处于的某个状态时，为其添加对应的样式，这个状态是根据用户行为而动态变化的。

- **伪元素：** 用于创建一些不在文档树中的元素，并为其添加样式。

- **区别：**伪类的操作对象是文档树中已有的元素，而伪元素则创建了一个文档树外的元素。因此，伪类与伪元素的区别在于：**有没有创建一个文档树之外的元素。**



## 层叠上下文

![img](https://user-gold-cdn.xitu.io/2019/8/30/16ce245b90085292?imageslim)

position 和 transform 会自动创建层叠上下文，为z-index: auto