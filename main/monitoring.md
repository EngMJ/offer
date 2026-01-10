# 前端监控与可观测性面试题

> 本文档涵盖前端错误监控、性能监控、用户行为分析、日志系统等可观测性面试题

---

## 一、监控体系概述

### 1. 前端监控体系包含哪些方面？

```
┌─────────────────────────────────────────────────────────────────┐
│                       前端监控体系                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  错误监控   │  │  性能监控   │  │  行为监控   │              │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤              │
│  │ JS 运行错误 │  │ 加载性能   │  │ 页面访问   │              │
│  │ 资源加载错误│  │ 运行时性能 │  │ 用户行为   │              │
│  │ Promise 异常│  │ 接口性能   │  │ 功能使用   │              │
│  │ 接口错误   │  │ 资源加载   │  │ 转化漏斗   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐            │
│  │                   数据上报                       │            │
│  │  Beacon API / Fetch / Image                     │            │
│  └─────────────────────────────────────────────────┘            │
│                          │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────┐            │
│  │                   后端服务                       │            │
│  │  数据清洗 → 存储 → 分析 → 告警 → 可视化         │            │
│  └─────────────────────────────────────────────────┘            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

| 监控类型 | 核心指标 | 常用工具 |
|----------|----------|----------|
| 错误监控 | 错误数、错误率、影响用户数 | Sentry, Bugsnag |
| 性能监控 | FCP, LCP, CLS, INP | Lighthouse, Web Vitals |
| 行为监控 | PV, UV, 停留时长, 跳出率 | Google Analytics, 神策 |
| 接口监控 | 成功率、响应时间、错误码 | 自建系统 |

---

## 二、错误监控

### 2. 如何捕获前端各类错误？

```typescript
// 1. JS 运行时错误
window.onerror = function(message, source, lineno, colno, error) {
  reportError({
    type: 'runtime',
    message,
    source,
    lineno,
    colno,
    stack: error?.stack,
  });
  return false; // 不阻止默认错误处理
};

// 2. Promise 未捕获错误
window.addEventListener('unhandledrejection', (event) => {
  reportError({
    type: 'promise',
    message: event.reason?.message || String(event.reason),
    stack: event.reason?.stack,
  });
});

// 3. 资源加载错误
window.addEventListener('error', (event) => {
  const target = event.target as HTMLElement;
  if (target.tagName) {
    reportError({
      type: 'resource',
      tagName: target.tagName,
      src: (target as HTMLImageElement).src || (target as HTMLScriptElement).src,
    });
  }
}, true); // 捕获阶段

// 4. 接口错误 - 拦截 fetch
const originalFetch = window.fetch;
window.fetch = async function(...args) {
  const startTime = Date.now();
  try {
    const response = await originalFetch.apply(this, args);
    const duration = Date.now() - startTime;
    
    if (!response.ok) {
      reportError({
        type: 'api',
        url: args[0],
        status: response.status,
        duration,
      });
    }
    
    return response;
  } catch (error) {
    reportError({
      type: 'api',
      url: args[0],
      message: error.message,
      duration: Date.now() - startTime,
    });
    throw error;
  }
};

// 5. 接口错误 - 拦截 XMLHttpRequest
const originalXHROpen = XMLHttpRequest.prototype.open;
const originalXHRSend = XMLHttpRequest.prototype.send;

XMLHttpRequest.prototype.open = function(method, url) {
  this._url = url;
  this._method = method;
  return originalXHROpen.apply(this, arguments);
};

