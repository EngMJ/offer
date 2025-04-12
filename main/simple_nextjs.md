# Next.js 常用内容

1. packge.json script命令解释
next dev：启动开发服务器。
next build：构建用于生产的应用程序。
next start：启动生产服务器。
next lint：运行 ESLint。

2. 文件路由
版本13以前: pages目录下的文件名即为路由路径。
```txt
pages/index.js => /
pages/about.js => /about
pages/posts/[id].js => /posts/:id

```

版本13以后: app目录下的文件名即为路由路径。
```txt
app/page.js => /
app/about/page.js => /about
app/posts/[id]/page.js => /posts/:id

```

3. 项目结构
```text
my-nextjs-app/
├── .env                   // 环境变量文件（可有多个，如 .env.development、.env.production 等）
├── .gitignore             // Git 忽略规则
├── next.config.ts         // Next.js 配置文件,配置打包规则/国际化等
├── next-env.d.ts          // ts声明文件
├── tsconfig.json          // ts配置文件
├── postcss.config.mjs     // postcss 配置文件
├── package.json           // 项目依赖、脚本及元数据
├── tsconfig.json          // TypeScript 配置（或 jsconfig.json，用于 JS 项目）
├── public/                // 静态资源目录（图片、图标、字体、静态 HTML 等）
│   ├── vercel.svg
│   └── images/
│       └── logo.png
├── app/                   // App Router 路由目录（推荐，新项目使用）
│   ├── layout.tsx          // 全局布局文件，所有页面都会继承
│   ├── page.tsx            // 根页面，响应 "/"
│   ├── globals.css        // 全局css
│   ├── favicon.ico        // 网页图标
│   ├── loading.tsx         // 全局加载状态页面（异步数据加载时显示）
│   ├── error.tsx           // 全局错误处理页面
│   ├── not-found.tsx       // 404 页面
│   ├── dashboard/         // 嵌套路由示例
│   │   ├── layout.tsx      // dashboard 专用布局，可嵌套全局布局
│   │   └── page.tsx        // 响应 "/dashboard"
│   ├── blog/              // 另一组路由示例
│   │   ├── [id]/          // 动态路由文件夹，id 为动态参数
│   │   │   └── page.tsx    // 响应 "/blog/xxx"
│   │   └── page.tsx        // 响应 "/blog"
│   └── api/               // App Router 内的 API 路由
│       └── hello/         
│           └── route.tsx   // 响应 "/api/hello" 的 API 端点
├── pages/                 // Pages Router 路由目录（传统方式，可选，与 app/ 共存）
│   ├── _app.js            // 自定义应用组件
│   ├── _document.js       // 自定义 HTML 文档结构
│   ├── index.js           // 首页，对应 "/"
│   ├── about.js           // 关于页面，对应 "/about"
│   └── api/               // API 路由目录（传统方式）
│       └── hello.js       // 响应 "/api/hello"
└── components/            // 公共组件目录，可按业务模块或功能分子目录
    ├── layout/            // 布局组件，如 Header、Footer、Sidebar 等
    │   ├── Header.jsx
    │   └── Footer.jsx
    ├── cards/             // 卡片、展示组件
    │   └── CatCard.jsx
    └── ui/                // 其他常用 UI 组件（按钮、输入框等）
        └── Button.jsx

```







## 1. 基础介绍

**概念：**  
Next.js 是一个基于 React 的框架，提供服务端渲染（SSR）、静态站点生成（SSG）以及增量静态生成（ISR）等功能，能有效提升页面加载性能和 SEO 效果。

**示例：**  
项目初始化通常使用以下命令：
```bash
npx create-next-app my-next-app
cd my-next-app
npm run dev
```

---

## 2. 数据获取方式

### 2.1 `getStaticProps`

**用途：** 在构建时获取数据，生成静态页面，适用于数据变化不频繁的场景。  
**示例代码：**
```jsx
// pages/index.js
export async function getStaticProps() {
  // 模拟从 API 获取数据
  const res = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=5')
  const posts = await res.json()

  return {
    props: { posts }, // 将数据作为 props 传递给页面组件
    // revalidate: 10, // 可选：开启 ISR，每 10 秒重新生成页面
  }
}

export default function Home({ posts }) {
  return (
    <div>
      <h1>最新文章</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  )
}
```

### 2.2 `getServerSideProps`

**用途：** 每次请求时在服务器端获取数据，适合实时数据。  
**示例代码：**
```jsx
// pages/dashboard.js
export async function getServerSideProps(context) {
  // 根据请求上下文获取数据，例如用户认证信息等
  const res = await fetch('https://jsonplaceholder.typicode.com/users/1')
  const user = await res.json()

  return { props: { user } }
}

export default function Dashboard({ user }) {
  return (
    <div>
      <h1>用户信息</h1>
      <p>用户名：{user.username}</p>
    </div>
  )
}
```

### 2.3 `getStaticPaths`

**用途：** 与 `getStaticProps` 搭配使用，用于动态路由下的静态页面生成。  
**示例代码：**
```jsx
// pages/posts/[id].js
export async function getStaticPaths() {
  // 模拟 API 请求获取所有文章 id
  const res = await fetch('https://jsonplaceholder.typicode.com/posts')
  const posts = await res.json()

  const paths = posts.map(post => ({
    params: { id: post.id.toString() },
  }))

  return { paths, fallback: false } // fallback 为 false 表示未预生成的路径返回 404
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts/${params.id}`)
  const post = await res.json()

  return { props: { post } }
}

