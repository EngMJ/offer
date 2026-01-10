# 前端工程化与 DevOps 面试题

> 本文档涵盖 CI/CD、自动化测试、代码质量、发布策略等前端工程化面试题

---

## 一、CI/CD 流程

### 1. 什么是 CI/CD？前端项目如何实现？

**CI (Continuous Integration) 持续集成：**
- 频繁合并代码到主分支
- 自动运行构建和测试
- 快速发现集成问题

**CD (Continuous Delivery/Deployment) 持续交付/部署：**
- 持续交付：代码随时可部署到生产环境
- 持续部署：自动部署到生产环境

**典型 CI/CD 流程：**

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  Push   │ -> │  Lint   │ -> │  Test   │ -> │  Build  │ -> │ Deploy  │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
                   │              │              │              │
                   v              v              v              v
              代码规范检查    单元/集成测试    构建产物      部署到环境
```

---

### 2. GitHub Actions 如何配置前端 CI/CD？

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  # 代码质量检查
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm type-check

  # 单元测试
  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          
      - run: pnpm install --frozen-lockfile
      - run: pnpm test:coverage
      
      # 上传覆盖率报告
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  # 构建
  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      
      # 上传构建产物
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # 部署到预发环境
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
          
      - name: Deploy to Staging
        run: |
          # 部署到预发环境
          rsync -avz dist/ user@staging-server:/var/www/app/
          
  # 部署到生产环境
  deploy-production:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
          
      - name: Deploy to Production
        run: |
          # 部署到生产环境
          aws s3 sync dist/ s3://my-bucket/ --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_DISTRIBUTION_ID }} --paths "/*"
```

---

### 3. 如何配置 GitLab CI/CD？

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

# 缓存 node_modules
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .pnpm-store/

# 基础镜像
default:
  image: node:${NODE_VERSION}
  before_script:
    - corepack enable
    - pnpm install --frozen-lockfile

# 代码检查
lint:
  stage: lint
  script:
    - pnpm lint
    - pnpm type-check
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"

# 单元测试
test:
  stage: test
  script:
    - pnpm test:coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# 构建
build:
  stage: build
  script:
    - pnpm build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

# 部署预发
deploy_staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - rsync -avz dist/ $STAGING_SERVER:/var/www/app/
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

# 部署生产
deploy_production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - aws s3 sync dist/ s3://$S3_BUCKET/ --delete
  environment:
    name: production
    url: https://example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  when: manual  # 手动触发
```

---

## 二、自动化测试

### 4. 前端测试金字塔是什么？各层测试如何配比？

```
              /\
             /  \
            / E2E \        10% - 端到端测试
           /______\
          /        \
         /Integration\     20% - 集成测试
        /______________\
       /                \
      /    Unit Tests    \   70% - 单元测试
     /____________________\
```

| 测试类型 | 占比 | 特点 | 工具 |
|----------|------|------|------|
| 单元测试 | 70% | 快速、隔离、成本低 | Jest, Vitest |
| 集成测试 | 20% | 组件交互、API 集成 | Testing Library |
| E2E 测试 | 10% | 真实用户流程、慢、成本高 | Playwright, Cypress |

---

### 5. 如何使用 Vitest 编写单元测试？

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function divide(a: number, b: number): number {
  if (b === 0) throw new Error('Cannot divide by zero');
  return a / b;
}

// math.test.ts
import { describe, it, expect, vi } from 'vitest';
import { add, divide } from './math';

describe('math utils', () => {
  describe('add', () => {
    it('should add two positive numbers', () => {
      expect(add(1, 2)).toBe(3);
    });
    
    it('should handle negative numbers', () => {
      expect(add(-1, -2)).toBe(-3);
    });
  });
  
  describe('divide', () => {
    it('should divide two numbers', () => {
      expect(divide(6, 2)).toBe(3);
    });
    
    it('should throw error when dividing by zero', () => {
      expect(() => divide(1, 0)).toThrow('Cannot divide by zero');
    });
  });
});

// 测试异步函数
describe('async functions', () => {
  it('should fetch user data', async () => {
    const user = await fetchUser(1);
    expect(user).toHaveProperty('name');
  });
});

// Mock 示例
describe('with mocks', () => {
  it('should call API with correct params', async () => {
    const mockFetch = vi.fn().mockResolvedValue({ data: 'test' });
    vi.stubGlobal('fetch', mockFetch);
    
    await fetchData('/api/test');
    
    expect(mockFetch).toHaveBeenCalledWith('/api/test', expect.any(Object));
  });
});
```