XMLHttpRequest.prototype.send = function(body) {
  const startTime = Date.now();
  
  this.addEventListener('loadend', () => {
    if (this.status >= 400) {
      reportError({
        type: 'api',
        url: this._url,
        method: this._method,
        status: this.status,
        duration: Date.now() - startTime,
      });
    }
  });
  
  return originalXHRSend.apply(this, arguments);
};
```

---

### 3. React 中如何捕获渲染错误？

```tsx
// ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  state: State = {
    hasError: false,
    error: null,
  };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // 上报错误
    reportError({
      type: 'react',
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
    });
    
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-fallback">
          <h2>页面出错了</h2>
          <button onClick={() => window.location.reload()}>
            刷新页面
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 使用
function App() {
  return (
    <ErrorBoundary
      fallback={<ErrorPage />}
      onError={(error) => console.error('Caught error:', error)}
    >
      <MainContent />
    </ErrorBoundary>
  );
}
```

---

### 4. 如何接入 Sentry 进行错误监控？

```typescript
// sentry.ts
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'https://xxx@sentry.io/xxx',
  environment: import.meta.env.MODE,
  release: __APP_VERSION__, // 版本号，用于关联 sourcemap
  
  // 采样率
  tracesSampleRate: 0.1, // 性能监控采样 10%
  replaysSessionSampleRate: 0.1, // 回放采样 10%
  replaysOnErrorSampleRate: 1.0, // 错误时 100% 录制回放
  
  // 过滤噪音错误
  ignoreErrors: [
    'ResizeObserver loop limit exceeded',
    'Non-Error promise rejection',
    /Loading chunk \d+ failed/,
  ],
  
  // 敏感信息脱敏
  beforeSend(event) {
    // 移除敏感信息
    if (event.request?.cookies) {
      delete event.request.cookies;
    }
    
    // 脱敏处理
    if (event.user) {
      event.user.email = event.user.email?.replace(/(.{2}).*(@.*)/, '$1***$2');
    }
    
    return event;
  },
  
  // 集成
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration(),
  ],
});

// 设置用户上下文
Sentry.setUser({
  id: user.id,
  email: user.email,
  username: user.name,
});

// 添加自定义标签
Sentry.setTag('page', 'checkout');

// 手动上报错误
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { module: 'payment' },
    extra: { orderId: '12345' },
  });
}

// 上报消息
Sentry.captureMessage('用户完成支付', 'info');
```

**Sourcemap 上传配置：**

```typescript
// vite.config.ts
import { sentryVitePlugin } from '@sentry/vite-plugin';

export default defineConfig({
  build: {
    sourcemap: true,
  },
  plugins: [
    sentryVitePlugin({
      org: 'my-org',
      project: 'my-project',
      authToken: process.env.SENTRY_AUTH_TOKEN,
      sourcemaps: {
        assets: './dist/**',
      },
      release: {
        name: process.env.npm_package_version,
      },
    }),
  ],
});
```

---

### 5. 如何自建前端错误监控系统？

```typescript
// monitor/index.ts
interface ErrorInfo {
  type: 'runtime' | 'promise' | 'resource' | 'api' | 'react';
  message: string;
  stack?: string;
  url?: string;
  line?: number;
  column?: number;
  timestamp: number;
  userAgent: string;
  pageUrl: string;
  userId?: string;
}

class ErrorMonitor {
  private queue: ErrorInfo[] = [];
  private timer: number | null = null;
  private readonly BATCH_SIZE = 10;
  private readonly FLUSH_INTERVAL = 5000;
  
  constructor(private endpoint: string) {
    this.init();
  }
  
  private init() {
    // 监听各类错误
    window.onerror = (msg, url, line, col, error) => {
      this.report({
        type: 'runtime',
        message: String(msg),
        url,
        line,
        column: col,
        stack: error?.stack,
      });
    };
    
    window.addEventListener('unhandledrejection', (e) => {
      this.report({
        type: 'promise',
        message: e.reason?.message || String(e.reason),
        stack: e.reason?.stack,
      });
    });
    
    window.addEventListener('error', (e) => {
      const target = e.target as HTMLElement;
      if (target.tagName) {
        this.report({
          type: 'resource',
          message: `${target.tagName} load failed`,
          url: (target as any).src || (target as any).href,
        });
      }
    }, true);
    
    // 页面卸载时发送剩余数据
    window.addEventListener('beforeunload', () => {
      this.flush(true);
    });
  }
  
