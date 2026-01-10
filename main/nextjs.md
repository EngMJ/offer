# Next.js 16 常见面试题

## [参考: Next.js 常用内容](./simple_nextjs.md)
## [参考: Next.js 开发部署全过程](./deploy_nextjs.md)

---

## Next.js 16 新特性概览

> Next.js 16 是框架的最新主要版本，带来了重大性能改进和新功能

### Turbopack 稳定版

Turbopack 在 Next.js 16 中已完全稳定，成为默认开发构建工具：

- **开发服务器启动速度提升 10x+**
- **热更新速度提升 700x+**
- **完全替代 Webpack 用于开发环境**

```bash
# Next.js 16 中 Turbopack 默认启用
npx create-next-app@latest my-app
```

### 完整 React 19 支持

Next.js 16 完整支持 React 19 的所有新特性：

- **Server Components**: 稳定的服务器组件
- **Server Actions**: 在服务器上直接执行的函数
- **Actions**: 表单和数据变更处理
- **新 Hooks**: `useActionState`, `useFormStatus`, `useOptimistic`

```jsx
// app/actions.js
'use server'

export async function createPost(formData) {
  const title = formData.get('title')
  await db.posts.create({ title })
  revalidatePath('/posts')
}

// app/posts/create/page.jsx
import { createPost } from '@/app/actions'

export default function CreatePost() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="标题" />
      <button type="submit">创建</button>
    </form>
  )
}
```

### 新的缓存策略

Next.js 16 更新了缓存默认行为：

```js
// next.config.js
module.exports = {
  experimental: {
    // 配置缓存行为
    staleTimes: {
      dynamic: 30, // 动态页面缓存时间（秒）
      static: 180, // 静态页面缓存时间（秒）
    },
  },
}
```

**变更要点：**
- `fetch()` 请求默认不再缓存
- Route Handlers 默认不缓存
- 客户端导航数据默认不缓存

### Partial Prerendering (PPR) 改进

部分预渲染功能更加成熟：

```jsx
// app/page.jsx
import { Suspense } from 'react'

// 静态部分会在构建时预渲染
export default function Page() {
  return (
    <main>
      <h1>我的博客</h1>
      {/* 动态部分会在请求时渲染 */}
      <Suspense fallback={<Loading />}>
        <DynamicContent />
      </Suspense>
    </main>
  )
}
```

### 改进的开发体验

1. **更好的错误提示**：智能错误分析和修复建议
2. **HMR 优化**：保持更多组件状态
3. **TypeScript 支持增强**：更准确的类型推断

---

## 1. **什么是 Next.js？其主要特点有哪些？**  
Next.js 是一个基于 React 的前端框架，主要提供服务端渲染（SSR）和静态站点生成（SSG）的能力。其主要特点包括：
- **自动代码分割与预加载：** 用户仅加载当前页面所需的代码，提升性能；
- **文件系统路由：** 根据 `pages` 文件夹中的文件结构自动生成路由；
- **内置 CSS 支持：** 包括全局 CSS、CSS 模块以及 Sass 等；
- **API 路由：** 无需单独搭建后端即可在 `pages/api` 下编写后端 API 接口；
- **SSR/SSG 与 ISR 支持：** 根据业务需求选择最佳的数据渲染策略。

---

## 2. **Next.js 中有哪些数据获取方法？请分别说明它们的适用场景。**  
Next.js 提供了以下几种数据获取方法：
- **`getStaticProps`：** 在构建时获取数据，适用于内容更新不频繁的页面，生成静态页面；
- **`getServerSideProps`：** 每次请求时获取数据，适用于数据频繁变化或需要实时数据的页面；
- **`getStaticPaths`：** 配合 `getStaticProps` 使用，用于动态路由页面的静态生成，需要提前定义所有可能的动态路由；
- **增量静态生成（ISR）：** 结合 `revalidate` 参数，在静态生成的页面中定期更新数据，兼具静态页面的性能和数据的新鲜度。