**Vitest 配置：**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'test/'],
    },
    include: ['**/*.{test,spec}.{js,ts,jsx,tsx}'],
  },
});
```

---

### 6. 如何测试 React 组件？

```tsx
// Button.tsx
interface ButtonProps {
  onClick: () => void;
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
}

export function Button({ onClick, disabled, loading, children }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      aria-busy={loading}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
}

// Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('should render children', () => {
    render(<Button onClick={() => {}}>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('should call onClick when clicked', async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('should not call onClick when disabled', async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick} disabled>Click me</Button>);
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).not.toHaveBeenCalled();
  });
  
  it('should show loading state', () => {
    render(<Button onClick={() => {}} loading>Click me</Button>);
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
  });
});

// 测试 Hooks
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
});
```

---

### 7. 如何使用 Playwright 编写 E2E 测试？

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'Mobile Chrome', use: { ...devices['Pixel 5'] } },
  ],
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});

// e2e/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });
  
  test('should login successfully with valid credentials', async ({ page }) => {
    // 填写表单
    await page.getByLabel('邮箱').fill('test@example.com');
    await page.getByLabel('密码').fill('password123');
    await page.getByRole('button', { name: '登录' }).click();
    
    // 验证登录成功
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('欢迎回来')).toBeVisible();
  });
  
  test('should show error with invalid credentials', async ({ page }) => {
    await page.getByLabel('邮箱').fill('wrong@example.com');
    await page.getByLabel('密码').fill('wrongpassword');
    await page.getByRole('button', { name: '登录' }).click();
    
    await expect(page.getByText('邮箱或密码错误')).toBeVisible();
    await expect(page).toHaveURL('/login');
  });
  
  test('should validate required fields', async ({ page }) => {
    await page.getByRole('button', { name: '登录' }).click();
    
    await expect(page.getByText('请输入邮箱')).toBeVisible();
    await expect(page.getByText('请输入密码')).toBeVisible();
  });
});

// e2e/shopping-cart.spec.ts
test.describe('Shopping Cart', () => {
  test('should add item to cart', async ({ page }) => {
    await page.goto('/products');
    
    // 点击商品
    await page.getByRole('link', { name: /iPhone 15/ }).click();
    
    // 添加到购物车
    await page.getByRole('button', { name: '加入购物车' }).click();
    
    // 验证购物车数量
    await expect(page.getByTestId('cart-count')).toHaveText('1');
    
    // 进入购物车
    await page.getByRole('link', { name: '购物车' }).click();
    await expect(page.getByText('iPhone 15')).toBeVisible();
  });
});
```

---

## 三、代码质量

### 8. 如何配置 ESLint + Prettier + TypeScript？

```javascript
// eslint.config.js (ESLint 9 flat config)
import js from '@eslint/js';
import typescript from '@typescript-eslint/eslint-plugin';
import typescriptParser from '@typescript-eslint/parser';
import react from 'eslint-plugin-react';
import reactHooks from 'eslint-plugin-react-hooks';
import prettier from 'eslint-config-prettier';

export default [
  js.configs.recommended,
  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parser: typescriptParser,
      parserOptions: {
        project: './tsconfig.json',
      },
    },
    plugins: {
      '@typescript-eslint': typescript,
      react,
      'react-hooks': reactHooks,
    },
    rules: {
      // TypeScript 规则
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'off',
      '@typescript-eslint/no-explicit-any': 'warn',
      
      // React 规则
      'react/react-in-jsx-scope': 'off',
      'react/prop-types': 'off',
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      
      // 通用规则
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'prefer-const': 'error',
    },
  },
  prettier, // 关闭与 Prettier 冲突的规则
];
```

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

---

### 9. 如何使用 Husky + lint-staged 实现 Git Hooks？

```bash
# 安装依赖
pnpm add -D husky lint-staged

# 初始化 husky
pnpm exec husky init
```

```json
// package.json
{
  "scripts": {
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md,yml,yaml}": [
      "prettier --write"
    ],
    "*.css": [
      "stylelint --fix",
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
pnpm lint-staged
```