  report(info: Partial<ErrorInfo>) {
    const errorInfo: ErrorInfo = {
      ...info,
      type: info.type || 'runtime',
      message: info.message || 'Unknown error',
      timestamp: Date.now(),
      userAgent: navigator.userAgent,
      pageUrl: location.href,
      userId: this.getUserId(),
    };
    
    this.queue.push(errorInfo);
    
    if (this.queue.length >= this.BATCH_SIZE) {
      this.flush();
    } else if (!this.timer) {
      this.timer = window.setTimeout(() => this.flush(), this.FLUSH_INTERVAL);
    }
  }
  
  private flush(useBeacon = false) {
    if (this.queue.length === 0) return;
    
    const data = [...this.queue];
    this.queue = [];
    
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
    
    if (useBeacon && navigator.sendBeacon) {
      navigator.sendBeacon(this.endpoint, JSON.stringify(data));
    } else {
      fetch(this.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
        keepalive: true,
      }).catch(() => {
        // 失败时存入 localStorage，下次重试
        const failed = JSON.parse(localStorage.getItem('error_queue') || '[]');
        localStorage.setItem('error_queue', JSON.stringify([...failed, ...data]));
      });
    }
  }
  
  private getUserId(): string | undefined {
    // 从 cookie 或 localStorage 获取用户 ID
    return localStorage.getItem('userId') || undefined;
  }
}

export const errorMonitor = new ErrorMonitor('/api/monitor/errors');
```

---

## 三、性能监控

### 6. Web Vitals 核心指标有哪些？如何采集？

| 指标 | 全称 | 含义 | 良好标准 |
|------|------|------|----------|
| LCP | Largest Contentful Paint | 最大内容绘制时间 | < 2.5s |
| INP | Interaction to Next Paint | 交互到下一帧绘制 | < 200ms |
| CLS | Cumulative Layout Shift | 累计布局偏移 | < 0.1 |
| FCP | First Contentful Paint | 首次内容绘制 | < 1.8s |
| TTFB | Time to First Byte | 首字节时间 | < 800ms |

```typescript
// 使用 web-vitals 库
import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals';

function reportMetric(metric) {
  console.log(metric.name, metric.value);
  
  // 上报到监控平台
  fetch('/api/monitor/vitals', {
    method: 'POST',
    body: JSON.stringify({
      name: metric.name,
      value: metric.value,
      rating: metric.rating, // 'good' | 'needs-improvement' | 'poor'
      delta: metric.delta,
      id: metric.id,
      navigationType: metric.navigationType,
      url: location.href,
      timestamp: Date.now(),
    }),
    keepalive: true,
  });
}

onLCP(reportMetric);
onINP(reportMetric);
onCLS(reportMetric);
onFCP(reportMetric);
onTTFB(reportMetric);
```

**原生 API 采集：**

```typescript
// 使用 Performance API
function collectPerformanceMetrics() {
  const timing = performance.timing;
  const paint = performance.getEntriesByType('paint');
  
  return {
    // 页面加载时间
    pageLoad: timing.loadEventEnd - timing.navigationStart,
    
    // DNS 查询时间
    dns: timing.domainLookupEnd - timing.domainLookupStart,
    
    // TCP 连接时间
    tcp: timing.connectEnd - timing.connectStart,
    
    // 请求响应时间
    request: timing.responseEnd - timing.requestStart,
    
    // DOM 解析时间
    domParse: timing.domComplete - timing.domInteractive,
    
    // 白屏时间
    whiteScreen: timing.domLoading - timing.navigationStart,
    
    // FP (First Paint)
    fp: paint.find(e => e.name === 'first-paint')?.startTime,
    
    // FCP (First Contentful Paint)
    fcp: paint.find(e => e.name === 'first-contentful-paint')?.startTime,
  };
}

// 使用 PerformanceObserver 监听 LCP
const lcpObserver = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log('LCP:', lastEntry.startTime);
});

lcpObserver.observe({ type: 'largest-contentful-paint', buffered: true });
```

---

### 7. 如何监控长任务（Long Tasks）？

```typescript
// 监控长任务（> 50ms）
const longTaskObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      reportPerformance({
        type: 'long-task',
        duration: entry.duration,
        startTime: entry.startTime,
        name: entry.name,
        attribution: entry.attribution,
      });
      
      // 如果任务超过 100ms，标记为严重
      if (entry.duration > 100) {
        console.warn('Critical long task detected:', entry);
      }
    }
  }
});

