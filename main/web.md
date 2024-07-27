## 一、web

### 前端性能优化

性能评级工具（PageSpeed 或 YSlow）

css

+   CSS优化、提高性能的方法有哪些
+   多个css合并，尽量减少HTTP请求
+   将css文件放在页面最上面
+   移除空的css规则
+   避免使用CSS表达式
+   选择器优化嵌套，尽量避免层级过深
+   充分利用css继承属性，减少代码量
+   抽象提取公共样式，减少代码量
+   属性值为0时，不加单位
+   属性值为小于1的小数时，省略小数点前面的0
+   使用CSS Sprites将多张图片拼接成一张图片，通过CSS background 属性来访问图片内容

js

+   节流、防抖
+   长列表滚动到可视区域动态加载（大数据渲染）
+   图片懒加载（data-src）
+   使用闭包时，在函数结尾手动删除不需要的局部变量，尤其在缓存dom节点的情况下
+   DOM 操作优化
    +   批量添加dom可先`createElement`创建并添加节点，最后一次性加入dom
    +   批量绑定事件，使用`事件委托`绑定父节点实现，利用了事件冒泡的特性
    +   如果可以使用`innerHTML`代替`appendChild`
    +   在 DOM 操作时添加样式时尽量增加 class 属性，而不是通过 style 操作样式，以减少重排（`Reflow`）

网络

+   减少 HTTP 请求数量
+   利用浏览器缓存，公用依赖包（如vue、Jquery、ui组件等）单独打包/单文件在一起，避免重复请求
+   减小`cookie`大小，尽量用`localStorage`代替
+   CDN托管静态文件
+   开启 Gzip 压缩

### 浏览器内核

+   主要分成两部分：渲染引擎(`layout engineer`或`Rendering Engine`)和`JS`引擎

+   渲染引擎：负责取得网页的内容（`HTML`、`XML`、图像等等）、整理讯息（例如加入`CSS`等），以及计算网页的显示方式，然后会输出至显示器或打印机。浏览器的内核的不同对于网页的语法解释会有不同，所以渲染的效果也不相同。所有网页浏览器、电子邮件客户端以及其它需要编辑、显示网络内容的应用程序都需要内核

+   `JS`引擎则：解析和执行`javascript`来实现网页的动态效果

+   最开始渲染引擎和`JS`引擎并没有区分的很明确，后来JS引擎越来越独立，内核就倾向于只指渲染引擎


**常见的浏览器内核有哪些**

+   `Trident`内核：`IE,MaxThon,TT,The World,360`,搜狗浏览器等。\[又称MSHTML\]
+   `Gecko`内核：`Netscape6`及以上版本，`FF,MozillaSuite/SeaMonkey`等
+   `Presto`内核：`Opera7`及以上。 \[`Opera`内核原为：Presto，现为：`Blink`;\]
+   `Webkit`内核：`Safari,Chrome`等。 \[ `Chrome`的`Blink`（`WebKit`的分支）\]

### cookies、sessionStorage、localStorage 和 indexDB 的区别

+   cookie是网站为了标示用户身份而储存在用户本地的数据
+   是否在http请求只能够携带
    +   cookie数据始终在**同源**的http请求中携带，跨域需要设置`withCredentials = true`
    +   sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存
+   存储大小：
    +   cookie数据大小不能超过`4k`；
    +   sessionStorage和localStorage虽然也有存储大小的限制，但比cookie大得多，可以达到`5M`或更大，因不同浏览器大小不同；
+   有效时间：
    +   cookie 设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭
    +   localStorage 硬盘存储持久数据，浏览器关闭后数据不丢失除非主动删除数据
    +   sessionStorage 存在内存中，数据在当前浏览器窗口关闭后自动删除

画表对比：

| 特性 | cookie | localStorage | sessionStorage | indexDB |
| --- | --- | --- | --- | --- |
| 数据生命周期 | 一般由服务器生成，可以设置过期时间 | 除非被清理，否则一直存在 | 页面关闭就清理 | 除非被清理，否则一直存在 |
| 数据存储大小 | **4K** | **5M** | **5M** | 无限 |
| 与服务端通信 | 每次都会携带在 header 中，对于请求性能影响 | 不参与 | 不参与 | 不参与 |

对于 `cookie`，还需要注意安全性：

| 属性 | 作用 |
| --- | --- |
| `value` | 如果用于保存用户登录态，应该将该值加密，不能使用明文的用户标识 |
| `http-only` | 不能通过 `JS`访问 `Cookie`，减少 `XSS`攻击 |
| `secure` | 只能在协议为 `HTTPS` 的请求中携带 |
| `same-site` | 规定浏览器不能在跨域请求中携带 `Cookie`，减少 `CSRF` 攻击 |

### 从输入URL到浏览器显示页面过程中都发生了什么