```bash
# .husky/commit-msg
pnpm exec commitlint --edit $1
```

**Commitlint 配置：**

```javascript
// commitlint.config.js
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // 新功能
        'fix',      // 修复 bug
        'docs',     // 文档更新
        'style',    // 代码格式（不影响功能）
        'refactor', // 重构
        'perf',     // 性能优化
        'test',     // 测试
        'chore',    // 构建/工具变动
        'revert',   // 回滚
        'ci',       // CI 配置
      ],
    ],
    'subject-case': [0],
  },
};
```

---

### 10. 什么是 Conventional Commits？如何自动生成 Changelog？

**Conventional Commits 格式：**

```
<type>(<scope>): <subject>

<body>

<footer>
```

**示例：**

```
feat(auth): add OAuth2 login support

- Add Google OAuth2 provider
- Add GitHub OAuth2 provider
- Update login page UI

BREAKING CHANGE: removed legacy login API
Closes #123
```

**自动生成 Changelog：**

```bash
# 安装
pnpm add -D standard-version
# 或
pnpm add -D @changesets/cli
```

```json
// package.json
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major"
  }
}
```

```javascript
// .versionrc.js
module.exports = {
  types: [
    { type: 'feat', section: '✨ Features' },
    { type: 'fix', section: '🐛 Bug Fixes' },
    { type: 'perf', section: '⚡ Performance' },
    { type: 'refactor', section: '♻️ Refactoring' },
    { type: 'docs', section: '📚 Documentation' },
    { type: 'test', hidden: true },
    { type: 'chore', hidden: true },
  ],
};
```

---

## 四、发布策略

### 11. 什么是灰度发布？前端如何实现？

**灰度发布** 是将新版本逐步推送给部分用户，降低发布风险。

**实现方式：**

```typescript
// 1. 基于用户 ID 的灰度
function shouldEnableFeature(userId: string, percentage: number): boolean {
  const hash = hashString(userId);
  return (hash % 100) < percentage;
}

// 2. 基于 Cookie 的灰度
function setGrayRelease(version: 'stable' | 'canary') {
  document.cookie = `app_version=${version}; path=/; max-age=86400`;
}

// 3. Nginx 配置灰度
/*
upstream stable {
    server 192.168.1.1:80;
}

upstream canary {
    server 192.168.1.2:80;
}

split_clients "${remote_addr}" $variant {
    10%     canary;
    *       stable;
}

server {
    location / {
        proxy_pass http://$variant;
    }
}
*/

// 4. 特性开关（Feature Flag）
import { useFeatureFlag } from './featureFlags';

function NewFeature() {
  const isEnabled = useFeatureFlag('new-checkout-flow');
  
  if (!isEnabled) {
    return <OldCheckout />;
  }
  
  return <NewCheckout />;
}
```

**Feature Flag 服务实现：**

```typescript
// featureFlags.ts
interface FeatureFlag {
  name: string;
  enabled: boolean;
  percentage?: number;  // 灰度比例
  whitelist?: string[]; // 白名单用户
  rules?: Rule[];       // 自定义规则
}

class FeatureFlagService {
  private flags: Map<string, FeatureFlag> = new Map();
  
  async init() {
    // 从服务端获取配置
    const response = await fetch('/api/feature-flags');
    const flags = await response.json();
    flags.forEach((flag: FeatureFlag) => {
      this.flags.set(flag.name, flag);
    });
  }
  
  isEnabled(flagName: string, context?: { userId?: string }): boolean {
    const flag = this.flags.get(flagName);
    if (!flag) return false;
    if (!flag.enabled) return false;
    
    // 检查白名单
    if (context?.userId && flag.whitelist?.includes(context.userId)) {
      return true;
    }
    
    // 检查灰度比例
    if (flag.percentage !== undefined && context?.userId) {
      const hash = this.hashString(context.userId);
      return (hash % 100) < flag.percentage;
    }
    
    return flag.enabled;
  }
  
  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) - hash) + str.charCodeAt(i);
      hash |= 0;
    }
    return Math.abs(hash);
  }
}

export const featureFlags = new FeatureFlagService();
```

---

### 12. 蓝绿部署和金丝雀部署有什么区别？

