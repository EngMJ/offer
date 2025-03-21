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
+   长列表滚动到可视区域动态加载（大数据渲染）参考: [虚拟列表](virtual_list.md)
+   图片懒加载（data-src）参考: [1. vue/原生懒加载](vue_img_lazy.md) [2.react懒加载](react_img_lazy.md)
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

+   [从输入URL到浏览器显示页面过程中都发生了什么](url.md)

### 模块化的理解

常见模块化方案: **AMD**、**CMD**、**CommonJS** 和 **ES Modules (ESM)**

**模块化的演进过程:**

+   1.文件划分的方式 污染全局作用域 命名冲突 无法管理模块依赖关系
+   2.命名空间方式 在第一个阶段的基础上 将每个模块只暴露一个全局对象 所有的变量都挂载到这个全局对象上
+   3.IIFE 立即执行函数 为模块提供私有空间
+   以上是早起在没有工具和规范的情况下对模块化的落地方式

---

#### 1. AMD（Asynchronous Module Definition）

+ 特点:

**异步加载**：设计初衷就是在浏览器环境下异步加载模块，避免页面阻塞。

**依赖提前声明**：在定义模块时就明确写出所有依赖，加载时并行请求所有依赖模块。

**典型实现**：[RequireJS](https://requirejs.org/) 就是基于 AMD 规范的实现。

+ 语法示例:
```javascript
// 使用 define() 来定义一个模块，依赖以数组形式提前声明
define(['moduleA', 'moduleB'], function(moduleA, moduleB) {
    // 模块内部代码
    var result = moduleA.doSomething() + moduleB.doSomethingElse();
    return {
        result: result
    };
});
```

+ 优缺点:

**优点**：非常适合浏览器环境，能实现模块的并行加载，提高性能。

**缺点**：语法略显冗长；由于依赖必须提前声明/加载，灵活性相对较差。

---

#### 2. CMD（Common Module Definition）

+ 特点:

**就近依赖**：由 SeaJS 等库提出，强调“延迟执行”和“就近依赖”。即依赖不必在模块最开始就声明，而是在需要的时候调用 `require` 加载。

**适用于浏览器端**：与 AMD 一样，主要面向浏览器环境，但更注重动态依赖和按需加载。

+ 语法示例:
```javascript
define(function(require, exports, module) {
    // 当运行时才加载依赖
    var moduleA = require('moduleA');
    
    exports.doSomething = function() {
        // 使用 moduleA
        moduleA.action();
    };
});
```

+ 优缺点:

**优点**：灵活性较好，允许在代码中动态地加载模块，适合依赖在运行时才能确定的场景。

**缺点**：由于依赖不是在定义阶段明确列出，静态分析和工具优化（如打包工具的依赖图构建）会比较困难。

---

#### 3. CommonJS

+ 特点:

**简单直观**：文件划分模块,使用 `require` 引入模块，使用 `module.exports` 或 `exports` 导出模块。

**同步加载**：主要用于服务器端（如 Node.js），因为在服务器环境中文件读取是本地操作，速度快，不存在浏览器的网络延迟问题。

**动态加载**：可以通过代码逻辑动态定义加载模块。

**顺序加载**：按照代码中的加载顺序进行加载。

**导出值克隆**：导出的内容是当前模块的值克隆,改变加载的内容并不影响原模块。

**模块缓存**：模块只会在首次加载时执行一次，后续加载直接使用缓存。

+ 语法示例:
```javascript
// a.js 模块 导出内容
module.exports = function() {
    console.log("Hello from module A");
};

// b.js 模块 加载内容
var moduleA = require('./a.js');
moduleA();
```

+ 优缺点:

**优点**：语法简单易懂，非常适合服务端开发；模块加载和缓存机制明确。

**缺点**：同步加载不适合浏览器端（会导致阻塞），但通常可以通过构建工具（如 Browserify 或 Webpack）进行转换。

---

#### 4. ES Modules (ESM)

+ 特点:

**语言级支持**：ES6（ECMAScript 2015）标准正式引入，成为 JavaScript 官方的模块系统。

**双端支持**：既支持浏览器（通过 `<script type="module">` 加载），也支持 Node.js（新版本中已原生支持）。

**静态加载**：使用 import 和 export 语法来引入和导出模块，所有依赖都必须在模块顶层静态声明，利于静态分析、打包优化和 tree shaking。

**引用加载**：通过CORS方式进行模块加载,且加载的ES模块为值引用,修改加载模块会影响原模块。

**自动严格模式**：模块中语法使用严格模式检查。

**可加载commonJS模块**：ES module中可以导入CommonJS模块,但CommonJS中不能导入ES module模块。

**浏览器支持高**: 浏览器可异步/延迟/并行加载模块,避免阻塞页面渲染.

+ 语法示例:
```javascript
// a.mjs 或 a.js 模块（根据环境配置）
export function doSomething() {
    console.log("Doing something");
}

// b.mjs 或 b.js 模块
import { doSomething } from './a.js';
doSomething();
```

+ 优缺点

**优点**：语法简洁、明确；静态结构使得优化和安全性更好；逐渐成为未来的标准模块系统。

**缺点**：老版本的 Node.js 和部分浏览器可能不支持，需要转译（例如通过 Babel）或者使用特定的文件扩展名（如 `.mjs`）。

---

#### 模块化方案的异同

| 模块系统             | 加载方式          | 语法风格                                  | 依赖声明方式                     | 主要应用场景                             | 模块划分方式                                                                      |
|----------------------|---------------|-------------------------------------------|----------------------------------|------------------------------------------|-----------------------------------------------------------------------------|
| **AMD**              | 异步加载          | `define([...], function(){})`             | 在模块定义时提前声明所有依赖       | 浏览器端（如 RequireJS 等库）              | 灵活：可以在一个文件中定义多个模块，但通常为了便于管理和维护，约定每个模块写在单独的文件中。打包工具也可将多个模块合并以减少 HTTP 请求。     |
| **CMD**              | 异步加载（就近依赖）    | `define(function(require, exports, module){})` | 根据需要动态加载依赖              | 浏览器端（如 SeaJS 等库）                  | 灵活：虽然允许在一个文件中定义多个模块，但实际开发中也通常采用每个模块单独一个文件的方式，以提高代码清晰度和维护性。                  |
| **CommonJS**         | 同步加载 (可动态加载)  | `require()` / `module.exports`            | 在运行时调用                     | Node.js 服务端                           | 文件即模块：Node.js 中每个文件都有独立作用域，惯例是每个文件作为一个独立模块，这种方式简洁直观，有利于模块缓存和管理。             |
| **ES Modules (ESM)** | 静态加载（可配合异步加载） | `import` / `export`                       | 静态声明（编译时就能确定依赖）      | 浏览器和 Node.js（现代前后端均适用）         | 文件就是模块：任何使用 `import`/`export` 的文件都被视为独立模块。这种方式便于静态分析和 tree shaking，推荐用于新项目。 |

---

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
