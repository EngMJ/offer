## 二、HTML

### 1\. DOCTYPE 的作用

`<!DOCTYPE html>` 声明位于文档最前面，告诉浏览器以哪种 HTML 或 XHTML 规范解析页面。

**作用**：
- 触发浏览器的**标准模式**（Standards Mode）
- 不声明或声明错误会触发**怪异模式**（Quirks Mode），按照旧版浏览器方式渲染

**区别**：
- 标准模式：按 W3C 标准解析渲染
- 怪异模式：模拟老式浏览器行为，盒模型等解析方式不同

---

### 2\. HTML5 新特性

**新增功能**：HTML5 现在已经不是 SGML 的子集，主要是关于图像，位置，存储，多任务等功能的增加

+   新增选择器 document.querySelector、document.querySelectorAll
+   拖拽释放(Drag and drop) API
+   媒体播放的 video 和 audio
+   本地存储 localStorage 和 sessionStorage
+   离线应用 manifest
+   桌面通知 Notifications
+   语意化标签 article、footer、header、nav、section
+   增强表单控件 calendar、date、time、email、url、search
+   地理位置 Geolocation
+   多任务 webworker
+   全双工通信协议 websocket
+   历史管理 history
+   跨域资源共享(CORS) Access-Control-Allow-Origin
+   页面可见性改变事件 visibilitychange
+   跨窗口通信 PostMessage
+   Form Data 对象
+   绘画 canvas

**移除的元素**：

+   纯表现的元素：basefont、big、center、font、 s、strike、tt、u
+   对可用性产生负面影响的元素：frame、frameset、noframes

---

### 4\. viewport

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no" />
<!-- 
    width    设置viewport宽度，为一个正整数，或字符串‘device-width’
    device-width  设备宽度
    height   设置viewport高度，一般设置了宽度，会自动解析出高度，可以不用设置
    initial-scale    默认缩放比例（初始缩放比例），为一个数字，可以带小数
    minimum-scale    允许用户最小缩放比例，为一个数字，可以带小数
    maximum-scale    允许用户最大缩放比例，为一个数字，可以带小数
    user-scalable    是否允许手动缩放
-->
```

**延伸提问**：怎样处理 移动端 1px 被 渲染成 2px问题？

+   局部处理
    +   meta标签中的 viewport属性 ，initial-scale 设置为 1
    +   rem按照设计稿标准走，外加利用transfrome 的scale(0.5) 缩小一倍即可；
+   全局处理
    +   mate标签中的 viewport属性 ，initial-scale 设置为 0.5
    +   rem 按照设计稿标准走即可

### 5\. HTML 文档解析

+   [html文档解析为AST语法树](html_ast.md)

---

### 6\. 语义化标签

**什么是语义化**：使用正确的标签表达正确的内容结构

**好处**：
- 便于 SEO，搜索引擎更好理解页面结构
- 便于阅读和维护，代码可读性更强
- 便于屏幕阅读器等辅助设备解析（无障碍）
- 在 CSS 加载失败时也能保持基本结构

**常用语义化标签**：

| 标签 | 说明 |
|------|------|
| `<header>` | 页面或区块的头部 |
| `<nav>` | 导航链接 |
| `<main>` | 文档主要内容 |
| `<article>` | 独立的内容区块 |
| `<section>` | 文档中的章节 |
| `<aside>` | 侧边栏内容 |
| `<footer>` | 页面或区块的底部 |
| `<figure>` | 图片、图表等独立内容 |
| `<figcaption>` | figure 的标题 |
| `<time>` | 时间日期 |

---

### 7\. src 和 href 的区别

| 属性 | 用途 | 资源处理 | 阻塞 |
|------|------|----------|------|
| `src` | 引入资源替换当前元素 | 下载并嵌入文档 | 会阻塞页面解析 |
| `href` | 建立当前文档与资源的链接 | 并行下载 | 不阻塞解析 |

```html
<!-- src：img、script、iframe、video、audio -->
<script src="app.js"></script>
<img src="logo.png">