| 特性 | 蓝绿部署 | 金丝雀部署 |
|------|----------|------------|
| 流量切换 | 一次性切换全部流量 | 逐步增加新版本流量 |
| 风险 | 中等（可快速回滚） | 低（小范围试错） |
| 资源成本 | 需要两套环境 | 按比例分配资源 |
| 回滚速度 | 极快（切换流量即可） | 较快 |
| 适用场景 | 重大更新、DB 变更 | 功能迭代、A/B 测试 |

**蓝绿部署流程：**

```
1. 当前生产环境（蓝）正在运行
2. 部署新版本到另一环境（绿）
3. 测试验证绿环境
4. 切换负载均衡指向绿环境
5. 蓝环境保留作为回滚备份
```

**金丝雀部署流程：**

```
1. 部署新版本到小部分实例
2. 导入 5% 流量到新版本
3. 监控错误率、性能指标
4. 逐步增加流量：10% → 25% → 50% → 100%
5. 如有问题，随时回滚
```

---

### 13. 前端项目如何做版本管理？

```json
// package.json
{
  "version": "2.1.0"
}
```

**语义化版本（Semantic Versioning）：**

```
MAJOR.MINOR.PATCH

MAJOR: 不兼容的 API 变更
MINOR: 向下兼容的功能新增
PATCH: 向下兼容的问题修复

示例：
1.0.0 → 初始版本
1.0.1 → Bug 修复
1.1.0 → 新增功能
2.0.0 → 破坏性变更
```

**版本号注入：**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import pkg from './package.json';

export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(pkg.version),
    __BUILD_TIME__: JSON.stringify(new Date().toISOString()),
    __GIT_COMMIT__: JSON.stringify(process.env.GIT_COMMIT || 'unknown'),
  },
});

// 使用
console.log(`App Version: ${__APP_VERSION__}`);
console.log(`Build Time: ${__BUILD_TIME__}`);
```

---

## 五、Docker 容器化

### 14. 如何将前端应用容器化？

```dockerfile
# Dockerfile
# 构建阶段
FROM node:20-alpine AS builder

WORKDIR /app

# 安装 pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

# 复制依赖文件
COPY package.json pnpm-lock.yaml ./

# 安装依赖
RUN pnpm install --frozen-lockfile

# 复制源代码
COPY . .

# 构建
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN pnpm build

# 生产阶段
FROM nginx:alpine

