# Vite 5.4.9 概述

## Vite 与 webpack 优缺点

### 1. **构建速度**

- **Vite**：
    - **开发服务器启动速度快**：Vite 使用原生 ES 模块（ESM）直接在浏览器中加载模块，并且使用` esbuild 预构建依赖`。
    - **按需加载开发代码**：Vite 只会在请求时即时编译和加载模块，这种按需编译机制使得它的构建速度在大型项目中非常快。

- **Webpack**：
    - **开发服务器启动较慢**：Webpack 需要在启动开发服务器时`打包所有模块`，尤其是在大型项目中，启动时间可能会较长。
    - **打包后加载**：Webpack 在开发阶段会将所有代码打包到一个或多个文件中，加载时一次性读取，编译时间较长。

**总结**：Vite 在开发阶段的启动速度和响应速度都明显优于 Webpack，特别是对于大型项目。

### 2. **热更新 (HMR)**

- **Vite**：
    - **极速热更新**：由于 Vite `只重新编译发生变化的模块`，并且直接通过 ESM 提供给浏览器，因此热更新非常快，能实时反映代码改动，页面刷新也更少，用户体验更好。

- **Webpack**：
    - **热更新较慢**：Webpack 在热更新时`需要重新打包整个模块`，并更新整个 bundle，速度相对较慢，特别是在大型项目中表现较为明显。

**总结**：Vite 的热更新速度通常快于 Webpack，提供了更流畅的开发体验。

### 3. **打包速度**

- **Vite**：
    - **生产环境打包使用 Rollup,快速,复杂项目灵活性低**：Vite 在生产环境下依赖 Rollup 进行打包。Rollup 作为一种较为轻量的打包工具，适合对代码进行高效的 Tree-shaking 和按需加载，生成较为精简的打包结果，但对于极其复杂的构建场景可能不如 Webpack 灵活。

- **Webpack**：
    - **打包复杂项目,扩展性强**：Webpack 在生产模式下具有更灵活和强大的打包能力，支持各种复杂的插件和 loader，可以进行高度定制的优化。例如代码拆分、懒加载、动态导入等功能支持非常全面。
    - **慢速但灵活**：Webpack 在复杂项目中的打包时间可能会较长，但可以通过多种插件和配置进行优化，比如 `TerserPlugin`、`MiniCssExtractPlugin`、`Code Splitting` 等。

**总结**：Vite 的打包速度在简单和中型项目中可能更快，而 Webpack 在大型项目或需要高度定制化时更具优势。

### 4. **生态系统与插件**

- **Vite**：
    - **数量较少,正在快速发展的生态系统**：Vite 的插件系统借鉴了 Rollup，但相对来说生态系统还在快速发展中。对于某些特定场景（例如复杂的插件需求），可能不如 Webpack 生态成熟。不过，Vite 的插件生态正在不断增长，常见功能如样式处理、图片压缩等已有较多插件支持。
    - **兼容 Rollup 插件**：Vite 的插件系统是基于 Rollup 的，因此许多现有的 Rollup 插件可以直接用于 Vite。

- **Webpack**：
    - **数量多且成熟的生态系统**：Webpack 经过多年的发展，拥有庞大而成熟的插件和 loader 生态，几乎可以覆盖所有构建场景的需求，无论是代码拆分、模块懒加载、图片压缩，还是复杂的打包优化方案，Webpack 都能找到合适的解决方案。
    - **可定制性强**：Webpack 的插件和 loader 机制非常灵活，几乎可以满足所有场景下的自定义需求。

**总结**：Webpack 具有更加成熟和广泛的生态系统，而 Vite 的生态系统较轻量，但其 Rollup 兼容性使其仍然具备一定的灵活性。

### 5. **配置复杂度**

- **Vite**：
    - **简单配置,易上手**：Vite 的配置文件较为简洁，基于现代 ESM 标准。默认情况下，Vite 提供了很多开箱即用的功能，如 JSX 支持、TypeScript 支持等，开发者无需为简单项目做过多配置。

- **Webpack**：
    - **配置复杂,扩展性强**：Webpack 的配置相对复杂，特别是在处理大型项目时，配置文件可能非常冗长，并且需要使用大量的插件和 loader 来处理不同的资源类型和优化场景。

**总结**：Vite 在配置上更加简单直观，而 Webpack 虽然强大但在配置上更加繁琐，特别是在复杂项目中。

