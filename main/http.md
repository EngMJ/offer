## 网络

### 1\. 网络七层协议（OSI模型）

OSI是一个定义良好的协议规范集，并有许多可选部分完成类似的任务。它定义了开放系统的层次结构、层次之间的相互关系以及各层所包括的可能的任务，作为一个框架来协调和组织各层所提供的服务。

OSI参考模型并没有提供一个可以实现的方法，而是描述了一些概念，用来协调进程间通信标准的制定。即OSI参考模型并不是一个标准，而是一个在制定标准时所使用的概念性框架。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae50a27269bb4ca9bbb8bc612c208e4d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

+   第7层 **应用层** 应用层（Application Layer）提供为应用软件而设计的接口，以设置与另一应用软件之间的通信。例如：HTTP、HTTPS、FTP、Telnet、SSH、SMTP、POP3等。

+   第6层 **表示层** 表示层（Presentation Layer）把数据转换为能与接收者的系统格式兼容并适合传输的格式。

+   第5层 **会话层** 会话层（Session Layer）负责在数据传输中设置和维护计算机网络中两台计算机之间的通信连接。

+   第4层 **传输层** 传输层（Transport Layer）把传输表头（TH）加至数据以形成数据包。传输表头包含了所使用的协议等发送信息。例如:传输控制协议（TCP）等。

+   第3层 **网络层** 网络层（Network Layer）决定数据的路径选择和转寄，将网络表头（NH）加至数据包，以形成分组。网络表头包含了网络资料。例如:互联网协议（IP）等。

+   第2层 **数据链路层** 数据链路层（Data Link Layer）负责网络寻址、错误侦测和改错。当表头和表尾被加至数据包时，会形成信息框（Data Frame）。数据链表头（DLH）是包含了物理地址和错误侦测及改错的方法。数据链表尾（DLT）是一串指示数据包末端的字符串。例如以太网、无线局域网（Wi-Fi）和通用分组无线服务（GPRS）等。


分为两个子层：逻辑链路控制（logical link control，LLC）子层和介质访问控制（Media access control，MAC）子层。

+   第1层 **物理层** 物理层（Physical Layer）在局部局域网上发送数据帧（Data Frame），它负责管理电脑通信设备和网络媒体之间的互通。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡等。

### 2\. TCP 协议三次握手

+   客户端通过 `SYN` 报文段发送连接请求，确定服务端是否开启端口准备连接。状态设置为 `SYN_SEND`;
+   服务器如果有开着的端口并且决定接受连接，就会返回一个 `SYN+ACK` 报文段给客户端，状态设置为 `SYN_RECV`；
+   客户端收到服务器的 `SYN+ACK` 报文段，向服务器发送 `ACK` 报文段表示确认。此时客户端和服务器都设置为 `ESTABLISHED` 状态。连接建立，可以开始数据传输了。

翻译成大白话就是： **客户端**：你能接收到我的消息吗？ **服务端**：可以的，那你能接收到我的回复吗？ **客户端**：可以，那我们开始聊正事吧。

**为什么是3次？**：避免历史连接，确认客户端发来的请求是这次通信的人 **为什么不是4次？**：3次够了第四次浪费

### 3\. TCP 协议四次挥手

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd751c4c2c914db0b38e70e83714d3e6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

1.  为什么不是两次？
    +   两次情况客户端说完结束就立马断开不再接收，无法确认服务端是否接收到断开消息，但且服务端可能还有消息未发送完。
2.  为什么不是三次？
    +   3次情况服务端接收到断开消息，向客户端发送确认接受消息，客户端未给最后确认断开的回复。

## http

### 1\. HTTP 请求报文结构

1.  首行是**Request-Line**包括：**请求方法**，**请求URI**，**协议版本**，**CRLF**
2.  首行之后是若干行**请求头**，包括**general-header**，**request-header**或者**entity-header**，每个一行以CRLF结束
3.  请求头和消息实体之间有一个**CRLF分隔**
4.  根据实际请求需要可能包含一个**消息实体** 一个请求报文例子如下：

```sh
GET /Protocols/rfc2616/rfc2616-sec5.html HTTP/1.1
Host: www.w3.org
Connection: keep-alive
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36
Referer: https://www.google.com.hk/
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: authorstyle=yes
If-None-Match: "2cc8-3e3073913b100"
If-Modified-Since: Wed, 01 Sep 2004 13:24:52 GMT

name=qiu&age=25
```

### 2\. HTTP 响应报文结构

+   首行是状态行包括：**HTTP版本，状态码，状态描述**，后面跟一个CRLF
+   首行之后是**若干行响应头**，包括：**通用头部，响应头部，实体头部**
+   响应头部和响应实体之间用**一个CRLF空行**分隔
+   最后是一个可能的**消息实体** 响应报文例子如下：