---

## 3. **请解释 SSR、SSG 和 ISR 之间的区别及各自的适用场景。**  
- **SSR（Server-Side Rendering）：** 每次请求都在服务器端生成页面内容，适用于实时性要求高、数据动态变化或需要用户个性化展示的场景。
- **SSG（Static Site Generation）：** 在构建时生成静态页面，加载速度快、易于缓存，适用于内容较为固定、更新频率低的页面。
- **ISR（Incremental Static Regeneration）：** 允许静态页面在后台定时重新生成，通过 `revalidate` 参数实现数据定时更新，适用于既希望享受静态页面性能优势又需要定期更新内容的场景。

---

## 4. **Next.js 的路由机制是如何实现的？**  
Next.js 采用基于文件系统的路由机制。具体来说：
- 任何放在 `app` 目录下的 React 组件都会自动映射到对应的 URL 路径，如 `app/index.js` 对应根路径 `/`；
- 动态路由使用中括号表示，例如 `app/posts/[id].js` 对应动态路由 `/posts/:id`；
- 提供了 [Link](https://nextjs.org/docs/api-reference/next/link) 组件用于客户端路由导航，并支持自动预加载页面资源。

---

## 5. **如何在 Next.js 中处理全局样式和 CSS 模块？**  
- **全局样式：** 全局 CSS 文件只能在 `pages/_app.js` 中导入，以确保样式在整个应用中生效。
- **CSS 模块：** 通过创建以 `.module.css` 为后缀的文件，实现样式局部化，避免样式冲突。Next.js 原生支持 CSS 模块，开发者只需在组件中直接导入使用。

---

## 6. **什么是 Next.js 的 API Routes？它们主要解决什么问题？**  
API Routes 允许开发者在 `pages/api` 目录下编写后端 API 接口，每个文件默认导出一个处理 HTTP 请求的函数。这种方式可以：
- 快速实现后端逻辑，如表单提交、用户认证、数据处理等；
- 无需单独搭建后端服务器，使前后端代码在同一项目中管理；
- 便于部署与维护，特别适用于小型或中型应用。

---

## 7. **Next.js 如何实现代码分割和组件懒加载？**  
Next.js 内置自动代码分割机制，页面级代码会自动拆分为独立的代码块，确保用户只加载当前页面所需的代码。对于组件级别的懒加载，可以使用：
- React 的 `lazy` 和 `Suspense`；
- Next.js 提供的 `dynamic` 函数，该函数允许按需加载组件并支持服务端渲染。

---

## 8. **在 Next.js 中如何实现 SEO 优化？**  
由于 Next.js 支持 SSR 和 SSG，可以在服务器端生成完整的 HTML 页面，有助于搜索引擎抓取页面内容。此外，使用 `next/head` 组件可以在每个页面中动态添加 `<title>`、`<meta>` 标签等，进一步优化页面的 SEO 表现。

---

## 9. **描述一下 Next.js 应用的部署流程以及常见的部署平台。**  
部署 Next.js 应用主要包括以下步骤：
- **构建阶段：** 运行 `next build` 生成生产环境所需的构建文件；
- **启动阶段：** 使用 `next start` 启动服务器。  
  常见部署平台包括：
- **Vercel：** Next.js 官方平台，支持无缝部署；
- **Netlify、AWS、Heroku 等：** 也支持 Next.js 部署，但可能需要额外配置构建和路由设置；
- 注意根据平台要求配置环境变量和 CDN 缓存策略。

---

## 10. **Next.js 如何支持国际化（i18n）？**  
从 Next.js 10 版本开始，内置支持国际化。开发者可以在 `next.config.js` 文件中配置 `i18n` 选项，包括：
- 定义支持的语言列表；
- 指定默认语言；
- 自动生成带语言前缀的路由。  
  这样可以方便地为全球用户提供多语言支持。

---

## 11. **如何理解 Incremental Static Regeneration（ISR）？它是如何工作的？**  
ISR 结合了静态站点生成和实时数据更新的优势：
- 在初次构建时生成静态页面；
- 通过设置 `revalidate` 参数，定义一个时间间隔，当用户请求页面且页面超过此时间间隔后，Next.js 会在后台重新生成页面；
- 重新生成后的页面会在后续请求中被使用，从而保证页面数据的时效性，同时保持静态页面的加载速度。

---

## 12. **在 Next.js 项目中如何选择使用 `getStaticProps` 和 `getServerSideProps`？**  
选择依据主要考虑数据更新频率和页面实时性需求：
- **`getStaticProps`：** 当数据较为稳定、更新频率低，或者允许通过 ISR 定时更新时使用，优势在于页面加载速度快；
- **`getServerSideProps`：** 当页面内容需要每次请求都实时获取最新数据，或者需要依赖请求上下文（如用户认证）时使用，虽然会增加服务器负担但确保数据准确。

---

## 13. **在使用 Next.js 动态路由时，如何利用 `getStaticPaths` 进行静态页面生成？**  
在 Next.js 中，动态路由页面（如 `[id].js`）需要配合 `getStaticPaths` 使用。具体流程为：
- 在 `getStaticPaths` 中返回一个包含所有动态路由参数的数组，告诉 Next.js 哪些页面需要预先生成；
- 然后使用 `getStaticProps` 根据传入的参数获取数据，生成静态页面；  
  适用于博客、商品详情等页面预渲染场景。

---

## 14. **介绍 Next.js 中的 `next/head` 组件及其主要用途。**  
`next/head` 组件允许在页面中动态添加 `<head>` 标签内的元素，如 `<title>`、`<meta>`、`<link>` 等。主要用途包括：
- 设置页面标题，提升用户体验；
- 添加 SEO 优化所需的 meta 标签；
- 引入外部资源，如字体、样式表等，帮助构建更丰富的页面。

---

## 15. **如何在 Next.js 中配置和使用环境变量？**  
Next.js 支持通过 `.env` 文件配置环境变量：
- 在项目根目录下创建 `.env.local`（开发环境）或 `.env.production`（生产环境）等文件；
- 使用 `process.env.VARIABLE_NAME` 访问环境变量；
- 注意：若要在客户端代码中访问，需要在变量名前添加 `NEXT_PUBLIC_` 前缀。

---

## 16. **与 Create React App（CRA）相比，Next.js 有哪些显著优势？**  
Next.js 与 CRA 的主要区别和优势在于：
- **服务端渲染（SSR）和静态站点生成（SSG）：** 提供更好的 SEO 和首屏加载速度；
- **文件系统路由：** 自动生成路由，无需额外配置；
- **内置 API 路由：** 允许在同一项目中构建前后端；
- **性能优化：** 自动代码分割、图片优化等内置功能，使得 Next.js 更适合构建大规模、高性能应用。

---

## 17. **如何在 Next.js 中实现客户端路由导航？**  
Next.js 提供了内置的 [Link](https://nextjs.org/docs/api-reference/next/link) 组件来实现客户端路由导航：
- 使用 `<Link href="/目标路由">` 包裹需要点击跳转的元素；
- Link 组件支持预加载目标页面的代码和数据，从而实现更流畅的页面切换体验；
- 该方式不需要刷新整个页面，保持 SPA 的体验。

---

## 18. **请描述 Next.js 中的数据预取机制及其工作原理。**  
Next.js 利用内置的路由组件，在用户浏览当前页面时，会预先加载与下一个可能访问页面相关的代码和数据：
- 通过 [Link](https://nextjs.org/docs/api-reference/next/link) 组件自动预取页面资源；
- 在后台加载数据与 JavaScript 代码，使得用户点击链接时能够更快地展示页面；
- 这种机制提升了应用的响应速度和用户体验。

---

## 19. **如何在 Next.js 中实现自定义 404 页面和错误处理？**  
- **自定义 404 页面：** 在 `pages` 目录下创建一个 `404.js` 文件，Next.js 会自动将其作为未匹配路由时显示的页面；
- **全局错误处理：** 可以通过创建 `pages/_error.js`（或在新版 Next.js 中使用错误边界）来自定义其他错误状态下的展示逻辑，为用户提供统一的错误提示界面。

---

## 20. **有哪些常见的性能优化措施适用于 Next.js 应用？**  
针对 Next.js 应用，可以采取以下优化措施：
- **利用 SSR/SSG/ISR：** 根据页面需求选择合适的数据获取方式，提升首屏速度；
- **自动代码分割与懒加载：** 仅加载当前页面必需代码，减少初始加载时间；
- **图片优化：** 使用 Next.js 内置的 Image 组件进行图片压缩与优化；
- **静态资源缓存：** 配置 CDN 和缓存策略，加速静态资源的加载；
- **减少客户端 JavaScript：** 精简不必要的依赖，优化 bundle 大小；
- **使用 Web Vitals 监控：** 持续监控性能指标，定位并改进性能瓶颈。

---

## App Router 高级面试题

---

## 21. **App Router 与 Pages Router 有什么区别？如何选择？**

| 特性 | App Router (`app/`) | Pages Router (`pages/`) |
|------|---------------------|-------------------------|
| 组件模式 | 默认 Server Components | 默认 Client Components |
| 数据获取 | `fetch` + `cache` | `getStaticProps` / `getServerSideProps` |
| 布局 | 嵌套布局，自动共享 | 需手动实现 |
| 路由文件 | `page.js`, `layout.js`, `loading.js` | 文件即路由 |
| Streaming | 原生支持 | 不支持 |
| React 19 | 完整支持 | 部分支持 |

**选择建议：**
- 新项目推荐使用 App Router
- 旧项目可渐进式迁移，两者可共存
- 需要完整 React 19 特性时必须使用 App Router

---

## 22. **Server Components 和 Client Components 的区别是什么？何时使用？**

**Server Components（默认）：**
- 在服务器端渲染，不发送 JavaScript 到客户端
- 可以直接访问数据库、文件系统、环境变量
- 不能使用 `useState`、`useEffect` 等 Hooks
- 不能使用浏览器 API

**Client Components：**
- 需要在文件顶部添加 `'use client'`
- 可以使用 React Hooks 和浏览器 API
- 可以添加交互事件（onClick 等）

```jsx
// Server Component（默认）
async function ServerComponent() {
  const data = await db.query('SELECT * FROM posts')
  return <div>{data.map(post => <p key={post.id}>{post.title}</p>)}</div>
}

// Client Component
'use client'
import { useState } from 'react'

function ClientComponent() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

**使用原则：**
- 默认使用 Server Components
- 需要交互、状态、浏览器 API 时才使用 Client Components
- Client Components 可以包含 Server Components（通过 children）

---

## 23. **Next.js Middleware 是什么？有哪些应用场景？**

Middleware 在请求完成之前运行，可以修改响应、重定向、重写 URL 等。

```js
// middleware.js（项目根目录）
import { NextResponse } from 'next/server'

export function middleware(request) {
  // 1. 认证检查
  const token = request.cookies.get('token')
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  // 2. 国际化重定向
  const locale = request.headers.get('accept-language')?.split(',')[0]
  if (locale?.startsWith('zh') && !request.nextUrl.pathname.startsWith('/zh')) {
    return NextResponse.redirect(new URL(`/zh${request.nextUrl.pathname}`, request.url))
  }

  // 3. 添加自定义 Header
  const response = NextResponse.next()
  response.headers.set('x-custom-header', 'value')
  return response
}

// 配置匹配路径
export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*']
}
```

**常见应用场景：**
- 用户认证和授权
- 国际化路由
- A/B 测试
- Bot 检测
- 请求日志
- 地理位置重定向

---

## 24. **App Router 中如何进行数据获取？与 Pages Router 有何不同？**

**App Router 数据获取方式：**

```jsx
// 1. 直接在 Server Component 中 fetch
async function Page() {
  // 默认会被缓存（等同于 getStaticProps）
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()
  return <PostList posts={posts} />
}

// 2. 禁用缓存（等同于 getServerSideProps）
async function Page() {
  const res = await fetch('https://api.example.com/posts', {
    cache: 'no-store'
  })
  const posts = await res.json()
  return <PostList posts={posts} />
}

// 3. 定时重新验证（等同于 ISR）
async function Page() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 } // 60 秒后重新验证
  })
  const posts = await res.json()
  return <PostList posts={posts} />
}