### 6. **浏览器支持**

- **Vite**：
    - **优先支持现代浏览器,旧浏览器支持差**：Vite 是为现代浏览器设计的，使用原生 ESM 支持。如果需要支持旧版浏览器（如 IE），则需要额外的配置和插件，如使用 `@vitejs/plugin-legacy`。

- **Webpack**：
    - **支持新旧浏览器**：Webpack 默认支持广泛的浏览器，包括旧版浏览器，如 IE 11。它可以通过 Babel、Polyfill 等工具轻松实现旧浏览器的兼容。

**总结**：Webpack 对浏览器兼容性支持更为广泛，而 Vite 针对现代浏览器优化，如果需要支持旧浏览器需要额外的插件。

### 7. **开发环境与生产环境包构建一致性**

- **Vite**：
  - **开发环境ESBUILD构建,生产环境使用Rollup构建**：由于使用不同的技术栈,构建的结果一致性有时无法保证.

- **Webpack**：
  - **开发与生产环境使用相同构建技术**：Webpack 开发与生产环境一致性稳定。

**总结**：Webpack 构建一致性是稳定的，而 Vite 构建一致性并不稳定。

### 8. **适用场景**

- **Vite**：
    - 适合`中小型项目`，尤其是需要快速启动和热更新的项目。
    - `适用于现代前端框架`（如 Vue 3、React）开发，特别是单页面应用（SPA）。
    - 对于`不需要复杂构建逻辑的项目`非常友好，配置简单且性能出色。

- **Webpack**：
    - `适合大型项目`，尤其是需要复杂打包、优化和兼容性的项目。
    - 更适合多页面应用（MPA）和需要复杂插件或自定义配置的大型企业级应用。
    - 适用于需要`广泛兼容浏览器支持`（包括旧版浏览器）的项目。


### 总结

| 特性             | Vite                   | Webpack               |
|----------------|------------------------|-----------------------|
| **开发启动速度**     | 快速，依赖预构建，按需编译启动内容      | 较慢，需要先打包整个项目再启动       |
| **热更新速度**      | 快，基于原生 ESM,按需更新实际变化的模块 | 较慢，尤其在大型项目中, 重新打包整个模块 |
| **打包速度**       | 生产模式使用 Rollup，速度较快     | 灵活扩展性强，支持复杂场景，但较慢     |
| **生态系统**       | 较少，基于 Rollup           | 成熟，插件丰富，支持复杂场景        |
| **配置复杂度**      | 简单，开箱即用,扩展性相对不足        | 配置复杂，尤其是在复杂项目中, 扩展性强  |
| **浏览器支持**      | 现代浏览器优先，需插件支持旧浏览器      | 广泛支持新旧版浏览器            |
| **开发与生产构建一致性** | 构建一致性并不稳定              | 构建一致性是稳定的             |
| **适用场景**       | 适合中小型项目和现代框架开发         | 适合大型项目和复杂构建场景         |