longTaskObserver.observe({ type: 'longtask', buffered: true });

// 监控帧率
let lastTime = performance.now();
let frameCount = 0;
let fps = 60;

function measureFPS() {
  frameCount++;
  const currentTime = performance.now();
  
  if (currentTime - lastTime >= 1000) {
    fps = Math.round((frameCount * 1000) / (currentTime - lastTime));
    frameCount = 0;
    lastTime = currentTime;
    
    // FPS 低于 30 时上报
    if (fps < 30) {
      reportPerformance({
        type: 'low-fps',
        fps,
        url: location.href,
      });
    }
  }
  
  requestAnimationFrame(measureFPS);
}

requestAnimationFrame(measureFPS);
```

---

### 8. 如何监控资源加载性能？

```typescript
// 监控资源加载
const resourceObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    const resource = entry as PerformanceResourceTiming;
    
    // 只关注关键资源
    if (['script', 'link', 'img'].includes(resource.initiatorType)) {
      const metrics = {
        name: resource.name,
        type: resource.initiatorType,
        duration: resource.duration,
        transferSize: resource.transferSize,
        decodedBodySize: resource.decodedBodySize,
        
        // 各阶段耗时
        dns: resource.domainLookupEnd - resource.domainLookupStart,
        tcp: resource.connectEnd - resource.connectStart,
        ssl: resource.secureConnectionStart > 0 
          ? resource.connectEnd - resource.secureConnectionStart 
          : 0,
        ttfb: resource.responseStart - resource.requestStart,
        download: resource.responseEnd - resource.responseStart,
        
        // 是否命中缓存
        cached: resource.transferSize === 0 && resource.decodedBodySize > 0,
      };
      
      // 慢资源上报（> 3s）
      if (resource.duration > 3000) {
        reportPerformance({
          type: 'slow-resource',
          ...metrics,
        });
      }
    }
  }
});

resourceObserver.observe({ type: 'resource', buffered: true });
```

---

## 四、用户行为监控

### 9. 如何设计前端埋点方案？

**埋点类型：**

| 类型 | 描述 | 触发方式 |
|------|------|----------|
| 页面埋点 | PV、UV、停留时长 | 自动 |
| 点击埋点 | 按钮点击、链接点击 | 声明式/自动 |
| 曝光埋点 | 元素进入视口 | 自动 |
| 自定义埋点 | 业务事件 | 手动调用 |

```typescript
// tracker.ts
interface TrackEvent {
  event: string;
  params?: Record<string, any>;
  timestamp?: number;
}

class Tracker {
  private queue: TrackEvent[] = [];
  private userId: string | null = null;
  private sessionId: string;
  
  constructor(private endpoint: string) {
    this.sessionId = this.generateSessionId();
    this.initAutoTrack();
  }
  
  // 设置用户 ID
  setUserId(userId: string) {
    this.userId = userId;
  }
  
  // 手动埋点
  track(event: string, params?: Record<string, any>) {
    this.send({
      event,
      params,
      timestamp: Date.now(),
    });
  }
  
  // PV 埋点
  trackPageView(pageName?: string) {
    this.track('page_view', {
      page: pageName || document.title,
      url: location.href,
      referrer: document.referrer,
    });
  }
  
  // 点击埋点
  trackClick(element: HTMLElement, eventName?: string) {
    const name = eventName || 
      element.dataset.track || 
      element.innerText?.slice(0, 20);
      
    this.track('click', {
      element: element.tagName,
      name,
      xpath: this.getXPath(element),
    });
  }
  