// 4. 使用 unstable_cache 缓存数据库查询
import { unstable_cache } from 'next/cache'

const getCachedPosts = unstable_cache(
  async () => await db.posts.findMany(),
  ['posts'],
  { revalidate: 3600 }
)
```

**对比：**
| Pages Router | App Router |
|--------------|------------|
| `getStaticProps` | `fetch()` 默认缓存 |
| `getServerSideProps` | `fetch({ cache: 'no-store' })` |
| `revalidate` | `fetch({ next: { revalidate: 60 } })` |

---

## 25. **Route Handlers 是什么？如何使用？**

Route Handlers 是 App Router 中的 API 路由，用于创建后端接口。

```js
// app/api/posts/route.js
import { NextResponse } from 'next/server'

// GET 请求
export async function GET(request) {
  const posts = await db.posts.findMany()
  return NextResponse.json(posts)
}

// POST 请求
export async function POST(request) {
  const body = await request.json()
  const post = await db.posts.create({ data: body })
  return NextResponse.json(post, { status: 201 })
}

// 动态路由：app/api/posts/[id]/route.js
export async function GET(request, { params }) {
  const { id } = await params
  const post = await db.posts.findUnique({ where: { id } })
  
  if (!post) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 })
  }
  
  return NextResponse.json(post)
}

