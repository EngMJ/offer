# 前端系统设计与架构面试题

> 本文档涵盖大型前端应用架构设计、微前端、Monorepo、组件库设计等高级架构面试题

---

## 一、架构设计原则

### 1. 大型前端应用架构设计应遵循哪些原则？

**核心原则：**

| 原则 | 描述 | 实践 |
|------|------|------|
| 单一职责 | 每个模块只负责一件事 | 按功能拆分组件和模块 |
| 开闭原则 | 对扩展开放，对修改关闭 | 使用插件机制、配置化 |
| 依赖倒置 | 依赖抽象而非具体实现 | 使用接口、依赖注入 |
| 关注点分离 | UI、业务逻辑、数据分离 | 分层架构 |
| 高内聚低耦合 | 模块内部紧密，模块间松散 | 明确模块边界 |

**架构分层示例：**

```
┌─────────────────────────────────────────────────┐
│                   View Layer                     │
│           (React/Vue Components)                 │
├─────────────────────────────────────────────────┤
│                  State Layer                     │
│         (Redux/Pinia/Zustand)                   │
├─────────────────────────────────────────────────┤
│                Service Layer                     │
│           (API 调用、业务逻辑)                    │
├─────────────────────────────────────────────────┤
│                  Data Layer                      │
│        (HTTP Client、缓存、持久化)               │
└─────────────────────────────────────────────────┘
```

---

### 2. 如何设计前端分层架构？

```typescript
// 1. Data Layer - 数据访问层
// src/data/http.ts
class HttpClient {
  private baseURL: string;
  
  async request<T>(config: RequestConfig): Promise<T> {
    // 统一处理请求、响应、错误
  }
}

// 2. Service Layer - 业务服务层
// src/services/userService.ts
class UserService {
  constructor(private http: HttpClient) {}
  
  async getUser(id: string): Promise<User> {
    return this.http.get(`/users/${id}`);
  }
  
  async updateProfile(data: ProfileData): Promise<void> {
    // 业务逻辑处理
    const validated = this.validateProfile(data);
    await this.http.put('/users/profile', validated);
  }
}

// 3. State Layer - 状态管理层
// src/stores/userStore.ts
const useUserStore = create((set, get) => ({
  user: null,
  loading: false,
  
  fetchUser: async (id: string) => {
    set({ loading: true });
    const user = await userService.getUser(id);
    set({ user, loading: false });
  },
}));

// 4. View Layer - 视图层
// src/components/UserProfile.tsx
function UserProfile({ userId }: Props) {
  const { user, loading, fetchUser } = useUserStore();
  
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);
  
  if (loading) return <Skeleton />;
  return <ProfileCard user={user} />;
}
```

---

### 3. 什么是领域驱动设计（DDD）？如何在前端应用？

**DDD 核心概念：**

| 概念 | 描述 | 前端应用 |
|------|------|----------|
| 领域 (Domain) | 业务问题空间 | 按业务功能划分模块 |
| 实体 (Entity) | 有唯一标识的对象 | User、Order 等模型 |
| 值对象 (Value Object) | 无标识的不可变对象 | Address、Money |
| 聚合 (Aggregate) | 相关实体的集合 | 订单+订单项 |
| 领域服务 (Domain Service) | 跨实体的业务逻辑 | OrderService |

**前端 DDD 目录结构：**

```
src/
├── domains/                    # 领域层
│   ├── user/
│   │   ├── entities/          # 实体定义
│   │   │   └── User.ts
│   │   ├── services/          # 领域服务
│   │   │   └── UserService.ts
│   │   ├── repositories/      # 数据仓库
│   │   │   └── UserRepository.ts
│   │   └── index.ts
│   ├── order/
│   │   ├── entities/
│   │   │   ├── Order.ts
│   │   │   └── OrderItem.ts
│   │   ├── services/
│   │   │   └── OrderService.ts
│   │   └── index.ts
│   └── shared/                # 共享领域
│       └── valueObjects/
│           ├── Money.ts
│           └── Address.ts
├── application/               # 应用层（用例）
│   ├── useCases/
│   │   ├── CreateOrderUseCase.ts
│   │   └── CheckoutUseCase.ts
│   └── dto/
├── infrastructure/            # 基础设施层
│   ├── http/
│   ├── storage/
│   └── analytics/
└── presentation/              # 展示层
    ├── components/
    ├── pages/
    └── hooks/
```