# 复制 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost:80/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    gzip_min_length 1000;

    # 静态资源缓存
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # SPA 路由支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 代理
    location /api/ {
        proxy_pass http://backend:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      args:
        VITE_API_URL: http://api.example.com
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    image: my-backend:latest
    ports:
      - "3000:3000"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

---

## 六、环境管理

### 15. 如何管理多环境配置？

```
.env                # 所有环境共享
.env.local          # 本地覆盖（不提交）
.env.development    # 开发环境
.env.staging        # 预发环境
.env.production     # 生产环境
```

```bash
# .env.development
VITE_API_URL=http://localhost:3000
VITE_ENV=development
VITE_DEBUG=true

# .env.staging
VITE_API_URL=https://api.staging.example.com
VITE_ENV=staging
VITE_DEBUG=true

# .env.production
VITE_API_URL=https://api.example.com
VITE_ENV=production
VITE_DEBUG=false
```

**环境变量类型安全：**

```typescript
// env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_ENV: 'development' | 'staging' | 'production';
  readonly VITE_DEBUG: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

// config.ts
export const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  env: import.meta.env.VITE_ENV,
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD,
  debug: import.meta.env.VITE_DEBUG === 'true',
} as const;
```

---

### 16. CI/CD 中如何安全管理密钥？

```yaml
# GitHub Actions - 使用 Secrets
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 sync dist/ s3://${{ secrets.S3_BUCKET }}/

# 使用 Environment Secrets
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # 使用 production 环境的 secrets
    steps:
      - name: Deploy
        run: echo "Deploying with ${{ secrets.API_KEY }}"
```

**最佳实践：**

| 实践 | 说明 |
|------|------|
| 使用 Secrets | 不要硬编码密钥 |
| 最小权限 | 只授予必要的权限 |
| 定期轮换 | 定期更换密钥 |
| 审计日志 | 记录密钥使用情况 |
| 环境隔离 | 不同环境使用不同密钥 |

---

## 七、构建优化

### 17. 如何优化前端构建速度？

```typescript
// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
  // 1. 依赖预构建优化
  optimizeDeps: {
    include: ['lodash-es', 'axios'], // 预构建常用依赖
    exclude: ['@vueuse/core'],       // 排除不需要预构建的
  },
  
  // 2. 构建缓存
  cacheDir: 'node_modules/.vite',
  
  // 3. esbuild 优化
  esbuild: {
    target: 'esnext',
    drop: ['console', 'debugger'], // 生产环境移除
  },
  
  build: {
    // 4. 使用 esbuild 压缩
    minify: 'esbuild',
    
    // 5. 关闭 sourcemap（生产）
    sourcemap: false,
    
    // 6. 分包策略
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash-es', 'dayjs'],
        },
      },
    },
  },
});
```

**Monorepo 构建优化（Turborepo）：**

```json
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"],
      "cache": true  // 启用缓存
    }
  },
  "remoteCache": {
    "enabled": true  // 启用远程缓存
  }
}
```

---

### 18. 如何分析和优化 Bundle 大小？

```bash
# 安装分析工具
pnpm add -D rollup-plugin-visualizer
```

```typescript
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
});
```

**优化策略：**

```typescript
// 1. 动态导入
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// 2. 按需导入
// Bad
import _ from 'lodash';
// Good
import { debounce } from 'lodash-es';

// 3. 替换大型库
// moment.js (67KB) → dayjs (2KB)
// lodash (71KB) → lodash-es + tree shaking

// 4. 外部化依赖（CDN）
export default defineConfig({
  build: {
    rollupOptions: {
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
      },
    },
  },
});

// 5. 压缩配置
export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
});
```

---

## 八、综合面试题

### 19. 设计一个前端项目的完整 CI/CD 流程

```
┌─────────────────────────────────────────────────────────────────┐
│                         开发阶段                                  │
├─────────────────────────────────────────────────────────────────┤
│  本地开发 → pre-commit hooks (lint, format) → commit            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                          CI 阶段                                 │
├─────────────────────────────────────────────────────────────────┤
│  1. Checkout 代码                                                │
│  2. 安装依赖 (pnpm install --frozen-lockfile)                    │
│  3. 代码检查 (ESLint + TypeScript)                               │
│  4. 单元测试 (Vitest + Coverage)                                 │
│  5. 构建 (Vite build)                                           │
│  6. E2E 测试 (Playwright)                                        │
│  7. 上传产物 (artifacts)                                         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                          CD 阶段                                 │
├─────────────────────────────────────────────────────────────────┤
│  develop 分支 → 自动部署到 Staging                               │
│  main 分支 → 手动审批 → 部署到 Production                         │
│                                                                  │
│  部署步骤:                                                        │
│  1. 下载构建产物                                                  │
│  2. 上传到 CDN/S3                                                │
│  3. 更新 Nginx 配置                                               │
│  4. 清除 CDN 缓存                                                 │
│  5. 发送部署通知                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         监控阶段                                  │
├─────────────────────────────────────────────────────────────────┤
│  1. Sentry 错误监控                                              │
│  2. 性能指标采集 (Web Vitals)                                    │
│  3. 用户行为分析                                                  │
│  4. 告警通知 (钉钉/Slack)                                        │
└─────────────────────────────────────────────────────────────────┘
```

---

### 20. 如何建立前端代码质量体系？

**质量门禁（Quality Gate）：**

| 阶段 | 检查项 | 工具 | 阈值 |
|------|--------|------|------|
| 开发时 | 类型检查 | TypeScript | 0 errors |
| 提交前 | 代码规范 | ESLint + Prettier | 0 errors |
| CI | 单元测试覆盖率 | Vitest | > 80% |
| CI | E2E 测试 | Playwright | 100% pass |
| CI | Bundle 大小 | Size Limit | < 500KB |
| CI | Lighthouse 分数 | Lighthouse CI | > 90 |

**配置示例：**

```json
// package.json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.tsx",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "size": "size-limit",
    "quality": "pnpm lint && pnpm type-check && pnpm test:coverage && pnpm size"
  }
}
```

```javascript
// .size-limit.js
module.exports = [
  {
    path: 'dist/assets/*.js',
    limit: '500 KB',
    gzip: true,
  },
  {
    path: 'dist/assets/*.css',
    limit: '50 KB',
    gzip: true,
  },
];
```

---

