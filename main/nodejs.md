# Node.js 与 BFF 面试题

> 本文档涵盖 Node.js 核心概念、常用框架、BFF 设计、GraphQL、缓存策略等面试题

---

## 一、Node.js 核心概念

### 1. Node.js 的 Event Loop 是什么？与浏览器有什么区别？

**Node.js Event Loop 执行顺序：**

```
   ┌───────────────────────────┐
┌─>│           timers          │  执行 setTimeout/setInterval 回调
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  执行延迟到下一循环的 I/O 回调
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  仅系统内部使用
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │  等待新的 I/O 事件
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │  执行 setImmediate 回调
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  执行 close 事件回调
   └───────────────────────────┘
```

**与浏览器的区别：**

| 特性 | Node.js | 浏览器 |
|------|---------|--------|
| 宏任务 | setTimeout, setInterval, setImmediate, I/O | setTimeout, setInterval, requestAnimationFrame |
| 微任务 | Promise, process.nextTick, queueMicrotask | Promise, MutationObserver, queueMicrotask |
| 执行时机 | 每个阶段之间清空微任务队列 | 每个宏任务后清空微任务队列 |
| 特有 API | process.nextTick, setImmediate | requestAnimationFrame |

```javascript
// Node.js 执行顺序示例
console.log('1. 同步代码');

setTimeout(() => console.log('2. setTimeout'), 0);
setImmediate(() => console.log('3. setImmediate'));

Promise.resolve().then(() => console.log('4. Promise'));
process.nextTick(() => console.log('5. nextTick'));

console.log('6. 同步代码结束');

// 输出顺序：
// 1. 同步代码
// 6. 同步代码结束
// 5. nextTick (微任务，优先级最高)
// 4. Promise (微任务)
// 2. setTimeout (宏任务，timers 阶段)
// 3. setImmediate (宏任务，check 阶段)
```

---

### 2. Node.js 中的 Stream 是什么？有哪些类型？

**Stream 类型：**

| 类型 | 描述 | 示例 |
|------|------|------|
| Readable | 可读流 | fs.createReadStream, http.IncomingMessage |
| Writable | 可写流 | fs.createWriteStream, http.ServerResponse |
| Duplex | 双工流（可读可写） | net.Socket, TCP 连接 |
| Transform | 转换流（读写时修改数据） | zlib.createGzip, crypto.createCipher |

```javascript
const fs = require('fs');
const zlib = require('zlib');

// 1. 基本 Readable Stream
const readStream = fs.createReadStream('input.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024, // 64KB 缓冲区
});

readStream.on('data', (chunk) => {
  console.log('Received chunk:', chunk.length);
});

readStream.on('end', () => {
  console.log('Reading complete');
});

// 2. 管道（Pipe）- 最常用的模式
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())  // 压缩
  .pipe(fs.createWriteStream('output.txt.gz'))
  .on('finish', () => console.log('Compression complete'));

// 3. 自定义 Transform Stream
const { Transform } = require('stream');

class UppercaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

fs.createReadStream('input.txt')
  .pipe(new UppercaseTransform())
  .pipe(fs.createWriteStream('output.txt'));

// 4. 异步迭代器（Node.js 10+）
async function processFile() {
  const stream = fs.createReadStream('input.txt', { encoding: 'utf8' });
  
  for await (const chunk of stream) {
    console.log('Chunk:', chunk);
  }
}
```

---

### 3. Node.js 中的 Buffer 是什么？如何使用？