  // 曝光埋点
  trackExposure(element: HTMLElement, eventName: string) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          this.track('exposure', {
            name: eventName,
            xpath: this.getXPath(element),
          });
          observer.unobserve(element);
        }
      });
    }, { threshold: 0.5 });
    
    observer.observe(element);
  }
  
  // 自动埋点初始化
  private initAutoTrack() {
    // 自动 PV
    this.trackPageView();
    
    // 监听路由变化（SPA）
    const originalPushState = history.pushState;
    history.pushState = (...args) => {
      originalPushState.apply(history, args);
      this.trackPageView();
    };
    
    window.addEventListener('popstate', () => {
      this.trackPageView();
    });
    
    // 自动点击埋点
    document.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;
      if (target.dataset.track) {
        this.trackClick(target, target.dataset.track);
      }
    });
    
    // 页面停留时长
    let enterTime = Date.now();
    window.addEventListener('beforeunload', () => {
      this.track('page_leave', {
        duration: Date.now() - enterTime,
        url: location.href,
      });
    });
  }
  
  private send(event: TrackEvent) {
    const data = {
      ...event,
      userId: this.userId,
      sessionId: this.sessionId,
      userAgent: navigator.userAgent,
      screenSize: `${screen.width}x${screen.height}`,
    };
    
    // 使用 Beacon API 确保数据发送
    if (navigator.sendBeacon) {
      navigator.sendBeacon(this.endpoint, JSON.stringify(data));
    } else {
      fetch(this.endpoint, {
        method: 'POST',
        body: JSON.stringify(data),
        keepalive: true,
      });
    }
  }
  
  private generateSessionId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  private getXPath(element: HTMLElement): string {
    // 生成元素 XPath
    const paths: string[] = [];
    let current: HTMLElement | null = element;
    
    while (current && current !== document.body) {
      let index = 1;
      let sibling = current.previousElementSibling;
      
      while (sibling) {
        if (sibling.tagName === current.tagName) index++;
        sibling = sibling.previousElementSibling;
      }
      
      paths.unshift(`${current.tagName.toLowerCase()}[${index}]`);
      current = current.parentElement;
    }
    
    return '//' + paths.join('/');
  }
}

export const tracker = new Tracker('/api/track');
```

**声明式埋点使用：**

```tsx
// 点击埋点
<button data-track="submit_order">提交订单</button>

// 曝光埋点
<div ref={(el) => el && tracker.trackExposure(el, 'banner_1')}>
  广告横幅
</div>

// 手动埋点
const handlePurchase = () => {
  tracker.track('purchase', {
    productId: '123',
    price: 99.9,
    category: 'electronics',
  });
};
```

---

### 10. 如何实现用户行为回放（Session Replay）？

```typescript
// 使用 rrweb 实现行为回放
import { record, pack } from 'rrweb';

let events: any[] = [];

// 开始录制
const stopRecord = record({
  emit(event) {
    events.push(event);
    
    // 每 100 个事件上传一次
    if (events.length >= 100) {
      uploadEvents([...events]);
      events = [];
    }
  },
  // 配置选项
  maskAllInputs: true,        // 遮蔽所有输入框
  maskTextSelector: '.sensitive', // 遮蔽敏感文本
  blockSelector: '.no-record', // 不录制的元素
  sampling: {
    mousemove: false,          // 不记录鼠标移动
    scroll: 150,               // 滚动采样间隔
  },
});

// 页面卸载时上传剩余事件
window.addEventListener('beforeunload', () => {
  if (events.length > 0) {
    uploadEvents(events);
  }
});

function uploadEvents(data: any[]) {
  const compressed = pack(data);
  navigator.sendBeacon('/api/replay', compressed);
}

// 回放
import { Replayer } from 'rrweb';

async function playback(sessionId: string) {
  const events = await fetch(`/api/replay/${sessionId}`).then(r => r.json());
  
  const replayer = new Replayer(events, {
    root: document.getElementById('replay-container'),
    speed: 1,
  });
  
  replayer.play();
}
```

---

## 五、日志系统

### 11. 如何设计前端日志系统？

```typescript
// logger.ts
enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3,
}

interface LogEntry {
  level: LogLevel;
  message: string;
  data?: any;
  timestamp: number;
  url: string;
  userId?: string;
  traceId?: string;
}

class Logger {
  private level: LogLevel = LogLevel.INFO;
  private queue: LogEntry[] = [];
  private traceId: string | null = null;
  
  constructor(private endpoint: string) {
    // 生产环境只记录 WARN 及以上
    if (import.meta.env.PROD) {
      this.level = LogLevel.WARN;
    }
  }
  
