## 三、css

### 1\. css选择器优先级

`!important > inline > id > class > tag > * > inherit > default`

+   !important：大于其他
+   行内：1000
+   id选择器：100
+   类，伪类和属性选择器，如.content：10
+   类型选择器和伪元素选择器：1
+   通配符、子选择器、相邻选择器：0

同级别的后写的优先级高。

相同class样式，css中后写的优先级高，和html中的class名字前后无关

### 2\. 水平垂直居中

+   文本水平居中：`text-algin: center`
+   文本垂直居中：`line-height`等于容器`height`；`display: flex; algin-items: center;`
+   div水平居中：
    1.  margin: 0 auto;
    2.  已知父元素宽度：margin-left: width / 2; transform: tranlateX(-50%)
    3.  未知父元素宽度：position: absolute: top: 50%; transform: tranlateY(-50%)
    4.  display: flex; justify-content: center;
+   div垂直居中：
    1.  已知父元素高度：margin-top: height / 2; transform: tranlateY(-50%)
    2.  未知父元素高度：position: absolute: top: 50%; transform: tranlateY(-50%)
    3.  display: flex; algin-items: center;

### 3\. 移除inline-block间隙

+   移除空格
+   使用margin负值
+   使用font-size:0
+   letter-spacing
+   word-spacing

### 4\. 清除浮动

**浮动的影响**： （1）由于浮动元素脱离了文档流，所以父元素的高度无法被撑开，影响了与父元素同级的元素 （2）与浮动元素同级的非浮动元素会跟随其后，因为浮动元素脱离文档流不占据原来的位置 （3）如果该浮动元素不是第一个浮动元素，则该元素之前的元素也需要浮动，否则容易影响页面的结构显示 **清除浮动的3个方法**：

1.  使用`clear: both`清除浮动 在**浮动元素后面**放一个空的div标签，div设置样式clear:both来清除浮动。它的优点是简单，方便兼容性好，但是一般情况下不建议使用该方法，因为会造成结构混乱，不利于后期维护。

2.  利用伪元素`after`来清除浮动 给**父元素**添加了:after伪元素，通过清除伪元素的浮动，达到撑起父元素高度的目的


```css
.clearfix:after{
    content: "";
    display: block;
    visibility: hidden;
    clear: both;
}
```

3.  使用CSS的`overflow`属性 当给**父元素**设置了overflow样式，不管是overflow:hidden或overflow:auto都可以清除浮动只要它的值不为visible就可以了，它的本质就是建构了一个BFC，这样使得达到撑起父元素高度的效果