```javascript
// 1. 创建 Buffer
const buf1 = Buffer.alloc(10);           // 10 字节，初始化为 0
const buf2 = Buffer.allocUnsafe(10);     // 10 字节，未初始化（更快但不安全）
const buf3 = Buffer.from('Hello');       // 从字符串创建
const buf4 = Buffer.from([1, 2, 3, 4]);  // 从数组创建

// 2. 读写操作
const buf = Buffer.alloc(256);
const len = buf.write('Hello World');
console.log(`写入 ${len} 字节`);
console.log(buf.toString('utf8', 0, len));  // 'Hello World'

// 3. Buffer 转换
const jsonBuf = Buffer.from(JSON.stringify({ name: 'test' }));
const jsonStr = jsonBuf.toString();
const jsonObj = JSON.parse(jsonStr);

// 4. Base64 编码
const base64 = Buffer.from('Hello').toString('base64');  // 'SGVsbG8='
const decoded = Buffer.from(base64, 'base64').toString(); // 'Hello'

// 5. 合并 Buffer
const buf5 = Buffer.from('Hello ');
const buf6 = Buffer.from('World');
const combined = Buffer.concat([buf5, buf6]);
console.log(combined.toString());  // 'Hello World'

// 6. 比较 Buffer
const bufA = Buffer.from('ABC');
const bufB = Buffer.from('ABD');
console.log(bufA.compare(bufB));  // -1 (A < B)
```

---

### 4. Node.js 如何处理高并发？Cluster 模块如何使用？

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // 监听 worker 退出
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    // 重启 worker
    cluster.fork();
  });

  // 负载均衡统计
  cluster.on('message', (worker, message) => {
    console.log(`Message from worker ${worker.id}:`, message);
  });
} else {
  // Workers 共享 TCP 连接
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Worker ${process.pid} handled request\n`);
    
    // 向 master 发送消息
    process.send({ type: 'request', pid: process.pid });
  }).listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

**使用 PM2 管理集群：**

```bash
# 启动集群模式
pm2 start app.js -i max  # 根据 CPU 核心数启动

# 或使用配置文件
# ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api',
    script: './app.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
    },
  }]
};
```

---

### 5. Node.js 中如何处理未捕获的异常？

```javascript
// 1. 未捕获的同步异常
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  // 记录日志
  logger.error('Uncaught Exception', { error: err.stack });
  // 优雅退出
  process.exit(1);
});

// 2. 未处理的 Promise 拒绝
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  logger.error('Unhandled Rejection', { reason });
  // 在 Node.js 15+ 中，这会导致进程退出
});

// 3. 优雅关闭
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully');
  
  // 停止接收新请求
  server.close(() => {
    console.log('HTTP server closed');
  });
  
  // 关闭数据库连接
  await db.disconnect();
  
  // 等待现有请求完成（设置超时）
  setTimeout(() => {
    console.log('Forcing shutdown');
    process.exit(0);
  }, 10000);
});

// 4. 域（Domain）- 已废弃，使用 async_hooks 替代
const { AsyncLocalStorage } = require('async_hooks');

const asyncLocalStorage = new AsyncLocalStorage();

function runWithContext(callback) {
  const context = { requestId: generateId() };
  asyncLocalStorage.run(context, callback);
}

function getRequestId() {
  return asyncLocalStorage.getStore()?.requestId;
}
```

---

## 二、常用框架对比

### 6. Express、Koa、Fastify、NestJS 有什么区别？

| 特性 | Express | Koa | Fastify | NestJS |
|------|---------|-----|---------|--------|
| 设计理念 | 简单灵活 | 轻量现代 | 高性能 | 企业级架构 |
| 异步处理 | 回调/中间件 | async/await | async/await | async/await |
| 性能 | 中等 | 中等 | 高 | 中等 |
| 内置功能 | 路由、中间件 | 极简 | 验证、序列化 | 依赖注入、模块化 |
| 学习曲线 | 低 | 低 | 中 | 高 |
| TypeScript | 需要配置 | 需要配置 | 良好支持 | 原生支持 |
| 适用场景 | 小中型项目 | API 服务 | 高性能 API | 大型企业应用 |

**Express 示例：**

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000);
```

**Koa 示例：**

```javascript
const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 洋葱模型中间件
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.set('X-Response-Time', `${ms}ms`);
});

router.get('/users/:id', async (ctx) => {
  const user = await User.findById(ctx.params.id);
  ctx.body = user;
});

app.use(router.routes());
app.listen(3000);
```

**Fastify 示例：**