<!-- href：link、a -->
<link href="style.css" rel="stylesheet">
<a href="page.html">链接</a>
```

---

### 8\. script 标签的 async 和 defer

```html
<script src="app.js"></script>           <!-- 阻塞解析，立即执行 -->
<script src="app.js" async></script>     <!-- 异步加载，加载完立即执行 -->
<script src="app.js" defer></script>     <!-- 异步加载，DOMContentLoaded 前执行 -->
```

| 属性 | 加载 | 执行时机 | 执行顺序 | 适用场景 |
|------|------|----------|----------|----------|
| 无 | 同步 | 立即 | 按顺序 | 需要立即执行的关键脚本 |
| async | 异步 | 加载完立即 | 不保证 | 独立脚本如统计代码 |
| defer | 异步 | DOM 解析完成后 | 按顺序 | 依赖 DOM 的脚本 |

---

### 9\. meta 标签

```html
<!-- 字符编码 -->
<meta charset="UTF-8">

<!-- 视口设置（移动端适配） -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- SEO 相关 -->
<meta name="description" content="页面描述">
<meta name="keywords" content="关键词1,关键词2">
<meta name="author" content="作者">
<meta name="robots" content="index,follow">

<!-- HTTP 等效 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta http-equiv="refresh" content="5;url=http://example.com">
<meta http-equiv="Cache-Control" content="no-cache">

<!-- 社交媒体（Open Graph） -->
<meta property="og:title" content="标题">
<meta property="og:description" content="描述">
<meta property="og:image" content="图片URL">
```

---

### 10\. Web Workers

**Web Worker**：在后台线程执行 JavaScript，不阻塞主线程

```javascript
// 主线程
const worker = new Worker('worker.js');
worker.postMessage({ data: 'hello' });
worker.onmessage = (e) => console.log(e.data);

// worker.js
self.onmessage = (e) => {
  // 执行耗时计算
  self.postMessage(result);
};
```

**限制**：
- 无法访问 DOM
- 无法访问 window、document 对象
- 通过 postMessage 通信

**Service Worker**：可拦截网络请求，实现离线缓存、消息推送

```javascript
// 注册
navigator.serviceWorker.register('/sw.js');

// sw.js
self.addEventListener('fetch', (e) => {
  e.respondWith(caches.match(e.request) || fetch(e.request));
});
```

---

### 11\. 常见的行内元素和块级元素

**块级元素**：独占一行，可设置宽高
- `<div>`, `<p>`, `<h1>-<h6>`, `<ul>`, `<ol>`, `<li>`, `<table>`, `<form>`, `<header>`, `<footer>`, `<section>`

**行内元素**：不独占一行，宽高由内容决定
- `<span>`, `<a>`, `<strong>`, `<em>`, `<img>`, `<input>`, `<button>`, `<label>`

**行内块元素**：不独占一行，但可设置宽高
- `<img>`, `<input>`, `<button>`, `<textarea>`, `<select>`

---

### 12\. iframe 的优缺点

**优点**：
- 加载其他页面内容
- 实现微前端隔离
- 加载广告等第三方内容

**缺点**：
- 阻塞主页面 onload 事件
- 不利于 SEO
- 共享连接池，影响并行加载
- 增加 HTTP 请求

**优化**：使用 JavaScript 动态加载

```javascript
document.getElementById('iframe').src = 'page.html';
```

---

### 13\. Canvas 和 SVG 的区别

| 特性 | Canvas | SVG |
|------|--------|-----|
| 类型 | 位图（像素） | 矢量图 |
| 缩放 | 会失真 | 不失真 |
| 操作 | JavaScript 绑定像素 | DOM 操作元素 |
| 事件 | 整体绑定 | 每个元素可绑定 |
| 性能 | 大量图形时更优 | 少量图形时更优 |
| 适用 | 游戏、图像处理 | 图标、图表、地图 |