// DELETE 请求
export async function DELETE(request, { params }) {
  const { id } = await params
  await db.posts.delete({ where: { id } })
  return new NextResponse(null, { status: 204 })
}
```

**支持的 HTTP 方法：** `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`

---

## 26. **如何使用 Streaming 和 Suspense 实现流式渲染？**

流式渲染允许逐步发送 HTML，提升首屏加载速度。

```jsx
// app/page.jsx
import { Suspense } from 'react'

// 慢速数据组件
async function SlowComponent() {
  const data = await fetch('https://slow-api.com/data', { cache: 'no-store' })
  return <div>{/* 渲染数据 */}</div>
}

// 页面组件
export default function Page() {
  return (
    <main>
      {/* 这部分立即显示 */}
      <h1>页面标题</h1>
      
      {/* 这部分会流式加载 */}
      <Suspense fallback={<div>加载中...</div>}>
        <SlowComponent />
      </Suspense>
    </main>
  )
}
```

**使用 loading.js 实现页面级 Loading：**

```jsx
// app/dashboard/loading.jsx
export default function Loading() {
  return <div className="skeleton">加载中...</div>
}

// app/dashboard/page.jsx
// 此页面加载时会自动显示 loading.jsx
export default async function DashboardPage() {
  const data = await fetchDashboardData()
  return <Dashboard data={data} />
}
```

---

## 27. **Next.js 16 中的 Metadata API 如何使用？**

Metadata API 用于管理页面的 SEO 元数据。

```jsx
// 1. 静态元数据
// app/page.jsx
export const metadata = {
  title: '首页 - 我的网站',
  description: '这是网站首页的描述',
  keywords: ['Next.js', 'React', 'SEO'],
  openGraph: {
    title: '首页',
    description: '首页描述',
    images: ['/og-image.png'],
  },
  twitter: {
    card: 'summary_large_image',
    title: '首页',
  },
}