```javascript
const fastify = require('fastify')({ logger: true });

// JSON Schema 验证
const getUserSchema = {
  params: {
    type: 'object',
    properties: {
      id: { type: 'string' }
    }
  },
  response: {
    200: {
      type: 'object',
      properties: {
        id: { type: 'string' },
        name: { type: 'string' }
      }
    }
  }
};

fastify.get('/users/:id', { schema: getUserSchema }, async (request, reply) => {
  const user = await User.findById(request.params.id);
  return user;
});

fastify.listen({ port: 3000 });
```

**NestJS 示例：**

```typescript
// user.controller.ts
import { Controller, Get, Param } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.userService.findById(id);
  }
}

// user.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  async findById(id: string) {
    return { id, name: 'John' };
  }
}
```

---

## 三、BFF 层设计

### 7. 什么是 BFF？为什么需要 BFF？

**BFF (Backend For Frontend)** 是专门为前端服务的后端层。

```
┌─────────────────────────────────────────────────────────────┐
│                        前端应用                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │   Web   │  │  Mobile │  │   小程序 │                     │
│  └────┬────┘  └────┬────┘  └────┬────┘                     │
│       │            │            │                           │
│       ▼            ▼            ▼                           │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │ Web BFF │  │Mobile BFF│  │Mini BFF │  ← BFF 层           │
│  └────┬────┘  └────┬────┘  └────┬────┘                     │
└───────┼────────────┼────────────┼───────────────────────────┘
        │            │            │
        ▼            ▼            ▼
┌─────────────────────────────────────────────────────────────┐
│                       微服务层                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │用户服务  │  │订单服务  │  │商品服务  │  │支付服务  │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────┘
```

**BFF 的职责：**

| 职责 | 描述 |
|------|------|
| 数据聚合 | 合并多个微服务的数据 |
| 数据裁剪 | 只返回前端需要的字段 |
| 数据格式化 | 转换为前端友好的格式 |
| 认证授权 | 统一处理登录、权限 |
| 缓存 | 缓存热点数据 |
| 错误处理 | 统一错误格式 |
| 协议转换 | gRPC/Thrift → HTTP/JSON |

---

### 8. 如何设计 BFF 层的接口聚合？

```typescript
// bff/services/productDetailService.ts
import { productService } from './productService';
import { inventoryService } from './inventoryService';
import { reviewService } from './reviewService';
import { recommendService } from './recommendService';

interface ProductDetailResponse {
  product: Product;
  inventory: InventoryInfo;
  reviews: Review[];
  recommendations: Product[];
}

export async function getProductDetail(productId: string): Promise<ProductDetailResponse> {
  // 并行请求多个服务
  const [product, inventory, reviews, recommendations] = await Promise.all([
    productService.getProduct(productId),
    inventoryService.getStock(productId),
    reviewService.getReviews(productId, { limit: 10 }),
    recommendService.getRecommendations(productId, { limit: 6 }),
  ]);

  // 数据聚合和格式化
  return {
    product: {
      id: product.id,
      name: product.name,
      price: formatPrice(product.price),
      images: product.images.map(img => ({
        url: img.url,
        thumbnail: img.thumbnail,
      })),
    },
    inventory: {
      available: inventory.quantity > 0,
      quantity: inventory.quantity,
      estimatedDelivery: calculateDeliveryDate(inventory),
    },
    reviews: reviews.map(review => ({
      id: review.id,
      rating: review.rating,
      content: review.content,
      author: review.user.nickname,
      date: formatDate(review.createdAt),
    })),
    recommendations: recommendations.map(p => ({
      id: p.id,
      name: p.name,
      price: formatPrice(p.price),
      thumbnail: p.images[0]?.thumbnail,
    })),
  };
}

// 路由处理
router.get('/product/:id', async (ctx) => {
  const { id } = ctx.params;
  
  try {
    const data = await getProductDetail(id);
    ctx.body = { success: true, data };
  } catch (error) {
    ctx.status = 500;
    ctx.body = { success: false, error: error.message };
  }
});
```

---

### 9. BFF 如何处理超时和降级？