---

## 二、微前端架构

### 4. 什么是微前端？有哪些实现方案？

**微前端定义：** 将前端应用分解成更小、更简单的独立应用，可以独立开发、测试和部署。

**主流方案对比：**

| 方案 | 原理 | 优点 | 缺点 |
|------|------|------|------|
| qiankun | 基于 single-spa，JS 沙箱 | 成熟稳定、文档完善 | 接入成本中等 |
| Module Federation | Webpack 5 原生支持 | 共享依赖、运行时加载 | 强依赖 Webpack |
| single-spa | 路由劫持、应用注册 | 灵活、框架无关 | 需要自己实现沙箱 |
| iframe | 天然隔离 | 隔离性最强 | 性能差、通信复杂 |
| Web Components | 浏览器原生支持 | 标准化、隔离性好 | 兼容性、生态 |

---

### 5. qiankun 微前端如何实现？核心原理是什么？

**主应用配置：**

```typescript
// main-app/src/main.ts
import { registerMicroApps, start, setDefaultMountApp } from 'qiankun';

// 注册子应用
registerMicroApps([
  {
    name: 'user-app',
    entry: '//localhost:3001',
    container: '#subapp-container',
    activeRule: '/user',
    props: {
      globalState: store,
      utils: sharedUtils,
    },
  },
  {
    name: 'order-app',
    entry: '//localhost:3002',
    container: '#subapp-container',
    activeRule: '/order',
  },
]);

// 启动 qiankun
start({
  sandbox: {
    strictStyleIsolation: true, // Shadow DOM 样式隔离
    experimentalStyleIsolation: true, // scoped CSS
  },
  prefetch: 'all', // 预加载所有子应用
});

setDefaultMountApp('/user');
```

**子应用配置：**

```typescript
// sub-app/src/main.ts
import { createApp, App } from 'vue';
import Root from './App.vue';

let app: App | null = null;

// 渲染函数
function render(props: any = {}) {
  const { container } = props;
  app = createApp(Root);
  app.mount(container ? container.querySelector('#app') : '#app');
}

// 独立运行
if (!window.__POWERED_BY_QIANKUN__) {
  render();
}

// qiankun 生命周期钩子
export async function bootstrap() {
  console.log('子应用 bootstrap');
}

export async function mount(props: any) {
  console.log('子应用 mount', props);
  render(props);
}

export async function unmount() {
  console.log('子应用 unmount');
  app?.unmount();
  app = null;
}
```

**核心原理：**

1. **JS 沙箱**：Proxy 代理 window 对象，隔离全局变量
2. **样式隔离**：Shadow DOM 或 scoped CSS
3. **HTML Entry**：解析子应用 HTML，提取 JS/CSS
4. **路由劫持**：监听路由变化，动态加载/卸载子应用

```typescript
// qiankun JS 沙箱简化实现
class ProxySandbox {
  private proxy: Window;
  private running = false;
  
  constructor() {
    const fakeWindow = Object.create(null);
    
    this.proxy = new Proxy(fakeWindow, {
      get: (target, prop) => {
        // 优先从 fakeWindow 取，否则从真实 window 取
        return prop in target ? target[prop] : window[prop];
      },
      set: (target, prop, value) => {
        if (this.running) {
          target[prop] = value;
        }
        return true;
      },
    });
  }
  
  active() {
    this.running = true;
  }
  
  inactive() {
    this.running = false;
  }
}
```

---

### 6. Module Federation 如何实现微前端？

**Host 应用配置：**

```javascript
// host-app/vite.config.ts
import { defineConfig } from 'vite';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    federation({
      name: 'host-app',
      remotes: {
        remoteApp: 'http://localhost:3001/assets/remoteEntry.js',
      },
      shared: ['vue', 'pinia'], // 共享依赖
    }),
  ],
});

// host-app/src/App.vue
<script setup>
import { defineAsyncComponent } from 'vue';

// 动态加载远程组件
const RemoteButton = defineAsyncComponent(() => import('remoteApp/Button'));
const RemotePage = defineAsyncComponent(() => import('remoteApp/UserPage'));
</script>

<template>
  <div>
    <RemoteButton @click="handleClick">远程按钮</RemoteButton>
    <Suspense>
      <RemotePage />
      <template #fallback>加载中...</template>
    </Suspense>
  </div>
</template>
```

