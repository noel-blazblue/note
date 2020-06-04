# grid

它将网页划分成一个个网格，可以任意组合不同的网格，做出各种各样的布局。



Flex 布局是轴线布局，只能指定"项目"针对轴线的位置，可以看作是**一维布局**。Grid 布局则是将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是**二维布局**。Grid 布局远比 Flex 布局强大。



## 容器属性

### display: grid

`display: grid`指定一个容器采用网格布局。



> 注意，设为网格布局以后，容器子元素（项目）的`float`、`display: inline-block`、`display: table-cell`、`vertical-align`和`column-*`等设置都将失效。



### grid-template-columns 

###  grid-template-rows 

`grid-template-columns`属性定义每一列的列宽，`grid-template-rows`属性定义每一行的行高。

```css
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
}
```



**repeat()**

重复写同样的值非常麻烦，尤其网格很多时。这时，可以使用`repeat()`函数，简化重复的值。上面的代码用`repeat()`改写如下。

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 33.33%);
  grid-template-rows: repeat(3, 33.33%);
}
```



**auto-fill **关键字****

单元格的大小是固定的，但是容器的大小不确定。如果希望每一行（或每一列）容纳尽可能多的单元格，这时可以使用`auto-fill`关键字表示自动填充。

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fill, 100px);
}
```



**fr 关键字**

如果两列的宽度分别为`1fr`和`2fr`，就表示后者是前者的两倍。

```css
.container {
  display: grid;
  grid-template-columns: 1fr 1fr;
}
```



**minmax()**

函数产生一个长度范围，表示长度就在这个范围之中。它接受两个参数，分别为最小值和最大值。

```css
grid-template-columns: 1fr 1fr minmax(100px, 1fr);
```



**auto 关键字**

`auto`关键字表示由浏览器自己决定长度。

```css
grid-template-columns: 100px auto 100px;
```

上面代码中，第二列的宽度，基本上等于该列单元格的最大宽度，除非单元格内容设置了`min-width`，且这个值大于最大宽度。



**网格线的名称**

`grid-template-columns`属性和`grid-template-rows`属性里面，还可以使用方括号，指定每一根网格线的名字，方便以后的引用。

```css
.container {
  display: grid;
  grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4];
  grid-template-rows: [r1] 100px [r2] 100px [r3] auto [r4];
}
```



**网格线的名称**

`grid-template-columns`属性和`grid-template-rows`属性里面，还可以使用方括号，指定每一根网格线的名字，方便以后的引用。

```css
.container {
  display: grid;
  grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4];
  grid-template-rows: [r1] 100px [r2] 100px [r3] auto [r4];
}
```



### row-gap 

### column-gap 

### gap 

`grid-row-gap`属性设置行与行的间隔（行间距），`grid-column-gap`属性设置列与列的间隔（列间距）。

```css
.container {
  grid-row-gap: 20px;
  grid-column-gap: 20px;
}
```



### grid-template-areas 

网格布局允许指定"区域"（area），一个区域由单个或多个单元格组成。grid-template-areas属性用于定义区域。

```css
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
  grid-template-areas: 'a b c'
                       'd e f'
                       'g h i';
}
```



多个单元格合并成一个区域的写法如下。

```css
grid-template-areas: 'a a a'
                     'b b b'
                     'c c c';
```



### grid-auto-flow

划分网格以后，容器的子元素会按照顺序，自动放置在每一个网格。

这个顺序由`grid-auto-flow`属性决定，默认值是`row`，即"先行后列"。也可以将它设成`column`，变成"先列后行"。

```css
grid-auto-flow: column;
```



`grid-auto-flow`属性除了设置成`row`和`column`，还可以设成`row dense`和`column dense`。这两个值主要用于，某些项目指定位置以后，剩下的项目怎么自动放置。

现在修改设置，设为`row dense`，表示"先行后列"，并且尽可能紧密填满，尽量不出现空格。



### justify-items 

### align-items 

### place-items 

`justify-items`属性设置单元格内容的水平位置（左中右），`align-items`属性设置单元格内容的垂直位置（上中下）。

```css
.container {
  justify-items: start | end | center | stretch;
  align-items: start | end | center | stretch;
}
```

`place-items`属性是`align-items`属性和`justify-items`属性的合并简写形式。

```css
place-items: <align-items> <justify-items>;
```

如果省略第二个值，则浏览器认为与第一个值相等。



### justify-content

### align-content 

### place-content 

`justify-content`属性是整个内容区域在容器里面的水平位置（左中右），`align-content`属性是整个内容区域的垂直位置（上中下）。

```css
.container {
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;  
}
```



`place-content`属性是`align-content`属性和`justify-content`属性的合并简写形式。

```css
place-content: <align-content> <justify-content>
```



### grid-auto-columns 

### grid-auto-rows 

`grid-auto-columns`属性和`grid-auto-rows`属性用来设置，浏览器自动创建的多余网格的列宽和行高。它们的写法与`grid-template-columns`和`grid-template-rows`完全相同。如果不指定这两个属性，浏览器完全根据单元格内容的大小，决定新增网格的列宽和行高。



## 项目属性

### grid-column-start 

### grid-column-end 

###  grid-row-start 

### grid-row-end 