```typescript
// utils/resilience.ts
import pTimeout from 'p-timeout';
import pRetry from 'p-retry';

// 1. 超时处理
async function withTimeout<T>(
  promise: Promise<T>,
  ms: number,
  fallback?: T
): Promise<T> {
  try {
    return await pTimeout(promise, { milliseconds: ms });
  } catch (error) {
    if (fallback !== undefined) {
      console.warn(`Request timed out, using fallback`);
      return fallback;
    }
    throw error;
  }
}

// 2. 重试机制
async function withRetry<T>(
  fn: () => Promise<T>,
  options = { retries: 3, minTimeout: 1000 }
): Promise<T> {
  return pRetry(fn, {
    ...options,
    onFailedAttempt: (error) => {
      console.warn(`Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`);
    },
  });
}

// 3. 熔断器
class CircuitBreaker {
  private failures = 0;
  private lastFailure: number | null = null;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  
  constructor(
    private threshold = 5,
    private timeout = 30000
  ) {}

  async execute<T>(fn: () => Promise<T>, fallback?: () => T): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailure! > this.timeout) {
        this.state = 'HALF_OPEN';
      } else if (fallback) {
        return fallback();
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      if (fallback) {
        return fallback();
      }
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    this.lastFailure = Date.now();
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// 使用示例
const productServiceBreaker = new CircuitBreaker(5, 30000);

async function getProduct(id: string) {
  return productServiceBreaker.execute(
    () => withTimeout(
      withRetry(() => productService.getProduct(id)),
      5000
    ),
    () => ({ id, name: '商品信息暂不可用', price: 0 }) // 降级返回
  );
}
```

---

## 四、GraphQL

### 10. GraphQL 与 REST API 有什么区别？

| 特性 | REST API | GraphQL |
|------|----------|---------|
| 数据获取 | 多次请求多个端点 | 单次请求获取所需数据 |
| 数据量 | 可能过多或过少 | 精确获取需要的字段 |
| 版本管理 | URL 版本控制 | 无需版本，字段可废弃 |
| 类型系统 | 无强制类型 | 强类型 Schema |
| 文档 | 需要额外维护 | Schema 即文档 |
| 缓存 | HTTP 缓存 | 需要客户端缓存（Apollo） |
| 学习曲线 | 低 | 中 |

**GraphQL 示例：**

```typescript
// schema.ts
import { gql } from 'apollo-server';

export const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
    comments: [Comment!]!
  }

  type Comment {
    id: ID!
    text: String!
    author: User!
  }

  type Query {
    user(id: ID!): User
    users: [User!]!
    post(id: ID!): Post
    posts(limit: Int, offset: Int): [Post!]!
  }

  type Mutation {
    createPost(title: String!, content: String!): Post!
    updatePost(id: ID!, title: String, content: String): Post
    deletePost(id: ID!): Boolean!
  }

  type Subscription {
    postCreated: Post!
    commentAdded(postId: ID!): Comment!
  }
`;

// resolvers.ts
export const resolvers = {
  Query: {
    user: async (_, { id }, { dataSources }) => {
      return dataSources.userAPI.getUser(id);
    },
    posts: async (_, { limit = 10, offset = 0 }, { dataSources }) => {
      return dataSources.postAPI.getPosts({ limit, offset });
    },
  },
  Mutation: {
    createPost: async (_, { title, content }, { dataSources, user }) => {
      if (!user) throw new Error('Not authenticated');
      return dataSources.postAPI.createPost({ title, content, authorId: user.id });
    },
  },
  User: {
    posts: async (user, _, { dataSources }) => {
      return dataSources.postAPI.getPostsByAuthor(user.id);
    },
  },
  Post: {
    author: async (post, _, { dataSources }) => {
      return dataSources.userAPI.getUser(post.authorId);
    },
  },
};