## 配置示例
```js

import { defineConfig, loadEnv } from 'vite';
import vue from '@vitejs/plugin-vue';

// defineConfig 不用 jsdoc 注解也可以获取ts类型提示

// 1. 函数参数
export default defineConfig(({ command, mode, isSsrBuild, isPreview }) => {
  //  command参数 serve || build
  //  isSsrBuild ssr环境
  //  isPreview 预构建环境
  //  mode 编译时传递的参数,如 vite build --mode staging, staging字符串就是当前的mode
  if (command === 'serve') {
    return {
      // dev 独有配置
    }
  } else {
    // command === 'build'
    return {
      // build 独有配置
    }
  }
})
// 异步函数
export default defineConfig(async ({ command, mode }) => {
  const data = await asyncFunction()
  return {
    // vite 配置
  }
})

// .env文件加载
export default defineConfig(({ command, mode }) => {
  // 根据当前工作目录中的 `mode` 加载 .env 文件
  // 设置第三个参数为 '' 来加载所有环境变量，而不管是否有 `VITE_` 前缀。
  const env = loadEnv(mode, process.cwd(), '')
  return {
    // vite 配置
    define: {
      __APP_ENV__: JSON.stringify(env.APP_ENV),
    },
  }
})


// 2. 对象参数
export default defineConfig({
    // 应用的基础公共路径, 通常用于子路径部署。默认为 '/'，如果部署到子目录则需要指定，如 '/my-app/'
    base: '/',

    // 用于定义全局常量，静态替换构建时的值
    define: {
        __APP_VERSION__: JSON.stringify('1.0.0'), // 定义全局变量，可以在应用中使用
        'process.env.NODE_ENV': JSON.stringify('production'), // 伪造 `process.env` 中的值，常用于设置环境
    },

    // 配置开发服务器选项
    server: {
        host: '0.0.0.0', // 设置为 '0.0.0.0' 允许外部设备访问本地开发服务器
        port: 3000, // 指定开发服务器监听的端口
        strictPort: true, // 如果指定端口被占用，直接退出而不是尝试其他端口
        open: true, // 启动开发服务器时自动打开浏览器
        https: false, // 是否启用 HTTPS，默认为 false
        cors: true, // 是否允许跨域请求，默认为 false
        force: true, // 如果为 true，将强制重新启动服务器以清理缓存

        // 代理配置，解决开发环境下的跨域问题
        proxy: {
            '/api': {
                target: 'http://localhost:5000', // 目标服务器地址
                changeOrigin: true, // 修改请求头中的 origin
                rewrite: (path) => path.replace(/^\/api/, ''), // 重写路径，去除 `/api`
                secure: false, // 是否验证 SSL 证书
            },
        },

        // 热模块替换 (HMR) 相关配置
        hmr: {
            overlay: true, // 是否显示 HMR 错误覆盖层
            protocol: 'ws', // HMR 使用的通信协议
            port: 24678, // 指定 HMR 监听的端口
            clientPort: 3000, // 指定客户端连接的端口
        },

        // 文件监听配置
        watch: {
            ignored: ['!**/node_modules/**'], // 忽略哪些文件或目录的变化
            usePolling: true, // 启用轮询模式检查文件变动
            interval: 300, // 轮询的时间间隔（ms）
        },
    },

    // 构建选项配置
    build: {
        target: 'es2020', // 构建目标，支持 es2020 或更高，具体视浏览器支持情况而定
        outDir: 'dist', // 打包后文件输出目录，默认为 'dist'
        assetsDir: 'assets', // 静态资源文件夹名，默认为 'assets'
        assetsInlineLimit: 4096, // 小于此大小的资源将内联为 base64，单位字节，默认为 4096 (4KB)
        cssCodeSplit: true, // 启用/禁用 CSS 代码拆分，默认为 true
        sourcemap: false, // 是否生成 sourcemap 文件，默认为 false，适合生产环境调试
        minify: 'esbuild', // 压缩方式，可选 'esbuild' (默认) 或 'terser'
        manifest: false, // 是否生成 manifest.json 文件，适用于后端集成
        emptyOutDir: true, // 打包前是否清空输出目录，默认为 true
        brotliSize: true, // 是否计算并显示打包文件的 brotli 压缩大小，默认为 true
        chunkSizeWarningLimit: 500, // 触发警告的 chunk 文件大小（KB），默认为 500KB

        rollupOptions: {
            input: {
                main: './index.html', // 构建入口文件
                nested: './src/nested.html', // 可以有多个入口文件
            },
            output: {
                // 输出文件配置
                dir: 'dist', // 输出目录
                entryFileNames: '[name].[hash].js', // 入口文件名格式
                chunkFileNames: '[name].[hash].js', // 非入口 chunk 的文件名格式
                assetFileNames: '[name].[hash].[ext]', // 静态资源文件名格式
                manualChunks(id) {
                    if (id.includes('node_modules')) {
                        return 'vendor'; // 手动将 node_modules 中的代码打包到 vendor 文件中
                    }
                },
            },
        },
    },

    // 处理特定类型的静态资源
    assetsInclude: ['**/*.gltf'], // 包含特定类型的文件进行处理，默认为所有静态资源文件

    // CSS 相关配置
    css: {
        modules: {
            scopeBehaviour: 'local', // 样式的作用域范围，默认 'local'，可选 'global'
            generateScopedName: '[name]__[local]___[hash:base64:5]', // 自定义生成的 scoped 名称
            localsConvention: 'camelCaseOnly', // 将类名转换为 camelCase 格式
        },
        preprocessorOptions: {
            // 配置 CSS 预处理器选项
            less: {
                javascriptEnabled: true, // 启用 Less 中的 JavaScript 支持
                modifyVars: { '@primary-color': '#1DA57A' }, // 自定义全局样式变量
            },
            scss: {
                additionalData: `@import "src/styles/variables.scss";`, // 自动导入 SCSS 全局变量文件
            },
        },
        devSourcemap: true, // 开发模式下是否生成 CSS source map，默认 false
        postcss: {
            plugins: [require('autoprefixer')], // 配置 PostCSS 插件，例如 autoprefixer
        },
    },

    // JSON 相关配置
    json: {
        namedExports: true, // 支持从 JSON 文件中导出命名导出，默认 true
        stringify: false, // 是否将 JSON 文件视为字符串而非对象，默认为 false
    },

    // 插件配置，用于添加 Vite 插件
    plugins: [
        vue(), // 支持 Vue 单文件组件 (SFC)
    ],

    // 解析模块时的行为配置
    resolve: {
        alias: {
            '@': '/src', // 路径别名配置，@ 代表 /src 目录
            'components': '/src/components', // 其他自定义别名
        },
        extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'], // 文件扩展名解析顺序
        dedupe: ['vue'], // 防止模块重复安装，通常用于防止不同版本的 Vue 冲突
    },

    // 依赖优化配置
    optimizeDeps: {
        include: ['vue', 'vue-router'], // 强制预构建的依赖
        exclude: ['some-large-lib'], // 排除不需要优化的依赖
        esbuildOptions: {
            target: 'es2020', // 依赖的编译目标，默认 'es2020'
            minify: true, // 是否压缩依赖，默认 true
        },
    },

    // 构建后预览配置，适用于本地预览生产构建结果
    preview: {
        port: 5000, // 预览服务器的端口，默认为 5000
        strictPort: true, // 如果端口不可用则直接退出
        open: false, // 是否自动打开浏览器
        https: false, // 是否启用 HTTPS
        proxy: {
            '/api': {
                target: 'http://localhost:3000',
                changeOrigin: true,
            },
        },
    },

    // Web Worker 配置
    worker: {
        format: 'es', // Worker 的格式，默认为 'es' (可选 'iife')
        plugins: [], // 为 Worker 添加插件
    },

    // 调整 Vite 的日志输出级别
    logLevel: 'info', // 日志级别：'info' | 'warn' | 'error' | 'silent'

    // 控制实验性特性
    experimental: {
        renderBuiltUrl: false, // 是否允许自定义公共 URL 的处理方式
        progressiveEnhance: true, // 渐进式增强特性，启用某些优化
    },
});

```