// 2. 动态元数据
// app/posts/[id]/page.jsx
export async function generateMetadata({ params }) {
  const { id } = await params
  const post = await getPost(id)
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      images: [post.coverImage],
    },
  }
}

// 3. 模板和默认值
// app/layout.jsx
export const metadata = {
  title: {
    template: '%s | 我的网站',
    default: '我的网站',
  },
  metadataBase: new URL('https://mysite.com'),
}
```

---

## 28. **如何在 Next.js 中实现并行路由和拦截路由？**

**并行路由（Parallel Routes）：** 同时渲染多个页面

```
app/
├── layout.jsx
├── page.jsx
├── @modal/
│   └── page.jsx
└── @sidebar/
    └── page.jsx
```

```jsx
// app/layout.jsx
export default function Layout({ children, modal, sidebar }) {
  return (
    <div>
      <aside>{sidebar}</aside>
      <main>{children}</main>
      {modal}
    </div>
  )
}
```

**拦截路由（Intercepting Routes）：** 在当前布局中显示其他路由

```
app/
├── feed/
│   └── page.jsx
├── photo/
│   └── [id]/
│       └── page.jsx
└── @modal/
    └── (..)photo/
        └── [id]/
            └── page.jsx
```

- `(.)` - 拦截同级路由
- `(..)` - 拦截上一级路由
- `(..)(..)` - 拦截上两级路由
- `(...)` - 拦截根路由

**使用场景：** Modal 弹窗展示详情页（如 Instagram 图片预览）

---

## 29. **Next.js 中的错误处理机制有哪些？**

```jsx
// 1. error.js - 捕获路由段内的错误
// app/dashboard/error.jsx
'use client'

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>出错了！</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>重试</button>
    </div>
  )
}