// 前端查询
const GET_USER_WITH_POSTS = gql`
  query GetUserWithPosts($userId: ID!) {
    user(id: $userId) {
      id
      name
      posts {
        id
        title
        comments {
          id
          text
        }
      }
    }
  }
`;
```

---

### 11. GraphQL 如何解决 N+1 问题？

```typescript
// 使用 DataLoader 解决 N+1 问题
import DataLoader from 'dataloader';

// 创建 DataLoader
const userLoader = new DataLoader(async (userIds: readonly string[]) => {
  // 批量查询
  const users = await User.find({ _id: { $in: userIds } });
  
  // 按 ID 排序返回
  const userMap = new Map(users.map(u => [u.id.toString(), u]));
  return userIds.map(id => userMap.get(id) || null);
});

// Resolver 中使用
const resolvers = {
  Post: {
    author: async (post, _, { loaders }) => {
      // 使用 DataLoader 批量加载
      return loaders.userLoader.load(post.authorId);
    },
  },
};

// 在 context 中创建 loader（每个请求一个新实例）
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: () => ({
    loaders: {
      userLoader: new DataLoader(batchUsers),
      postLoader: new DataLoader(batchPosts),
    },
  }),
});
```

---

## 五、缓存策略

### 12. Node.js 中有哪些缓存策略？

```typescript
// 1. 内存缓存（LRU Cache）
import LRU from 'lru-cache';

const cache = new LRU<string, any>({
  max: 500,           // 最大缓存数量
  ttl: 1000 * 60 * 5, // 5 分钟过期
});

async function getUser(id: string) {
  const cacheKey = `user:${id}`;
  
  // 检查缓存
  let user = cache.get(cacheKey);
  if (user) return user;
  
  // 查询数据库
  user = await db.user.findById(id);
  
  // 写入缓存
  cache.set(cacheKey, user);
  
  return user;
}

// 2. Redis 缓存
import Redis from 'ioredis';

const redis = new Redis({
  host: 'localhost',
  port: 6379,
  keyPrefix: 'app:',
});

async function getUserWithRedis(id: string) {
  const cacheKey = `user:${id}`;
  
  // 检查缓存
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // 查询数据库
  const user = await db.user.findById(id);
  
  // 写入缓存（5 分钟过期）
  await redis.setex(cacheKey, 300, JSON.stringify(user));
  
  return user;
}

// 3. 缓存穿透防护
async function getUserSafe(id: string) {
  const cacheKey = `user:${id}`;
  const cached = await redis.get(cacheKey);
  
  if (cached === 'NULL') {
    return null; // 缓存空值
  }
  if (cached) {
    return JSON.parse(cached);
  }
  
  const user = await db.user.findById(id);
  
  if (!user) {
    // 缓存空值，短期过期
    await redis.setex(cacheKey, 60, 'NULL');
    return null;
  }
  
  await redis.setex(cacheKey, 300, JSON.stringify(user));
  return user;
}

// 4. 缓存雪崩防护 - 随机过期时间
function getRandomTTL(baseTTL: number) {
  const jitter = Math.floor(Math.random() * 60); // 0-60 秒随机
  return baseTTL + jitter;
}
```

---

### 13. 如何实现 HTTP 响应缓存？

```typescript
// Express 中间件
import { Request, Response, NextFunction } from 'express';

// 1. ETag 缓存
import etag from 'etag';

function etagMiddleware(req: Request, res: Response, next: NextFunction) {
  const originalSend = res.send;
  
  res.send = function(body) {
    const etagValue = etag(body);
    res.set('ETag', etagValue);
    
    if (req.headers['if-none-match'] === etagValue) {
      return res.status(304).end();
    }
    
    return originalSend.call(this, body);
  };
  
  next();
}

// 2. Cache-Control 设置
function cacheControl(maxAge: number) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (req.method === 'GET') {
      res.set('Cache-Control', `public, max-age=${maxAge}`);
    } else {
      res.set('Cache-Control', 'no-store');
    }
    next();
  };
}

