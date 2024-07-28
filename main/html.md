## 二、html

### html5有哪些新特性、移除了那些元素？

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

### viewport

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

### html文档解析

+   [html文档解析为AST语法树](html_ast.md)