  // 设置追踪 ID（用于链路追踪）
  setTraceId(traceId: string) {
    this.traceId = traceId;
  }
  
  debug(message: string, data?: any) {
    this.log(LogLevel.DEBUG, message, data);
  }
  
  info(message: string, data?: any) {
    this.log(LogLevel.INFO, message, data);
  }
  
  warn(message: string, data?: any) {
    this.log(LogLevel.WARN, message, data);
  }
  
  error(message: string, data?: any) {
    this.log(LogLevel.ERROR, message, data);
  }
  
  private log(level: LogLevel, message: string, data?: any) {
    // 低于当前级别的日志不记录
    if (level < this.level) return;
    
    const entry: LogEntry = {
      level,
      message,
      data: this.sanitize(data),
      timestamp: Date.now(),
      url: location.href,
      userId: this.getUserId(),
      traceId: this.traceId || undefined,
    };
    
    // 控制台输出
    const consoleMethods = ['debug', 'info', 'warn', 'error'];
    console[consoleMethods[level]](
      `[${LogLevel[level]}] ${message}`,
      data || ''
    );
    
    // 加入队列
    this.queue.push(entry);
    
    // ERROR 级别立即上报
    if (level === LogLevel.ERROR) {
      this.flush();
    } else if (this.queue.length >= 20) {
      this.flush();
    }
  }
  
  // 敏感信息脱敏
  private sanitize(data: any): any {
    if (!data) return data;
    
    const sensitiveKeys = ['password', 'token', 'secret', 'creditCard'];
    const sanitized = { ...data };
    
    for (const key of sensitiveKeys) {
      if (key in sanitized) {
        sanitized[key] = '***';
      }
    }
    
    return sanitized;
  }
  
  private flush() {
    if (this.queue.length === 0) return;
    
    const logs = [...this.queue];
    this.queue = [];
    
    fetch(this.endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(logs),
      keepalive: true,
    });
  }
  
  private getUserId(): string | undefined {
    return localStorage.getItem('userId') || undefined;
  }
}

export const logger = new Logger('/api/logs');
```

---

## 六、告警与通知

### 12. 如何设计前端监控告警策略？

```typescript
// 告警规则配置
interface AlertRule {
  name: string;
  metric: string;
  condition: 'gt' | 'lt' | 'eq';
  threshold: number;
  window: number;      // 时间窗口（分钟）
  minSamples: number;  // 最小样本数
  severity: 'P0' | 'P1' | 'P2' | 'P3';
  channels: ('dingtalk' | 'slack' | 'email')[];
}

const alertRules: AlertRule[] = [
  {
    name: 'JS 错误率过高',
    metric: 'js_error_rate',
    condition: 'gt',
    threshold: 1,  // > 1%
    window: 5,     // 5 分钟内
    minSamples: 100,
    severity: 'P1',
    channels: ['dingtalk', 'slack'],
  },
  {
    name: 'LCP 过慢',
    metric: 'lcp_p75',
    condition: 'gt',
    threshold: 4000,  // > 4s
    window: 10,
    minSamples: 50,
    severity: 'P2',
    channels: ['slack'],
  },
  {
    name: '接口成功率下降',
    metric: 'api_success_rate',
    condition: 'lt',
    threshold: 99,  // < 99%
    window: 5,
    minSamples: 200,
    severity: 'P0',
    channels: ['dingtalk', 'slack', 'email'],
  },
];

// 告警消息模板
function formatAlert(rule: AlertRule, currentValue: number): string {
  return `
🚨 **${rule.severity} 告警**

**规则名称**: ${rule.name}
**当前值**: ${currentValue}
**阈值**: ${rule.condition === 'gt' ? '>' : '<'} ${rule.threshold}
**时间窗口**: ${rule.window} 分钟
**触发时间**: ${new Date().toLocaleString()}

[查看详情](https://monitor.example.com/alerts)
  `.trim();
}