// 3. 条件请求处理
app.get('/api/products/:id', async (req, res) => {
  const product = await getProduct(req.params.id);
  const lastModified = product.updatedAt.toUTCString();
  
  res.set('Last-Modified', lastModified);
  res.set('Cache-Control', 'private, max-age=0, must-revalidate');
  
  // 检查 If-Modified-Since
  const ifModifiedSince = req.get('If-Modified-Since');
  if (ifModifiedSince && new Date(ifModifiedSince) >= product.updatedAt) {
    return res.status(304).end();
  }
  
  res.json(product);
});
```

---

## 六、认证授权

### 14. JWT 如何实现无状态认证？

```typescript
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_EXPIRES_IN = '7d';
const REFRESH_TOKEN_EXPIRES_IN = '30d';

interface TokenPayload {
  userId: string;
  role: string;
}

// 生成 Token
function generateTokens(payload: TokenPayload) {
  const accessToken = jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRES_IN,
  });
  
  const refreshToken = jwt.sign(payload, JWT_SECRET, {
    expiresIn: REFRESH_TOKEN_EXPIRES_IN,
  });
  
  return { accessToken, refreshToken };
}

// 验证 Token
function verifyToken(token: string): TokenPayload {
  return jwt.verify(token, JWT_SECRET) as TokenPayload;
}

// 认证中间件
function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    const payload = verifyToken(token);
    req.user = payload;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// 权限中间件
function requireRole(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// 路由使用
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await User.findByEmail(email);
  if (!user || !await user.verifyPassword(password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const tokens = generateTokens({ userId: user.id, role: user.role });
  
  // 存储 refresh token（可选，用于撤销）
  await redis.set(`refresh:${user.id}`, tokens.refreshToken, 'EX', 30 * 24 * 60 * 60);
  
  res.json(tokens);
});

app.post('/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  try {
    const payload = verifyToken(refreshToken);
    
    // 验证 refresh token 是否有效
    const stored = await redis.get(`refresh:${payload.userId}`);
    if (stored !== refreshToken) {
      return res.status(401).json({ error: 'Invalid refresh token' });
    }
    
    const tokens = generateTokens({ userId: payload.userId, role: payload.role });
    
    // 更新存储的 refresh token
    await redis.set(`refresh:${payload.userId}`, tokens.refreshToken, 'EX', 30 * 24 * 60 * 60);
    
    res.json(tokens);
  } catch (error) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});

app.get('/admin/users', authMiddleware, requireRole('admin'), async (req, res) => {
  const users = await User.findAll();
  res.json(users);
});
```

---

## 七、数据库操作

### 15. 如何使用 Prisma 进行数据库操作？

```typescript
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  tags      Tag[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
}

// prisma/client.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: ['query', 'error', 'warn'],
});

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

// 使用示例
// 创建用户
const user = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John',
    password: hashedPassword,
  },
});

// 关联查询
const userWithPosts = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 10,
    },
  },
});

// 事务
const [post, user] = await prisma.$transaction([
  prisma.post.create({
    data: { title: 'New Post', authorId: userId },
  }),
  prisma.user.update({
    where: { id: userId },
    data: { postCount: { increment: 1 } },
  }),
]);

// 复杂查询
const posts = await prisma.post.findMany({
  where: {
    AND: [
      { published: true },
      {
        OR: [
          { title: { contains: 'Prisma' } },
          { content: { contains: 'Prisma' } },
        ],
      },
    ],
  },
  select: {
    id: true,
    title: true,
    author: {
      select: { name: true },
    },
  },
  orderBy: { createdAt: 'desc' },
  skip: 0,
  take: 20,
});
```

---

## 八、SSR 实现

### 16. SSR 的原理是什么？如何手动实现？

```typescript
// server.ts
import express from 'express';
import React from 'react';
import { renderToString, renderToPipeableStream } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom/server';
import App from './App';

const app = express();

// 基本 SSR
app.get('*', async (req, res) => {
  // 预取数据
  const initialData = await fetchDataForRoute(req.url);
  
  // 渲染 React 组件为 HTML 字符串
  const html = renderToString(
    <StaticRouter location={req.url}>
      <App initialData={initialData} />
    </StaticRouter>
  );
  
  // 返回完整 HTML
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>SSR App</title>
        <link rel="stylesheet" href="/styles.css">
      </head>
      <body>
        <div id="root">${html}</div>
        <script>
          window.__INITIAL_DATA__ = ${JSON.stringify(initialData)};
        </script>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});

