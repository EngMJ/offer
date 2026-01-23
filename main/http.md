## 网络

### 1. OSI 七层模型

| 层级 | 名称 | 功能 | 协议/设备示例 |
|------|------|------|--------------|
| 7 | 应用层 | 为应用软件提供接口 | HTTP, HTTPS, FTP, SSH |
| 6 | 表示层 | 数据格式转换、加密 | SSL, JPEG |
| 5 | 会话层 | 建立/维护会话连接 | RPC, SQL |
| 4 | 传输层 | 端到端传输控制 | TCP, UDP |
| 3 | 网络层 | 路由选择、转发 | IP, ICMP |
| 2 | 数据链路层 | 物理寻址、错误检测 | 以太网, Wi-Fi |
| 1 | 物理层 | 比特流传输 | 网卡, 光纤 |

### 2. TCP 三次握手

1. 客户端发送 `SYN`，状态 → `SYN_SENT`
2. 服务端返回 `SYN+ACK`，状态 → `SYN_RECV`
3. 客户端发送 `ACK`，双方状态 → `ESTABLISHED`

> **为什么是3次？** 避免历史连接，确认双方收发能力正常

### 3. TCP 四次挥手

1. 客户端发送 `FIN`，请求关闭
2. 服务端返回 `ACK`，确认收到
3. 服务端发送 `FIN`，请求关闭
4. 客户端返回 `ACK`，确认关闭

> **为什么是4次？** 服务端可能还有数据未发送完，需要分开确认

---

## HTTP

### 4. 报文结构

**请求报文**：请求行 + 请求头 + 空行 + 请求体
```
GET /api/user HTTP/1.1
Host: example.com
Content-Type: application/json

{"name": "test"}
```

**响应报文**：状态行 + 响应头 + 空行 + 响应体
```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{"code": 0}
```

### 5. HTTP 常见状态码

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

### [6. 常见 Web 安全及防护](safe.md)

### 7. HTTP 版本区别

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

### 8. Cache-Control

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

### 9. GET 和 POST 的区别

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

### 10. HTTP 和 HTTPS 的区别

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

### 11. TCP 和 UDP 的区别

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

### 12. CORS 跨域

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

### 13. WebSocket

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

### 14. HTTP 请求方法

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

### 15. CDN 原理

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