```sh
HTTP/1.1 200 OK
Date: Tue, 08 Jul 2014 05:28:43 GMT
Server: Apache/2
Last-Modified: Wed, 01 Sep 2004 13:24:52 GMT # 最后修改时间，用于协商缓存
ETag: "40d7-3e3073913b100" # 文件hash，用于协商缓存
Accept-Ranges: bytes
Content-Length: 16599
Cache-Control: max-age=21600 # 强缓存（浏览器端）最大过期时间
Expires: Tue, 08 Jul 2014 11:28:43 GMT # 强缓存（浏览器端）过期时间
P3P: policyref="http://www.w3.org/2001/05/P3P/p3p.xml"
Content-Type: text/html; charset=iso-8859-1

{"name": "qiu", "age": 25}
```

### 3\. HTTP常见状态码及其含义

+   1XX：信息状态码
    +   100 Continue 继续，一般在发送post请求时，已发送了http header之后服务端将返回此信息，表示确认，之后发送具体参数信息
+   2XX：成功状态码
    +   200 OK 正常返回信息
    +   201 Created 请求成功并且服务器创建了新的资源
    +   202 Accepted 服务器已接受请求，但尚未处理
+   3XX：重定向
    +   301 Moved Permanently 请求的网页已永久移动到新位置。
    +   302 Found 临时性重定向。
    +   303 See Other 临时性重定向，且总是使用 GET 请求新的 URI。
    +   304 Not Modified 自从上次请求后，请求的网页未修改过。
+   4XX：客户端错误
    +   400 Bad Request 服务器无法理解请求的格式，客户端不应当尝试再次使用相同的内容发起请求。
    +   401 Unauthorized 请求未授权。
    +   403 Forbidden 禁止访问。
    +   404 Not Found 找不到如何与 URI 相匹配的资源。
+   5XX: 服务器错误
    +   500 Internal Server Error 最常见的服务器端错误。
    +   503 Service Unavailable 服务器端暂时无法处理请求（可能是过载或维护）。

### [4\. 常见web安全及防护](safe.md)

### 5\. http1.0、http1.1、http2.0区别

+ #### http1.0 vs http1.1

1.  **长连接**，Connection请求头的值为Keep-Alive时，客户端通知服务器返回本次请求结果后保持连接，可有效减少TCP的三次握手开销；
2.  **缓存处理**，增加了`Cache-Control`等缓存相关头（看下面详细介绍）；
3.  **请求头增加Host**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
4.  **请求头增加`range`可用于`断点续传`**，它支持只请求资源的某个部分，可用于`断点续传`。
5.  **增加请求方法**：（OPTIONS,PUT, DELETE, TRACE, CONNECT）
6.  **新增了24个状态码**，（如100Continue，发请求体之前先用`请求头`试探一下服务器，再决定要不要发`请求体`）

* #### http1.x vs http2.0

1.  **服务端推送**
2.  **多路复用**，多个请求都在同一个TCP连接上完成
3.  **报文头部压缩**，HTTP2.0可以维护一个字典，差量更新HTTP头部，大大降低因头部传输产生的流量

+ #### http2 vs http3

1.  **替换TCP使用UDP, 使用QUIC协议**
2.  **1RTT建联**，之前需要3RTT建连,降低建连成本
3.  **连接迁移**，任意网络切换野不需要重新连接
4.  **无队头阻塞**，不会像http2那样产生队头阻塞,阻碍后续数据传输