// 2. global-error.js - 捕获根布局错误
// app/global-error.jsx
'use client'

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>发生严重错误</h2>
        <button onClick={() => reset()}>重试</button>
      </body>
    </html>
  )
}

// 3. not-found.js - 404 页面
// app/not-found.jsx
export default function NotFound() {
  return (
    <div>
      <h2>页面不存在</h2>
      <Link href="/">返回首页</Link>
    </div>
  )
}

// 手动触发 404
import { notFound } from 'next/navigation'

async function Page({ params }) {
  const post = await getPost(params.id)
  if (!post) notFound()
  return <Post post={post} />
}
```

---

## 30. **如何在 Next.js 中使用 Server Actions？**

Server Actions 允许在客户端直接调用服务器函数，无需创建 API 路由。

```jsx
// 方式 1：在 Server Component 中定义
// app/posts/page.jsx
async function createPost(formData) {
  'use server'
  const title = formData.get('title')
  await db.posts.create({ data: { title } })
  revalidatePath('/posts')
}

export default function PostsPage() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">创建</button>
    </form>
  )
}

// 方式 2：在单独文件中定义
// app/actions.js
'use server'

export async function createPost(formData) {
  const title = formData.get('title')
  await db.posts.create({ data: { title } })
  revalidatePath('/posts')
}

export async function deletePost(id) {
  await db.posts.delete({ where: { id } })
  revalidatePath('/posts')
}

// 方式 3：配合 useActionState 使用
'use client'
import { useActionState } from 'react'
import { createPost } from './actions'

function CreatePostForm() {
  const [state, formAction, isPending] = useActionState(createPost, null)
  
  return (
    <form action={formAction}>
      <input name="title" />
      <button disabled={isPending}>
        {isPending ? '创建中...' : '创建'}
      </button>
      {state?.error && <p>{state.error}</p>}
    </form>
  )
}
```

**Server Actions 优势：**
- 减少客户端 JavaScript
- 自动处理表单提交
- 支持渐进增强（无 JS 也能工作）
- 类型安全（配合 TypeScript）

---