+   参考：[从输入URL到浏览器显示页面过程中都发生了什么](https://juejin.cn/post/6905931622374342670 "https://juejin.cn/post/6905931622374342670")

### 对AMD、CMD的理解

+   `CommonJS`是服务器端模块的规范，`Node.js`采用了这个规范。`CommonJS`规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。`AMD`规范则是非同步加载模块，允许指定回调函数

+   `AMD`推荐的风格通过返回一个对象做为模块对象，`CommonJS`的风格通过对`module.exports`或`exports`的属性赋值来达到暴露模块对象的目的

+   `AMD`


**es6模块 CommonJS、AMD、CMD**

+   `CommonJS` 的规范中，每个 `JavaScript` 文件就是一个独立的模块上下文（`module context`），在这个上下文中默认创建的属性都是私有的。也就是说，在一个文件定义的变量（还包括函数和类），都是私有的，对其他文件是不可见的。
+   `CommonJS`是同步加载模块,在浏览器中会出现堵塞情况，所以不适用
+   `AMD` 异步，需要定义回调`define`方式
+   `es6` 一个模块就是一个独立的文件，该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用`export`关键字输出该变量 `es6`还可以导出类、方法，自动适用严格模式

### 模块化（AMD、CMD、CommonJS、ES6）

**模块化的演进过程**

+   1.文件划分的方式 污染全局作用域 命名冲突 无法管理模块依赖关系
+   2.命名空间方式 在第一个阶段的基础上 将每个模块只暴露一个全局对象 所有的变量都挂载到这个全局对象上
+   3.IIFE 立即执行函数 为模块提供私有空间
+   以上是早起在没有工具和规范的情况下对模块化的落地方式

模块化规范的出现 模块化规范+模块加载器

**1\. AMD 异步加载**

+   require.js的实现 define('modle', \[加载资源\]， ()=>{})
+   使用起来比较复杂
+   模块js文件请求频繁
+   先加载依赖

```js
// require.js 就是使用的这种风格

define(['a.js', 'b.js'], function(A, B) {
    // do something
})


// 实现思路：建一个node节点, script标签
var node = document.createElement('script')
node.type = 'text/javascript'
node.src = '1.js'

// 1.js 加载完后onload的事件
node.addEventListener('load', function(evt) {
    // 开始加载 2.js
    var node2 = document.createElement('script')
    node2.type = 'text/javascript'
    node2.src = '2.js'
    // 插入 2.js script 节点
    document.body.appendChild(node2)
})
// 将script节点插入dom中
document.body.appendChild(node);
```

**2\. CMD sea.js**

+   sea.js
+   按需加载
+   碰到require('2.js')就立即执行2.js

```js
define(function() {
    var a = require('2.js')
    console.log(33333)
})
```

**3\. commonjs 服务端规范**

+   一个文件就是一个模块
+   每个模块都有单独的作用域
+   通过module.exports导出成员
+   通过require函数载入模块
+   commonjs是以`同步`的方式加载模块 node的执行机制是在启动时去加载模块 在执行阶段不需要加载模块
+   CommonJS 模块输出的是一个值的拷贝，一旦输出一个值，模块内部的变化就影响不到这个值
+   CommonJS 模块加载的顺序，按照其在代码中出现的顺序
+   由于 CommonJS 是同步加载模块的，在服务器端，文件都是保存在硬盘上，所以同步加载没有问题，但是对于浏览器端，需要将文件从服务器端请求过来，那么同步加载就不适用了，所以，CommonJS 是不适用于浏览器端的。
+   CommonJS 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存

**4\. ESmodules 浏览器模块化规范**

+   在语言层面实现了模块化 通过给script的标签 将type设置成module 就可以使用这个规范了
+   基本特性
    +   自动采用严格模式，忽略use strict
    +   每个ESM模块都是单独的私有作用域
    +   ESM是通过CORS去请求外部JS模块的
    +   ESM中的script标签会延迟执行脚本
    +   ES6 模块是动态引用，引用类型属性被改变会相互影响
+   export import 进行导入导出
    +   导出的并不是成员的值 而是内存地址 内部发生改变外部也会改变，外部导入的是只读成员不能修改
    +   ES module中可以导入CommonJS模块
    +   CommonJS中不能导入ES module模块
    +   CommonJS始终只会导出一个默认成员
    +   注意import不是解构导出对象

### 浏览器缓存

> 浏览器缓存分为强缓存和协商缓存。当客户端请求某个资源时，获取缓存的流程如下

+   先根据这个资源的一些 `http header` 判断它是否命中强缓存，如果命中，则直接从本地获取缓存资源，不会发请求到服务器；
+   当强缓存没有命中时，客户端会发送请求到服务器，服务器通过另一些`request header`验证这个资源是否命中协商缓存，称为`http`再验证，如果命中，服务器将请求返回，但不返回资源，而是告诉客户端直接从缓存中获取，客户端收到返回后就会从缓存中获取资源；
+   强缓存和协商缓存共同之处在于，如果命中缓存，服务器都不会返回资源； 区别是，强缓存不对发送请求到服务器，但协商缓存会。
+   当协商缓存也没命中时，服务器就会将资源发送回客户端。
+   当 `ctrl+f5` 强制刷新网页时，直接从服务器加载，跳过强缓存和协商缓存；
+   当 `f5`刷新网页时，跳过强缓存，但是会检查协商缓存；

**强缓存**

+   `Expires`（该字段是 `http1.0` 时的规范，值为一个绝对时间的 `GMT` 格式的时间字符串，代表缓存资源的过期时间）
+   `Cache-Control:max-age`（该字段是 `http1.1`的规范，强缓存利用其 `max-age` 值来判断缓存资源的最大生命周期，它的值单位为秒）

**协商缓存**

+   `Last-Modified`（值为资源最后更新时间，随服务器response返回，即使文件改回去，日期也会变化）
+   `If-Modified-Since`（通过比较两个时间来判断资源在两次请求期间是否有过修改，如果没有修改，则命中协商缓存）
+   `ETag`（表示资源内容的唯一标识，随服务器`response`返回，仅根据文件内容是否变化判断）
+   `If-None-Match`（服务器通过比较请求头部的`If-None-Match`与当前资源的`ETag`是否一致来判断资源是否在两次请求之间有过修改，如果没有修改，则命中协商缓存）

### 浏览器是如何渲染网页的

+   参考：[浏览器渲染页面过程](https://juejin.cn/post/6905931622374342670#heading-6 "https://juejin.cn/post/6905931622374342670#heading-6")

### 重绘（Repaint）和回流（Reflow）

> 重绘和回流会在我们设置节点样式时频繁出现，同时也会很大程度上影响性能。

+   **重绘**：当节点需要更改外观而不会影响布局的，比如改变 `color` 就叫称为重绘
+   **回流**：布局或者几何属性需要改变就称为回流。
+   回流必定会发生重绘，重绘不一定会引发回流。回流所需的成本比重绘高的多，改变父节点里的子节点很可能会导致父节点的一系列回流。

**以下几个动作可能会导致性能问题**：

+   改变 `window` 大小
+   改变字体
+   添加或删除样式
+   文字改变
+   定位或者浮动
+   盒模型

重绘和回流其实也和 `Eventloop` 有关。

+   当 `Eventloop` 执行完 `Microtasks` 后，会判断 `document` 是否需要更新，因为浏览器是 `60Hz` 的刷新率，每 `16.6ms` 才会更新一次。
+   然后判断是否有 `resize` 或者 `scroll` 事件，有的话会去触发事件，所以 `resize` 和 `scroll` 事件也是至少 `16ms` 才会触发一次，并且自带节流功能。
+   判断是否触发了 `media query`
+   更新动画并且发送事件
+   判断是否有全屏操作事件
+   执行 `requestAnimationFrame`回调
+   执行 `IntersectionObserver` 回调，该方法用于判断元素是否可见，可以用于懒加载上，但是兼容性不好 更新界面
+   以上就是一帧中可能会做的事情。如果在一帧中有空闲时间，就会去执行 `requestIdleCallback`回调

**如何减少重绘和回流**：

1.  使用 `transform` 替代 `top`
2.  使用 `visibility` 替换`display: none` ，因为前者只会引起重绘，后者会引发回流（改变了布局）
3.  不要把节点的属性值放在一个循环里当成循环里的变量
4.  不要使用 `table` 布局，可能很小的一个小改动会造成整个 `table` 的重新布局
5.  动画实现的速度的选择，动画速度越快，回流次数越多，也可以选择使用`requestAnimationFrame`
6.  `CSS` 选择符从右往左匹配查找，避免节点层级过多
7.  将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 `video` 标签来说，浏览器会自动将该节点变为图层。
8.  避免使用`css`表达式(`expression`)，因为每次调用都会重新计算值（包括加载页面）
9.  尽量使用 `css` 属性简写，如：用 `border` 代替 `border-width`, `border-style`, `border-color`
10.  批量修改元素样式：`elem.className` 和 `elem.style.cssText` 代替 `elem.style.xxx`
11.  需要要对元素进行复杂的操作时，可以先隐藏(`display:"none"`)，操作完成后再显示
12.  需要创建多个`DOM`节点时，使用`DocumentFragment`创建完后一次性的加入`document`
13.  缓存`Layout`属性值，如：`var left = elem.offsetLeft;` 这样，多次使用 `left` 只产生一次回流

> 设置节点为图层的方式有很多，我们可以通过以下几个常用属性可以生成新图层

+   `will-change`
+   `video`、`iframe` 标签

### 首屏加载优化方案

1.  Vue-Router路由懒加载（利用Webpack的代码切割）
2.  使用CDN加速，将通用的库从vendor进行抽离
3.  Nginx的gzip压缩
4.  Vue异步组件
5.  服务端渲染SSR
6.  如果使用了一些UI库，采用按需加载
7.  Webpack开启gzip压缩
8.  如果首屏为登录页，可以做成多入口
9.  图片懒加载减少占用网络带宽
10.  页面使用骨架屏
11.  利用好script标签的async和defer这两个属性。功能独立且不要求马上执行的js文件，可以加入async属性。如果是优先级低且没有依赖的js，可以加入defer属性。

可利用`performance.timing`看各个步骤的耗时： 白屏时间：`performance.timing.responseStart - performance.timing.navigationStart` 
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee27102c283c42e1b294a23200b2c003~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)