**Remote 应用配置：**

```javascript
// remote-app/vite.config.ts
import { defineConfig } from 'vite';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    federation({
      name: 'remote-app',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button.vue',
        './UserPage': './src/pages/UserPage.vue',
      },
      shared: ['vue', 'pinia'],
    }),
  ],
  build: {
    target: 'esnext',
    minify: false,
    cssCodeSplit: false,
  },
});
```

---

### 7. 微前端如何实现应用间通信？

```typescript
// 1. 全局状态管理（qiankun initGlobalState）
import { initGlobalState, MicroAppStateActions } from 'qiankun';

// 主应用
const actions: MicroAppStateActions = initGlobalState({
  user: null,
  token: '',
});

actions.onGlobalStateChange((state, prev) => {
  console.log('全局状态变化', state, prev);
});

actions.setGlobalState({ user: { name: 'John' } });

// 子应用
export async function mount(props) {
  props.onGlobalStateChange((state, prev) => {
    console.log('子应用收到状态变化', state);
  });
  props.setGlobalState({ token: 'xxx' });
}

// 2. 自定义事件（CustomEvent）
// 发送
window.dispatchEvent(new CustomEvent('micro-app:message', {
  detail: { type: 'USER_LOGIN', payload: { userId: '123' } },
}));

// 接收
window.addEventListener('micro-app:message', (e: CustomEvent) => {
  const { type, payload } = e.detail;
  if (type === 'USER_LOGIN') {
    console.log('用户登录', payload);
  }
});

// 3. 发布订阅模式
class EventBus {
  private events = new Map<string, Set<Function>>();
  
  on(event: string, callback: Function) {
    if (!this.events.has(event)) {
      this.events.set(event, new Set());
    }
    this.events.get(event)!.add(callback);
  }
  
  off(event: string, callback: Function) {
    this.events.get(event)?.delete(callback);
  }
  
  emit(event: string, ...args: any[]) {
    this.events.get(event)?.forEach(cb => cb(...args));
  }
}

// 挂载到 window
window.__MICRO_APP_EVENT_BUS__ = new EventBus();

// 4. URL 参数传递
// 主应用
router.push({ path: '/order', query: { userId: '123' } });

// 子应用
const userId = new URLSearchParams(location.search).get('userId');
```

---

## 三、Monorepo 管理

### 8. 什么是 Monorepo？有什么优缺点？

**Monorepo** 是将多个项目放在同一个代码仓库中管理的策略。

| 优点 | 缺点 |
|------|------|
| 代码共享方便 | 仓库体积大 |
| 统一构建和测试 | CI/CD 复杂 |
| 原子提交（跨包改动一次提交） | 权限管理粗粒度 |
| 依赖版本统一 | 构建时间长 |
| 重构方便 | 新人上手成本高 |

**主流工具对比：**

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| pnpm workspace | 轻量、快速 | 中小型项目 |
| Turborepo | 增量构建、缓存 | 中大型项目 |
| Nx | 功能全面、可视化 | 企业级项目 |
| Lerna | 版本管理、发布 | npm 包管理 |
| Rush | 微软出品、严格 | 超大型项目 |

---

### 9. 如何使用 pnpm workspace 搭建 Monorepo？

**目录结构：**

```
my-monorepo/
├── package.json
├── pnpm-workspace.yaml
├── packages/
│   ├── shared/              # 共享工具库
│   │   ├── package.json
│   │   └── src/
│   ├── ui/                  # UI 组件库
│   │   ├── package.json
│   │   └── src/
│   └── hooks/               # React Hooks
│       ├── package.json
│       └── src/
└── apps/
    ├── web/                 # Web 应用
    │   ├── package.json
    │   └── src/
    └── admin/               # 管理后台
        ├── package.json
        └── src/
```

**配置文件：**

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

```json
// 根目录 package.json
{
  "name": "my-monorepo",
  "private": true,
  "scripts": {
    "dev": "pnpm -r --parallel run dev",
    "build": "pnpm -r run build",
    "test": "pnpm -r run test",
    "lint": "pnpm -r run lint"
  },
  "devDependencies": {
    "typescript": "^5.9.0",
    "prettier": "^3.0.0",
    "eslint": "^8.0.0"
  }
}
```