// 发送钉钉通知
async function sendDingTalkAlert(message: string, webhook: string) {
  await fetch(webhook, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      msgtype: 'markdown',
      markdown: {
        title: '前端监控告警',
        text: message,
      },
    }),
  });
}
```

---

## 七、监控大盘

### 13. 前端监控大盘应包含哪些内容？

```
┌─────────────────────────────────────────────────────────────────┐
│                      前端监控大盘                                │
├─────────────────────────────────────────────────────────────────┤
│  概览                                                            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│  │  PV/UV  │  │ 错误数   │  │ 错误率   │  │  LCP    │            │
│  │ 12.5K   │  │   23    │  │  0.18%  │  │  2.1s   │            │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘            │
├─────────────────────────────────────────────────────────────────┤
│  性能趋势                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │     LCP / FCP / CLS 趋势图                                │   │
│  │  ──────────────────────────────────────────────────      │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  错误分布                                                        │
│  ┌────────────────────┐  ┌────────────────────────────────┐   │
│  │ 错误类型饼图        │  │ 错误趋势图                      │   │
│  │   JS: 45%          │  │  ─────────────────────────      │   │
│  │   API: 35%         │  │                                  │   │
│  │   Resource: 20%    │  │                                  │   │
│  └────────────────────┘  └────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  Top 错误列表                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 1. TypeError: Cannot read property 'x' of undefined    │   │
│  │    发生 156 次 | 影响用户 89 人 | 最近 10 分钟           │   │
│  │ 2. ChunkLoadError: Loading chunk 5 failed              │   │
│  │    发生 43 次 | 影响用户 41 人 | 最近 30 分钟            │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  慢接口 Top 10                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 接口                          | P95      | 成功率        │   │
│  │ /api/search                  | 2.3s     | 98.5%         │   │
│  │ /api/product/detail          | 1.8s     | 99.1%         │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 八、异常排查

### 14. 收到线上告警后如何排查？

**排查流程：**

```
1. 确认告警
   ├── 告警级别（P0-P3）
   ├── 影响范围（用户数、页面）
   └── 发生时间点

2. 快速定位
   ├── 查看错误堆栈
   ├── 分析 Sourcemap 还原源码
   ├── 查看用户行为回放
   └── 检查最近发布

3. 影响评估
   ├── 影响用户数
   ├── 错误频率
   └── 业务影响

4. 临时处理
   ├── 回滚发布
   ├── 开启降级开关
   └── 扩容/限流

5. 根因分析
   ├── 代码问题
   ├── 数据问题
   ├── 环境问题
   └── 第三方依赖

6. 修复验证
   ├── 修复代码
   ├── 测试验证
   └── 灰度发布

7. 复盘总结
   ├── 时间线梳理
   ├── 根因分析
   ├── 改进措施
   └── 经验沉淀
```

**常见问题排查：**

```typescript
// 1. ChunkLoadError - 资源加载失败
// 原因：CDN 缓存问题、网络问题、发布覆盖
// 解决：重试机制 + 版本化资源

// 2. 白屏问题
// 检查点：
// - JS 是否报错
// - 接口是否返回
// - 路由是否匹配
// - 兼容性问题

// 3. 性能问题
// 检查点：
// - 长任务（Long Tasks）
// - 大 Bundle
// - 慢接口
// - 内存泄漏
```

---

### 15. 如何做前端性能基线和持续监控？

```typescript
// 性能基线配置
const performanceBaseline = {
  // 核心 Web Vitals
  LCP: { p75: 2500, p95: 4000 },
  INP: { p75: 200, p95: 500 },
  CLS: { p75: 0.1, p95: 0.25 },
  
  // 加载性能
  FCP: { p75: 1800, p95: 3000 },
  TTFB: { p75: 800, p95: 1800 },
  
  // 资源
  bundleSize: { main: 200 * 1024, vendor: 500 * 1024 },
  
  // 接口
  apiLatency: { p75: 500, p95: 2000 },
};

// CI 中的性能检测
// lighthouse-ci.config.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000/', 'http://localhost:3000/products'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        'first-contentful-paint': ['warn', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 4000 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['warn', { maxNumericValue: 300 }],
      },
    },
    upload: {
      target: 'lhci',
      serverBaseUrl: 'https://lhci.example.com',
    },
  },
};
```

---