+   参考: [HTTP1.0、HTTP1.1 和 HTTP2.0 的区别](https://www.cnblogs.com/heluan/p/8620312.html "https://www.cnblogs.com/heluan/p/8620312.html")
+   参考: [HTTP3.0](https://www.infoq.cn/article/lddlsa5f21sty04li3hp)

### 6\. Cache-Control

用于判断`强缓存`，也就是是否直接取在客户端缓存文件，不请求后端。

请求头和响应头都可以使用。

**请求时的缓存指令**包括：no-cache、no-store、max-age、max-stale、min-fresh、only-if- cached， **响应消息中的指令**包括：public、private、no-cache、no-store、no-transform、must- revalidate、proxy-revalidate、max-age。

常用指令含义如下：

+   `no-cache`：不使用本地缓存。需要使用缓存协商，先与服务器确认返回的响应是否被更改，如果之前的响应中存在ETag，那么请求的时候会与服务端验证，如果资源未被更改，则可以避免重新下载。
+   `no-store`：直接禁止游览器缓存数据，每次用户请求该资源，都会向服务器发送一个请求，每次都会下载完整的资源。
+   `public`：可以被所有的用户缓存，包括终端用户和CDN等中间代理服务器。
+   `private`：只能被终端用户的浏览器缓存，不允许CDN等中继缓存服务器对其缓存。
+   `max-age`：指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。

---

### 7\. GET 和 POST 的区别

| 特性 | GET | POST |
|------|-----|------|
| 参数位置 | URL 查询字符串 | 请求体 |
| 长度限制 | 受 URL 长度限制（约 2KB） | 无限制 |
| 安全性 | 参数暴露在 URL，不安全 | 相对安全 |
| 缓存 | 可被缓存 | 默认不缓存 |
| 幂等性 | 幂等（多次请求结果相同） | 非幂等 |
| 后退/刷新 | 无害 | 数据会被重新提交 |
| 书签 | 可收藏为书签 | 不可收藏 |
| 数据类型 | 仅支持 ASCII | 支持多种编码 |

**使用场景**：
- GET：获取数据、搜索、查询
- POST：提交表单、上传文件、创建资源

---

### 8\. HTTP 和 HTTPS 的区别

| 特性 | HTTP | HTTPS |
|------|------|-------|
| 端口 | 80 | 443 |
| 安全性 | 明文传输，不安全 | SSL/TLS 加密，安全 |
| 证书 | 不需要 | 需要 CA 证书 |
| 性能 | 较快 | 略慢（加密开销） |
| SEO | 无影响 | 搜索引擎更青睐 |

**HTTPS 工作流程**：
1. 客户端发起 HTTPS 请求
2. 服务器返回证书（包含公钥）
3. 客户端验证证书有效性
4. 客户端生成随机对称密钥，用公钥加密后发送
5. 服务器用私钥解密获得对称密钥
6. 双方使用对称密钥加密通信

---

### 9\. TCP 和 UDP 的区别

| 特性 | TCP | UDP |
|------|-----|-----|
| 连接 | 面向连接（三次握手） | 无连接 |
| 可靠性 | 可靠（确认机制、重传） | 不可靠 |
| 顺序 | 保证顺序 | 不保证顺序 |
| 速度 | 较慢 | 较快 |
| 头部开销 | 20 字节 | 8 字节 |
| 流量控制 | 有 | 无 |
| 拥塞控制 | 有 | 无 |
| 传输方式 | 字节流 | 数据报 |

**使用场景**：
- TCP：网页、邮件、文件传输（可靠性要求高）
- UDP：视频、直播、游戏、DNS（实时性要求高）

---

### 10\. CORS 跨域

**同源策略**：协议、域名、端口都相同才是同源

**跨域解决方案**：

1. **CORS（跨域资源共享）**
```http
# 服务器响应头
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Credentials: true
```

2. **JSONP**（仅支持 GET）
```javascript
function callback(data) {
  console.log(data);
}
// <script src="https://api.com?callback=callback"></script>
```

3. **代理服务器**
```javascript
// Vite / Webpack devServer 代理
proxy: {
  '/api': {
    target: 'https://api.example.com',
    changeOrigin: true
  }
}
```

4. **postMessage**（跨窗口通信）

**简单请求 vs 预检请求**：
- 简单请求：GET/HEAD/POST，且 Content-Type 为 text/plain、multipart/form-data、application/x-www-form-urlencoded
- 预检请求：先发 OPTIONS 请求询问服务器是否允许

---

### 11\. WebSocket

**特点**：
- 全双工通信（服务器可主动推送）
- 持久连接（一次握手，持续通信）
- 协议标识：ws:// 或 wss://（加密）
- 较低开销（数据帧头部仅 2-10 字节）

```javascript
// 客户端
const ws = new WebSocket('wss://example.com/socket');

ws.onopen = () => {
  ws.send('Hello Server');
};

ws.onmessage = (event) => {
  console.log('收到消息:', event.data);
};

ws.onclose = () => {
  console.log('连接关闭');
};

ws.onerror = (error) => {
  console.error('错误:', error);
};
```

**与 HTTP 长轮询对比**：
| 特性 | WebSocket | 长轮询 |
|------|-----------|--------|
| 连接 | 持久连接 | 每次新建连接 |
| 方向 | 双向通信 | 服务器响应后才能发送 |
| 开销 | 低 | 高（频繁建立连接） |
| 实时性 | 高 | 较低 |

---

### 12\. HTTP 请求方法

| 方法 | 描述 | 幂等 | 安全 |
|------|------|------|------|
| GET | 获取资源 | ✓ | ✓ |
| POST | 创建资源 | ✗ | ✗ |
| PUT | 更新/替换资源 | ✓ | ✗ |
| PATCH | 部分更新资源 | ✗ | ✗ |
| DELETE | 删除资源 | ✓ | ✗ |
| HEAD | 获取响应头 | ✓ | ✓ |
| OPTIONS | 获取支持的方法 | ✓ | ✓ |
| CONNECT | 建立隧道 | ✗ | ✗ |
| TRACE | 追踪路径 | ✓ | ✗ |

**幂等**：多次执行结果相同
**安全**：不修改资源

---

### 13\. CDN 原理

**CDN（内容分发网络）**：将内容缓存到离用户最近的节点

**工作流程**：
1. 用户访问域名，DNS 解析到 CDN 负载均衡
2. CDN 根据用户 IP、节点负载等选择最优节点
3. 如果节点有缓存，直接返回；否则回源获取
4. 缓存内容到节点，下次直接返回

**优点**：
- 加速访问（就近获取）
- 减轻源站压力
- 提高可用性（节点故障自动切换）
- 抵御 DDoS 攻击