```json
// packages/shared/package.json
{
  "name": "@myorg/shared",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js"
    }
  }
}
```

```json
// apps/web/package.json
{
  "name": "@myorg/web",
  "version": "1.0.0",
  "dependencies": {
    "@myorg/shared": "workspace:*",
    "@myorg/ui": "workspace:*"
  }
}
```

**常用命令：**

```bash
# 安装所有依赖
pnpm install

# 给特定包添加依赖
pnpm add lodash --filter @myorg/shared

# 给所有包添加开发依赖
pnpm add -Dw typescript

# 运行特定包的脚本
pnpm --filter @myorg/web dev

# 运行所有包的脚本（并行）
pnpm -r --parallel run build
```

---

### 10. Turborepo 如何实现增量构建和缓存？

**配置文件：**

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"],
      "cache": true
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "cache": true
    },
    "lint": {
      "outputs": [],
      "cache": true
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

**核心特性：**

```bash
# 1. 增量构建 - 只构建有变化的包
turbo run build

# 2. 远程缓存 - 团队共享构建缓存
turbo login
turbo link

# 3. 并行执行 - 自动分析依赖图并行执行
turbo run build test lint

# 4. 过滤执行 - 只执行特定包
turbo run build --filter=@myorg/web
turbo run build --filter=@myorg/web...  # 包含依赖
turbo run build --filter=...@myorg/web  # 包含被依赖
```

**缓存原理：**

```
任务输入 Hash = hash(
  源文件内容 +
  依赖包版本 +
  环境变量 +
  turbo.json 配置
)

如果 Hash 相同 → 直接使用缓存
如果 Hash 不同 → 重新执行任务
```

---

## 四、组件库设计

### 11. 如何设计企业级组件库？

**架构设计：**

```
component-library/
├── packages/
│   ├── components/          # 基础组件
│   │   ├── button/
│   │   ├── input/
│   │   └── ...
│   ├── hooks/               # 通用 Hooks
│   ├── utils/               # 工具函数
│   ├── theme/               # 主题系统
│   │   ├── tokens/          # Design Tokens
│   │   ├── presets/         # 预设主题
│   │   └── createTheme.ts
│   └── icons/               # 图标库
├── docs/                    # 文档站点
├── playground/              # 组件试验场
└── scripts/                 # 构建脚本
```

**组件设计原则：**

```typescript
// 1. 组件 API 设计 - 简单直观、符合直觉
interface ButtonProps {
  // 类型 - 使用联合类型限制
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  // 尺寸 - 预设尺寸
  size?: 'sm' | 'md' | 'lg';
  // 状态
  loading?: boolean;
  disabled?: boolean;
  // 事件 - 标准命名
  onClick?: (e: MouseEvent) => void;
  // 子元素
  children: ReactNode;
  // 样式扩展
  className?: string;
  style?: CSSProperties;
}

// 2. 组件实现 - 支持 ref 转发
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', loading, disabled, children, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size }))}
        disabled={disabled || loading}
        {...props}
      >
        {loading && <Spinner className="mr-2" />}
        {children}
      </button>
    );
  }
);

// 3. 复合组件模式 - 灵活组合
<Select>
  <Select.Trigger>
    <Select.Value placeholder="请选择" />
  </Select.Trigger>
  <Select.Content>
    <Select.Item value="1">选项一</Select.Item>
    <Select.Item value="2">选项二</Select.Item>
  </Select.Content>
</Select>
```

---

### 12. Design Token 是什么？如何设计主题系统？

**Design Token** 是设计系统的原子化变量，包括颜色、字体、间距等。

```typescript
// tokens/colors.ts
export const colors = {
  // 基础色板
  gray: {
    50: '#f9fafb',
    100: '#f3f4f6',
    // ...
    900: '#111827',
  },
  primary: {
    50: '#eff6ff',
    100: '#dbeafe',
    // ...
    600: '#2563eb',
    700: '#1d4ed8',
  },
  // 语义化颜色
  semantic: {
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
    info: '#3b82f6',
  },
};

// tokens/spacing.ts
export const spacing = {
  0: '0',
  1: '0.25rem',  // 4px
  2: '0.5rem',   // 8px
  3: '0.75rem',  // 12px
  4: '1rem',     // 16px
  // ...
};

// tokens/typography.ts
export const typography = {
  fontFamily: {
    sans: 'Inter, system-ui, sans-serif',
    mono: 'JetBrains Mono, monospace',
  },
  fontSize: {
    xs: ['0.75rem', { lineHeight: '1rem' }],
    sm: ['0.875rem', { lineHeight: '1.25rem' }],
    base: ['1rem', { lineHeight: '1.5rem' }],
    // ...
  },
};
```

**主题系统设计：**

```typescript
// theme/createTheme.ts
interface ThemeConfig {
  colors: {
    background: string;
    foreground: string;
    primary: string;
    secondary: string;
    // ...
  };
  radius: string;
  // ...
}

function createTheme(config: ThemeConfig): string {
  return `
    :root {
      --background: ${config.colors.background};
      --foreground: ${config.colors.foreground};
      --primary: ${config.colors.primary};
      --radius: ${config.radius};
    }
  `;
}

// 预设主题
export const lightTheme = createTheme({
  colors: {
    background: '0 0% 100%',
    foreground: '222.2 84% 4.9%',
    primary: '221.2 83.2% 53.3%',
    // ...
  },
  radius: '0.5rem',
});

export const darkTheme = createTheme({
  colors: {
    background: '222.2 84% 4.9%',
    foreground: '210 40% 98%',
    primary: '217.2 91.2% 59.8%',
    // ...
  },
  radius: '0.5rem',
});
```

**CSS 变量使用：**

```css
/* 组件中使用 CSS 变量 */
.button {
  background-color: hsl(var(--primary));
  color: hsl(var(--primary-foreground));
  border-radius: var(--radius);
}

.button:hover {
  background-color: hsl(var(--primary) / 0.9);
}
```

---

### 13. 组件库如何做到按需加载？

**1. Tree Shaking（ESM）：**

```json
// package.json
{
  "sideEffects": ["*.css"],
  "module": "./dist/es/index.js",
  "exports": {
    ".": {
      "import": "./dist/es/index.js",
      "require": "./dist/cjs/index.js"
    },
    "./button": {
      "import": "./dist/es/button/index.js",
      "require": "./dist/cjs/button/index.js"
    }
  }
}
```

```typescript
// 用户使用 - 只打包 Button
import { Button } from 'my-ui';
// 或
import Button from 'my-ui/button';
```

**2. 构建时拆分：**

```typescript
// rollup.config.js / vite.config.ts
export default {
  build: {
    lib: {
      entry: {
        index: './src/index.ts',
        button: './src/button/index.ts',
        input: './src/input/index.ts',
        // ...
      },
      formats: ['es', 'cjs'],
    },
    rollupOptions: {
      output: {
        preserveModules: true,
        preserveModulesRoot: 'src',
      },
    },
  },
};
```

**3. CSS 按需加载：**

```typescript
// 使用 unplugin-vue-components 或 babel-plugin-import
// babel.config.js
module.exports = {
  plugins: [
    ['import', {
      libraryName: 'my-ui',
      libraryDirectory: 'es',
      style: (name) => `${name}/style/css`,
    }],
  ],
};
```

---

## 五、状态管理

### 14. 如何选择状态管理方案？

| 方案 | 特点 | 适用场景 |
|------|------|----------|
| useState/useReducer | 简单、内置 | 组件内部状态 |
| Context | 跨组件共享、轻量 | 主题、用户信息等 |
| Redux Toolkit | 可预测、中间件丰富 | 大型应用、复杂状态 |
| Zustand | 轻量、简洁 API | 中小型应用 |
| Jotai | 原子化、细粒度更新 | 复杂表单、实时数据 |
| Recoil | 异步支持好 | 数据依赖复杂 |
| Pinia | Vue 官方、TS 友好 | Vue 项目 |
| MobX | 响应式、OOP | 熟悉响应式编程者 |

**选择决策树：**

```
需要状态管理吗？
├── 只是组件内部状态 → useState
├── 需要跨组件共享
│   ├── 状态简单且不频繁更新 → Context
│   ├── 状态复杂或更新频繁
│   │   ├── 喜欢简洁 API → Zustand
│   │   ├── 需要中间件/DevTools → Redux Toolkit
│   │   ├── 原子化状态 → Jotai
│   │   └── Vue 项目 → Pinia
```

---

### 15. Zustand 如何设计全局状态？

```typescript
// stores/useAuthStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  
  // Actions
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  updateProfile: (data: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      immer((set, get) => ({
        user: null,
        token: null,
        isAuthenticated: false,
        
        login: async (credentials) => {
          const { user, token } = await authService.login(credentials);
          set((state) => {
            state.user = user;
            state.token = token;
            state.isAuthenticated = true;
          });
        },
        
        logout: () => {
          set((state) => {
            state.user = null;
            state.token = null;
            state.isAuthenticated = false;
          });
        },
        
        updateProfile: (data) => {
          set((state) => {
            if (state.user) {
              Object.assign(state.user, data);
            }
          });
        },
      })),
      {
        name: 'auth-storage',
        partialize: (state) => ({ token: state.token }), // 只持久化 token
      }
    ),
    { name: 'AuthStore' }
  )
);

// 选择器 - 避免不必要的重渲染
export const useUser = () => useAuthStore((state) => state.user);
export const useIsAuthenticated = () => useAuthStore((state) => state.isAuthenticated);
```

---

## 六、代码分割与懒加载

### 16. 前端代码分割有哪些策略？

```typescript
// 1. 路由级别分割
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// 2. 组件级别分割
const HeavyChart = lazy(() => import('./components/HeavyChart'));
const RichTextEditor = lazy(() => import('./components/RichTextEditor'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>显示图表</button>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}

// 3. 预加载策略
const Dashboard = lazy(() => import('./pages/Dashboard'));

// 鼠标悬停时预加载
function NavLink({ to, children }) {
  const handleMouseEnter = () => {
    if (to === '/dashboard') {
      import('./pages/Dashboard'); // 预加载
    }
  };
  
  return (
    <Link to={to} onMouseEnter={handleMouseEnter}>
      {children}
    </Link>
  );
}

// 4. 条件加载
const AdminPanel = lazy(() => 
  import('./components/AdminPanel').then(module => ({
    default: module.AdminPanel,
  }))
);

function App() {
  const { isAdmin } = useAuth();
  
  return (
    <div>
      {isAdmin && (
        <Suspense fallback={<Spinner />}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  );
}
```

---

### 17. Vite/Webpack 如何配置代码分割？

**Vite 配置：**

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          // node_modules 分包
          if (id.includes('node_modules')) {
            // React 相关
            if (id.includes('react')) {
              return 'react-vendor';
            }
            // UI 库
            if (id.includes('@radix-ui') || id.includes('class-variance-authority')) {
              return 'ui-vendor';
            }
            // 图表库
            if (id.includes('echarts') || id.includes('chart.js')) {
              return 'chart-vendor';
            }
            // 其他第三方库
            return 'vendor';
          }
        },
      },
    },
    chunkSizeWarningLimit: 500, // chunk 大小警告阈值
  },
});
```

**Webpack 配置：**

```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // React 相关
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router)[\\/]/,
          name: 'react-vendor',
          priority: 20,
        },
        // UI 组件库
        ui: {
          test: /[\\/]node_modules[\\/](@radix-ui|@headlessui)[\\/]/,
          name: 'ui-vendor',
          priority: 15,
        },
        // 其他 vendor
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          priority: 10,
        },
        // 公共代码
        common: {
          minChunks: 2,
          name: 'common',
          priority: 5,
        },
      },
    },
    runtimeChunk: 'single', // 提取 runtime
  },
};
```

---

## 七、国际化架构

### 18. 如何设计前端国际化方案？

```typescript
// 1. 目录结构
// locales/
// ├── zh-CN/
// │   ├── common.json
// │   ├── home.json
// │   └── user.json
// ├── en-US/
// │   ├── common.json
// │   ├── home.json
// │   └── user.json
// └── index.ts