## 常用插件

### 1. **`@vitejs/plugin-vue`**
- **用途**: 支持 Vue 3 单文件组件（SFC），使 Vite 可以处理 `.vue` 文件。
- **功能**: 解析 Vue 文件中的 `<template>`、`<script>` 和 `<style>` 块，并支持 Vue 的响应式特性。

```bash
# 安装插件
npm install @vitejs/plugin-vue --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

// 定义 Vite 配置
export default defineConfig({
  // 添加 Vue 插件，Vite 将能识别和处理 .vue 文件
  plugins: [vue()],
});
```

### 2. **`@vitejs/plugin-react`**
- **用途**: 在 Vite 中支持 React 项目。
- **功能**: 处理 JSX 语法、TypeScript 支持，并启用 React 的快速刷新（Fast Refresh）功能。

```bash
# 安装插件
npm install @vitejs/plugin-react --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// 定义 Vite 配置
export default defineConfig({
  // 添加 React 插件，使 Vite 支持 React 项目
  plugins: [react()],
});
```

### 3. **`@vitejs/plugin-legacy`**
- **用途**: 使 Vite 打包后的项目能够兼容不支持 ES 模块的旧版浏览器，如 IE 11。
- **功能**: 使用 Babel 将现代 JavaScript 语法转换为旧版浏览器可运行的代码。

```bash
# 安装插件
npm install @vitejs/plugin-legacy --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import legacy from '@vitejs/plugin-legacy';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    legacy({
      // targets 表示支持的浏览器版本，"defaults" 是通用的现代浏览器配置，"not IE 11" 明确排除 IE 11
      targets: ['defaults', 'not IE 11'],
    }),
  ],
});
```

### 4. **`vite-plugin-pwa`**
- **用途**: 让应用成为渐进式 Web 应用（PWA），实现离线缓存和安装功能。
- **功能**: 自动生成 `manifest.json` 文件，注册 Service Worker，管理离线缓存等。