+   参考：[css清除浮动](https://www.html.cn/qa/css3/13783.html "https://www.html.cn/qa/css3/13783.html")

### 5\. （外）边距重叠

布局垂直方向上两个元素的间距不等于margin的和，而是取较大的一个

1.  同级相邻元素 **现象**：上方元素设`置margin-bottom: 20px`，下方元素设置`margin-top: 10px`，实际的间隔是`20px` **避免办法**：同级元素不要同时设置，可都设置margin-bottom或margin-top的一个，或者设置padding

2.  父子元素 **现象**：父元素设`置margin-top: 20px`，下方元素设置`margin-top: 10px`，实际的间隔是`20px` **避免办法**：父元素有padding-top，或者border-top。或者触发`BFC`


### 6\. 三栏布局

**要求**：左边右边固定宽度，中间自适应

1.  float

```html
<style>
.left{
    float: left;
    width: 300px;
    height: 100px;
    background: #631D9F;
}
.right{
    float: right;
    width: 300px;
    height: 100px;
    background: red;
}
.center{
    margin-left: 300px;
    margin-right: 300px;
    background-color: #4990E2;
}
</style>
<body>
    <div class="left">左</div>
    <div class="right">右</div>
    <div class="center">中</div>  
    <!-- center如果在中间，则right会被中间block的元素挤到下一行 -->
</body>
```

2.  position

```html
<style>
.left{
    position: absolute;
    left: 0;
    width: 300px;
    background-color: red;
}
.center{
    position: absolute;
    left: 300px;
    right: 300px;
    background-color: blue;
}
.right{
    position: absolute;
    right: 0;
    width: 300px;
    background-color: #3A2CAC;
}
</style>
<body>
    <div class="left">左</div>
    <div class="center">中</div>  
    <div class="right">右</div>
</body>
```

3.  flex

```html
<style>
.main {
    display: flex;
}
.left{
    width: 300px;
}
.center{
    flex-grow: 1;
    flex-shrink: 1;
}
.right{
    width: 300px;
}
</style>
<body class="main">
    <div class="left">左</div>
    <div class="center">中</div>  
    <div class="right">右</div>
</body>
```

### 7\. BFC

> `BFC`是`Block Formatting Context`，也就是`块级格式化上下文`，是用于布局块级盒子的一块渲染区域。

简单来说，BFC 实际上是一块区域，在这块区域中遵循一定的规则，有一套独特的渲染规则。

文档流其实分为`普通流`、`定位流`和`浮动流`和三种，普通流其实就是指BFC中的FC，也即`格式化上下文`。

**普通流**：元素按照其在 HTML 中的先后位置**从上到下、从左到右**布局，在这个过程中，行内元素水平排列，直到当行被占满然后换行，块级元素则会被渲染为完整的一个新行。

**格式化上下文**：页面中的一块渲染区域，有一套渲染规则，决定了其子元素如何布局，以及和其他元素之间的关系和作用

**§ BFC的几条规则：** 1）BFC 区域内的元素外边距会发生重叠。  
2）BFC 区域内的元素不会与浮动元素重叠。  
3）计算 BFC 区域的高度时，浮动元素也参与计算。  
4）BFC 区域就相当于一个容器，内部的元素不会影响到外部，同样外部的元素也不会影响到内部。

**§ BFC的应用：**

1.  清除浮动：父元素设置overflow: hidden触发BFC实现清除浮动，防止父元素高度塌陷，后面的元素被覆盖，实现文字环绕等等。
2.  消除相邻元素垂直方向的边距重叠：第二个子元素套一层，并设置overflow: hidden，构建BFC使其不影响外部元素。
3.  消除父子元素边距重叠，父元素设置overflow: hidden

**§ 触发BFC的方式：**

1.  float 不为 none，浮动元素所在的区域就是一个 BFC 区域。
2.  position 的值不是 static 或 relative 的元素所在的区域就是一个 BFC 区域
3.  display为 table-cell 的表格单元格元素所在的区域也是一个 BFC 区域
4.  overflow 不为 visible 的元素所在的区域也是一个 BFC 区域

+   　参考：[CSS 中你应该了解的 BFC](https://juejin.cn/post/6904568682555375624 "https://juejin.cn/post/6904568682555375624")

### 8\. flex布局

弹性布局，Flex 布局将成为未来布局的首选方案。 **兼容性：**

1.  Webkit 内核的浏览器，必须加上-webkit前缀。
2.  ie9不支持

**基本概念：**

1.  容器&项目：采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。
2.  主轴&交叉轴：堆叠的方向，默认主轴是水平方向，交叉轴是垂直方向。可通过`flex-derection: column`设置主轴为垂直方向。

**容器属性：**

+   display: flex
+   flex-direction：主轴的方向（即项目的排列方向），row | row-reverse | column | column-reverse;
+   flex-wrap：是否换行，nowrap | wrap | wrap-reverse;
+   flex-flow：direction 和 wrap简写
+   justify-content：主轴对齐方式，flex-start | flex-end | center | space-between | space-around;
+   align-items：交叉轴对齐方式，flex-start | flex-end | center | baseline | stretch;
+   align-content: 多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。flex-start | flex-end | center | space-between | space-around | stretch;

**项目的属性:**

+   order：项目的排列顺序，数值越小，排列越靠前，默认为0。

+   flex-grow：放大比例，默认为0，指定元素分到的剩余空间的比例。

+   flex-shrink：缩小比例，默认为1，指定元素分到的缩减空间的比例。

+   flex-basis：分配多余空间之前，项目占据的主轴空间，默认值为auto

+   flex：grow, shrink 和 basis的简写，默认值为0 1 auto

+   align-self：单个项目不一样的对齐方式，默认值为auto，auto | flex-start | flex-end | center | baseline | stretch;

+   参考：[Flex 布局教程：语法篇 - 阮一峰](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html "http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html")


### 9\. CSS动画

**1\. transition过渡** 将变化按照设置的时间长度缓慢执行完毕，而不是立即执行。

`delay`的真正意义在于，它指定了动画发生的顺序，使得多个不同的transition可以连在一起，形成复杂效果。

transition的值是简写，扩展开依次是：

1.  transition-property：过渡属性
2.  transition-duration：过渡时间长度
3.  transition-delay：延迟几秒执行
4.  transition-timing-function
    +   linear：匀速
    +   ease-in：加速
    +   ease-out：减速
    +   cubic-bezier函数：自定义速度模式，最后那个cubic-bezier，可以使用[工具网站](http://cubic-bezier.com/ "http://cubic-bezier.com/")来定制。

```css
/* 变化在1s过渡 */
transition: 1s;
/* 指定过渡属性 */
transition: 1s height;
/* 指定多个属性同时发生过渡 */
transition: 1s height, 1s width;
/* 指定delay延时时间 */
transition: 1s height, 1s 1s width;
/* 指定状态变化速度 */
transition: 1s height ease;
/* 指定自定义移状态变化速度 */
transition: 1s height cubic-bezier(.83,.97,.05,1.44);
```

**transition的局限** transition的优点在于简单易用，但是它有几个很大的局限。

1.  transition需要事件触发，所以没法在网页加载时自动发生。
2.  transition是一次性的，不能重复发生，除非一再触发。
3.  transition只能定义开始状态和结束状态，不能定义中间状态，也就是说只有两个状态。
4.  一条transition规则，只能定义一个属性的变化，不能涉及多个属性。

CSS Animation就是为了解决这些问题而提出的。

+   Transition 强调过渡，Transition ＋ Transform ＝ 两个关键帧的Animation
+   Animation 强调流程与控制，Duration ＋ TransformLib ＋ Control ＝ 多个关键帧的Animation

**2\. animation动画**

```css
.element:hover {
  animation: 1s rainbow;
  /*
  animation: 1s rainbow infinite; 关键字infinite让动画无限次播放
  animation: 1s rainbow 3; 指定动画播放次数
  */
}

@keyframes rainbow {
  0% { background: #c00; }
  50% { background: orange; }
  100% { background: yellowgreen; }
}
```

其中animation的值是简写，扩展开依次是：

1.  animation-name: 指定一个 @keyframes 的名称，动画将要使用这个@keyframes定义。
2.  animation-duration: 整个动画需要的时长。
3.  animation-timing-function: 动画进行中的时速控制，比如 ease 或 linear.
4.  animation-delay: 动画延迟时间。
5.  animation-direction: 动画重复执行时运动的方向。
6.  animation-iteration-count: 动画循环执行的次数。
7.  animation-fill-mode: 设置动画执行完成后/开始执行前的状态，比如，你可以让动画执行完成后停留在最后一幕，或恢复到初始状态。
8.  animation-play-state: 暂停/启动动画。

+   参考：[CSS动画简介](http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html "http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html")

### 10\. CSS优化、提高性能的方法有哪些

+   多个css合并，尽量减少HTTP请求
+   将css文件放在页面最上面
+   移除空的css规则
+   避免使用CSS表达式
+   选择器优化嵌套，尽量避免层级过深
+   充分利用css继承属性，减少代码量
+   抽象提取公共样式，减少代码量
+   属性值为0时，不加单位
+   属性值为小于1的小数时，省略小数点前面的0
+   css雪碧图