// 2. i18n 配置（react-i18next）
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'zh-CN',
    supportedLngs: ['zh-CN', 'en-US', 'ja-JP'],
    ns: ['common', 'home', 'user'],
    defaultNS: 'common',
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    detection: {
      order: ['querystring', 'cookie', 'localStorage', 'navigator'],
      caches: ['localStorage', 'cookie'],
    },
    interpolation: {
      escapeValue: false,
    },
  });

// 3. 使用
function UserProfile() {
  const { t, i18n } = useTranslation('user');
  
  return (
    <div>
      <h1>{t('profile.title')}</h1>
      <p>{t('profile.welcome', { name: user.name })}</p>
      
      {/* 切换语言 */}
      <select 
        value={i18n.language}
        onChange={(e) => i18n.changeLanguage(e.target.value)}
      >
        <option value="zh-CN">中文</option>
        <option value="en-US">English</option>
      </select>
    </div>
  );
}

// 4. 语言包 JSON
// locales/zh-CN/user.json
{
  "profile": {
    "title": "个人资料",
    "welcome": "欢迎回来，{{name}}！",
    "logout": "退出登录"
  }
}

// locales/en-US/user.json
{
  "profile": {
    "title": "Profile",
    "welcome": "Welcome back, {{name}}!",
    "logout": "Logout"
  }
}
```

**国际化注意事项：**

| 方面 | 注意点 |
|------|--------|
| 日期时间 | 使用 Intl.DateTimeFormat 或 dayjs |
| 数字货币 | 使用 Intl.NumberFormat |
| 复数形式 | 英语 1/other，中文无复数 |
| 文本方向 | RTL 语言（阿拉伯语、希伯来语） |
| 字符长度 | 德语、俄语通常更长 |
| 图片文字 | 需要多语言版本图片 |

---

## 八、综合面试题

### 19. 设计一个大型电商前端架构，需要考虑哪些方面？

**架构设计：**

```
┌─────────────────────────────────────────────────────────────┐
│                        CDN Layer                             │
│              (静态资源、图片、JS/CSS)                          │
├─────────────────────────────────────────────────────────────┤
│                        BFF Layer                             │
│           (数据聚合、SSR、API 代理、鉴权)                       │
├─────────────────────────────────────────────────────────────┤
│                     Micro-Frontend                           │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │   首页   │  │  商品详情 │  │  购物车  │  │  订单   │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
├─────────────────────────────────────────────────────────────┤
│                    Shared Layer                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ 组件库   │  │  工具库  │  │  Hooks  │  │  Types  │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────┘
```

**技术选型：**

| 层级 | 技术方案 |
|------|----------|
| 框架 | React 19 / Next.js 16 |
| 微前端 | qiankun / Module Federation |
| 状态管理 | Zustand + React Query |
| 样式 | Tailwind CSS + CSS Modules |
| 构建 | Turborepo + Vite |
| 监控 | Sentry + 自建埋点 |
| CI/CD | GitHub Actions |

**关键技术点：**

1. **性能优化**：SSR/SSG、图片懒加载、虚拟列表
2. **用户体验**：骨架屏、乐观更新、离线支持
3. **安全**：XSS 防护、CSRF Token、敏感数据脱敏
4. **可观测性**：错误监控、性能监控、用户行为分析
5. **灰度发布**：特性开关、A/B 测试
6. **国际化**：多语言、多币种、多时区

---

### 20. 如何评估和选择前端技术栈？

**评估维度：**

| 维度 | 考量因素 |
|------|----------|
| 团队因素 | 团队技术栈、学习曲线、招聘难度 |
| 项目因素 | 项目规模、性能要求、SEO 需求 |
| 技术因素 | 生态成熟度、社区活跃度、维护状态 |
| 业务因素 | 迭代速度、扩展需求、长期规划 |
| 风险因素 | 技术债务、迁移成本、兼容性 |

**决策框架：**

```markdown
## 技术选型评估报告

### 1. 需求分析
- 项目类型：[管理后台/C 端/App]
- 性能要求：[首屏时间/FCP/LCP]
- SEO 要求：[是/否]
- 团队规模：[人数]
- 预期周期：[X 个月]

### 2. 候选方案
| 方案 | A | B | C |
|------|---|---|---|
| 框架 | React | Vue | Next.js |
| 优点 | ... | ... | ... |
| 缺点 | ... | ... | ... |

### 3. 评分（1-5 分）
| 维度 | 权重 | A | B | C |
|------|------|---|---|---|
| 开发效率 | 25% | 4 | 5 | 4 |
| 性能 | 20% | 4 | 4 | 5 |
| 生态 | 20% | 5 | 4 | 4 |
| 学习成本 | 15% | 3 | 4 | 3 |
| 招聘 | 10% | 5 | 4 | 4 |
| 长期维护 | 10% | 5 | 4 | 5 |

### 4. 结论
推荐方案：[X]
理由：...
风险：...
```

---