```bash
# 安装插件
npm install vite-plugin-pwa --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    VitePWA({
      // registerType 设置自动更新方式，"autoUpdate" 会在后台自动更新 Service Worker
      registerType: 'autoUpdate',
      // manifest 是 PWA 的配置信息，包括应用的名称、图标、主题色等
      manifest: {
        name: 'My App',
        short_name: 'App',
        theme_color: '#ffffff',
        icons: [
          {
            src: 'icon-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
        ],
      },
    }),
  ],
});
```

### 5. **`vite-plugin-compression`**
- **用途**: 为打包后的文件启用 gzip 或 brotli 压缩，减小文件体积，加快加载速度。
- **功能**: 自动生成压缩版本的打包文件。

```bash
# 安装插件
npm install vite-plugin-compression --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import compression from 'vite-plugin-compression';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 使用默认配置生成 gzip 压缩文件
    compression(),
    // 也可以配置生成 brotli 压缩文件
    compression({
      algorithm: 'brotliCompress',
    }),
  ],
});
```

### 6. **`vite-plugin-eslint`**
- **用途**: 实时在开发阶段检查代码中的 ESLint 问题。
- **功能**: 开发过程中会在终端或浏览器中显示 ESLint 检查结果，帮助保持代码风格一致性。

```bash
# 安装插件
npm install vite-plugin-eslint --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import eslintPlugin from 'vite-plugin-eslint';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 添加 ESLint 插件，启动后会对代码进行实时 ESLint 检查
    eslintPlugin(),
  ],
});
```

### 7. **`vite-plugin-style-import`**
- **用途**: 按需加载 CSS 和样式文件，适合与 UI 库（如 Element Plus、Ant Design）结合使用。
- **功能**: 避免一次性引入整个库的样式，按需导入相关组件所需的样式文件。

```bash
# 安装插件
npm install vite-plugin-style-import --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import styleImport from 'vite-plugin-style-import';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 配置按需引入 UI 库的样式文件
    styleImport({
      libs: [
        {
          // 指定 UI 库为 Element Plus
          libraryName: 'element-plus',
          // 确保样式文件被正确加载
          esModule: true,
          ensureStyleFile: true,
          // 根据组件名动态引入相应的样式文件
          resolveStyle: (name) => {
            return `element-plus/theme-chalk/${name}.css`;
          },
        },
      ],
    }),
  ],
});
```

### 8. **`vite-plugin-svg-icons`**
- **用途**: 将本地 SVG 文件转换为 SVG Sprite，方便在项目中按需使用 SVG 图标。
- **功能**: 自动生成图标雪碧图，减少 HTTP 请求。

```bash
# 安装插件
npm install vite-plugin-svg-icons --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import viteSvgIcons from 'vite-plugin-svg-icons';
import path from 'path';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 配置插件以处理本地 SVG 图标
    viteSvgIcons({
      // 指定 SVG 文件的存储目录
      iconDirs: [path.resolve(process.cwd(), 'src/assets/icons')],
      // symbolId 用于定义生成的图标 ID 格式
      symbolId: 'icon-[dir]-[name]',
    }),
  ],
});
```

### 9. **`vite-plugin-md`**
- **用途**: 支持在 Vite 项目中使用 Markdown 文件，并将其转换为 Vue 组件。
- **功能**: 允许在 Markdown 文件中嵌入 Vue 语法，方便用于文档类项目。

```bash
# 安装插件
npm install vite-plugin-md --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import Markdown from 'vite-plugin-md';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 添加 Markdown 支持插件，能将 Markdown 文件转换为 Vue 组件
    Markdown(),
  ],
  // 配置 Vite 解析 .md 文件
  resolve: {
    extensions: ['.js', '.vue', '.md'],
  },
});
```

### 10. **`vite-tsconfig-paths`**
- **用途**: 支持使用 TypeScript 的路径别名功能，从 `tsconfig.json` 中读取路径配置。
- **功能**: 在导入文件时使用简化路径，如 `@/components/Button` 而不是 `../../components/Button`。

```bash
# 安装插件
npm install vite-tsconfig-paths --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import tsconfigPaths from 'vite-tsconfig-paths';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 启用 TypeScript 路径别名解析
    tsconfigPaths(),
  ],
});
```