- `grid-column-start`属性：左边框所在的垂直网格线
- `grid-column-end`属性：右边框所在的垂直网格线
- `grid-row-start`属性：上边框所在的水平网格线
- `grid-row-end`属性：下边框所在的水平网格线

```css
.item-1 {
  grid-column-start: 2;
  grid-column-end: 4;
}
```



这四个属性的值还可以使用`span`关键字，表示"跨越"，即左右边框（上下边框）之间跨越多少个网格。

```css
.item-1 {
  grid-column-start: span 2;
}
```



### grid-column 

### grid-row 

`grid-column`属性是`grid-column-start`和`grid-column-end`的合并简写形式，`grid-row`属性是`grid-row-start`属性和`grid-row-end`的合并简写形式。

```css
.item-1 {
  grid-column: 1 / 3;
  grid-row: 1 / 2;
}
/* 等同于 */
.item-1 {
  grid-column-start: 1;
  grid-column-end: 3;
  grid-row-start: 1;
  grid-row-end: 2;
}
```



### grid-area

`grid-area`属性指定项目放在哪一个区域。

```css
.item-1 {
  grid-area: e;
}
```



`grid-area`属性还可用作`grid-row-start`、`grid-column-start`、`grid-row-end`、`grid-column-end`的合并简写形式，直接指定项目的位置。

```css
.item {
  grid-area: <row-start> / <column-start> / <row-end> / <column-end>;
}
```



### justify-self 

### align-self 

### place-self 

`justify-self`属性设置单元格内容的水平位置（左中右），跟`justify-items`属性的用法完全一致，但只作用于单个项目。

`align-self`属性设置单元格内容的垂直位置（上中下），跟`align-items`属性的用法完全一致，也是只作用于单个项目。

```css
.item {
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
}
```





| 模块           | 功能                                                         | 产品端   | 开发时长预估 | 优先级 | 进度(+jira链接) | 负责人 | 实际时长 | 产出 | 备注 |
| :------------- | :----------------------------------------------------------- | :------- | :----------- | :----- | :-------------- | :----- | :------- | :--- | :--- |
| 生产作业       | 权限配置-新增权限<br />- 给PC权限配置增加生产作业及下属模块<br />- 给APP权限配置增加生产作业及下属模块 | 运营平台 | 0.2          | 高     |                 | @石磊  |          |      |      |
| 生产作业       | 权限配置-新增权限<br />- PC权限配置中增加生产作业的权限，支持一级目录到四级目录的显示<br />- APP权限配置中增加生产作业模块及下属功能扫码报工的权限 | PC平台端 | 0.2          | 高     |                 | @石磊  |          |      |      |
| 生产作业       | 新增左侧菜单<br />智能生产                 1级<br/>  生产作业               2级<br/>   基础数据维护          3级<br/>     操作员工序配置      4级<br/>     工序提交配置        4级<br/>     报工综合配置        4级<br/>   报工管理              3级<br/>     生产进度            4级<br/>     报工记录            4级 | PC平台端 | 0.3          | 高     |                 | @石磊  |          |      |      |
| 操作员工序配置 | 操作员工序配置-页面搭建<br />- 操作员、工号、工序查询功能<br />- 操作员工序配置列表展示 | PC平台端 | 0.4          | 高     |                 | @石磊  |          |      |      |
| 工序提交配置   | 工序提交配置-页面搭建<br />- 工序名称查询功能<br />- 工序提交配置列表展示<br />- 新增页面配置<br />- 查看页面配置<br />- 修改页面配置 | PC平台端 | 1            | 高     |                 | @石磊  |          |      |      |
| 报工综合配置   | 报工综合配置-页面搭建<br />- 报工综合配置列表<br />- 查看报工配置详情<br />- 修改报工配置 | PC平台端 | 0.7          | 高     |                 | @石磊  |          |      |      |
| 生产进度       | 生产进度-页面搭建<br />- 支持报工时间、生产卡号、客户、订单号、状态的查询<br />- 生产进度列表<br />- 查看生产进度<br />- 修改：产出总量、可产出匹数 | PC平台端 | 1            | 高     |                 | @石磊  |          |      |      |
| 报工记录       | 报工记录-页面搭建<br />- 支持报工时间、生产卡号、操作员、工序的查询功能<br />- 报工记录列表<br />- 查看报工记录<br />- 修改：机台、车号 | PC平台端 | 1            | 高     |                 | @石磊  |          |      |      |
| 扫码报工       | 首页-新增扫码报工入口                                        | APP      | 0.1          | 高     |                 | @石磊  |          |      |      |
| 扫码报工       | 扫码报工-页面搭建及功能实现<br />- 一维码、二维码的扫码功能<br />- 手动输入生产卡号功能<br />- 手电筒开关功能<br />- 支持提交后的首页、报工记录页、扫码器页的三个页面跳转 | APP      | 1.7          | 高     |                 | @石磊  |          |      |      |
| 报工记录       | 我的-新增报工记录入口                                        | APP      | 0.1          | 高     |                 | @石磊  |          |      |      |
| 报工记录       | 报工记录-页面搭建<br />- 显示用户报工次数与产量的当日和本月的统计<br />- 显示用户每条报工信息的列表展示 | APP      | 0.5          | 高     |                 | @石磊  |          |      |      |
| 记录详情       | 记录详情-页面搭建<br />- 显示报工记录详情                    | APP      | 0.3          | 高     |                 | @石磊  |          |      |      |