export default function Post({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </div>
  )
}
```

---

## 3. 路由机制

**概念：**  
Next.js 采用基于文件系统的路由，将 `pages` 文件夹中的每个文件自动映射为对应的 URL 路径。

**示例：**
- `pages/index.js` → `/`
- `pages/about.js` → `/about`
- `pages/posts/[id].js` → `/posts/:id`

**使用 Link 组件实现客户端导航：**
```jsx
// pages/index.js
import Link from 'next/link'

export default function Home() {
  return (
    <div>
      <h1>欢迎来到首页</h1>
      <Link href="/about">
        <a>关于我们</a>
      </Link>
    </div>
  )
}
```

---

## 4. 样式处理

### 4.1 全局样式

**概念：**  
全局样式在 `pages/_app.js` 中导入，影响整个应用。

**示例代码：**
```jsx
// pages/_app.js
import '../styles/globals.css'

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}

export default MyApp
```

### 4.2 CSS 模块

**概念：**  
CSS 模块文件命名为 `.module.css`，用于局部样式，防止全局冲突。

**示例代码：**
```jsx
// components/Button.js
import styles from './Button.module.css'

export default function Button({ children }) {
  return <button className={styles.btn}>{children}</button>
}
```
```css
/* components/Button.module.css */
.btn {
  background-color: #0070f3;
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
}
```

---

## 5. API 路由

**概念：**  
在 `pages/api` 目录下创建文件，每个文件导出一个处理请求的函数，实现后端 API 接口。

**示例代码：**
```jsx
// pages/api/hello.js
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello from Next.js API!' })
}
```
访问 `http://localhost:3000/api/hello` 就可以看到返回的 JSON 数据。

---

## 6. SEO 优化

**概念：**  
利用 SSR/SSG 生成完整 HTML，提高搜索引擎抓取效率，同时通过 `next/head` 设置页面标题和 meta 标签。

**示例代码：**
```jsx
// pages/about.js
import Head from 'next/head'

export default function About() {
  return (
    <div>
      <Head>
        <title>关于我们 - 我的 Next.js 应用</title>
        <meta name="description" content="这是一个关于我们的页面" />
      </Head>
      <h1>关于我们</h1>
      <p>这里是关于我们页面的内容。</p>
    </div>
  )
}
```

---

## 7. 动态导入与代码分割

**概念：**  
Next.js 支持使用动态导入（Dynamic Import）来按需加载组件，减少初始加载包大小。

**示例代码：**
```jsx
// pages/dynamic.js
import dynamic from 'next/dynamic'

const DynamicComponent = dynamic(() => import('../components/HeavyComponent'), {
  loading: () => <p>加载中...</p>,
})

export default function DynamicPage() {
  return (
    <div>
      <h1>动态导入示例</h1>
      <DynamicComponent />
    </div>
  )
}
```
在上述代码中，`HeavyComponent` 只有在需要时才会加载，优化了性能。

---

## 8. 环境变量配置

**概念：**  
在项目根目录下创建环境变量文件（如 `.env.local`），并通过 `process.env` 访问。客户端变量需以 `NEXT_PUBLIC_` 开头。

**示例：**
```bash
# .env.local
NEXT_PUBLIC_API_URL=https://api.example.com
SECRET_API_KEY=your-secret-key
```
在代码中使用：
```jsx
// pages/index.js
export default function Home() {
  console.log('API URL:', process.env.NEXT_PUBLIC_API_URL)
  return <h1>检查控制台查看环境变量</h1>
}
```

---

## 9. 部署方案

**常用平台：**
- **Vercel：** 官方平台，支持零配置部署和全局 CDN 加速。
- **其他平台：** Netlify、Heroku、AWS 等。

**示例：**
在 Vercel 上部署时，只需将项目连接到 Vercel 账户，然后使用 `next build` 构建项目，Vercel 会自动处理部署和路由配置。

---

## 10. 国际化（i18n）

**概念：**  
Next.js 从 10 版本开始内置国际化支持，在 `next.config.js` 中配置多语言选项。

**示例代码：**
```js
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'zh'],
    defaultLocale: 'en',
  },
}
```
访问 `http://localhost:3000/zh` 将自动切换到中文页面。

---

## 11. 图片优化

**概念：**  
Next.js 内置 Image 组件可自动处理图片懒加载、格式转换、尺寸调整等优化任务。

**示例代码：**
```jsx
// pages/image-demo.js
import Image from 'next/image'

export default function ImageDemo() {
  return (
    <div>
      <h1>图片优化示例</h1>
      <Image
        src="/images/sample.jpg"
        alt="示例图片"
        width={600}
        height={400}
        quality={80}
      />
    </div>
  )
}
```

---

## 12. 性能优化

**常见措施：**
- 根据页面特性选择 SSR、SSG 或 ISR 以平衡数据实时性和加载速度。
- 利用自动代码分割和动态导入减小初始加载体积。
- 使用 Next.js Image 组件和 CDN 缓存静态资源。
- 利用 Bundle 分析工具检查并优化代码包大小。

**示例：**  
你可以在 `package.json` 中添加如下脚本以启动 Bundle 分析：
```json
"scripts": {
  "analyze": "cross-env ANALYZE=true next build"
}
```
然后运行 `npm run analyze` 来查看项目的打包情况。

---