// Streaming SSR (React 18+)
app.get('*', async (req, res) => {
  const { pipe, abort } = renderToPipeableStream(
    <StaticRouter location={req.url}>
      <App />
    </StaticRouter>,
    {
      bootstrapScripts: ['/bundle.js'],
      onShellReady() {
        res.setHeader('content-type', 'text/html');
        pipe(res);
      },
      onShellError(error) {
        res.status(500).send('Error');
      },
      onError(error) {
        console.error(error);
      },
    }
  );
  
  // 超时中止
  setTimeout(abort, 10000);
});

app.listen(3000);
```

**客户端 Hydration：**

```typescript
// client.ts
import { hydrateRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

// 获取服务端注入的数据
const initialData = window.__INITIAL_DATA__;

hydrateRoot(
  document.getElementById('root'),
  <BrowserRouter>
    <App initialData={initialData} />
  </BrowserRouter>
);
```

---

## 九、综合面试题

### 17. 设计一个高可用的 Node.js BFF 服务

```
┌─────────────────────────────────────────────────────────────┐
│                      Load Balancer                          │
│                     (Nginx / ALB)                           │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌───────────┐  ┌───────────┐  ┌───────────┐
│  Node.js  │  │  Node.js  │  │  Node.js  │   PM2 Cluster
│ Instance 1│  │ Instance 2│  │ Instance 3│
└─────┬─────┘  └─────┬─────┘  └─────┬─────┘
      │              │              │
      └──────────────┼──────────────┘
                     │
     ┌───────────────┼───────────────┐
     │               │               │
     ▼               ▼               ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│  Redis  │    │  MySQL  │    │ 微服务   │
│ (缓存)  │    │ (数据库) │    │  API    │
└─────────┘    └─────────┘    └─────────┘
```

**关键设计：**

```typescript
// 1. 健康检查
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    timestamp: Date.now(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
  });
});

// 2. 优雅关闭
let isShuttingDown = false;

process.on('SIGTERM', async () => {
  isShuttingDown = true;
  
  // 停止接收新请求
  server.close(async () => {
    // 关闭数据库连接
    await prisma.$disconnect();
    await redis.quit();
    
    process.exit(0);
  });
  
  // 强制退出超时
  setTimeout(() => process.exit(1), 30000);
});

// 拒绝新请求中间件
app.use((req, res, next) => {
  if (isShuttingDown) {
    res.set('Connection', 'close');
    res.status(503).json({ error: 'Server is shutting down' });
  } else {
    next();
  }
});

// 3. 请求超时
import timeout from 'connect-timeout';

app.use(timeout('30s'));
app.use((req, res, next) => {
  if (!req.timedout) next();
});

// 4. 限流
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 分钟
  max: 100, // 每 IP 100 次请求
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', limiter);

// 5. 链路追踪
import { v4 as uuid } from 'uuid';

app.use((req, res, next) => {
  req.traceId = req.headers['x-trace-id'] || uuid();
  res.set('X-Trace-Id', req.traceId);
  next();
});
```

---

### 18. Node.js 性能优化有哪些方法？

| 方向 | 优化方法 |
|------|----------|
| 代码层面 | 避免同步操作、使用 Stream、合理使用缓存 |
| 架构层面 | Cluster 多进程、负载均衡、微服务拆分 |
| 运行时 | 调整 V8 参数、使用最新 Node.js 版本 |
| 数据库 | 连接池、索引优化、读写分离 |
| 网络 | 开启 Keep-Alive、使用 HTTP/2、CDN |
| 缓存 | Redis 缓存、内存缓存、HTTP 缓存 |

```javascript
// V8 参数优化示例
node --max-old-space-size=4096 app.js  // 增加老生代内存
node --optimize-for-size app.js         // 优化内存占用
```

---

