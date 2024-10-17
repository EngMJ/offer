# Vite 5.4.9 概述

## vite 与 webpack 优缺点

`Vite` 和 `Webpack` 是两种流行的前端构建工具，它们各有优缺点，适用于不同的场景。以下是它们的主要区别和优缺点：

### 1. **构建速度**

- **Vite**：
    - **开发服务器启动速度快**：Vite 使用原生 ES 模块（ESM）直接在浏览器中加载模块，而不是像 Webpack 那样先进行打包。这意味着 Vite 无需在启动时打包整个应用，只需要转换和提供页面中实际使用的模块，极大地缩短了开发服务器的启动时间，尤其是在大型项目中表现尤为突出。
    - **按需编译**：Vite 只会在请求时即时编译和加载模块，这种按需编译机制使得它的构建速度在大型项目中非常快。

- **Webpack**：
    - **开发服务器启动较慢**：Webpack 需要在启动开发服务器时打包所有模块，尤其是在大型项目中，启动时间可能会较长。
    - **打包后加载**：Webpack 在开发阶段会将所有代码打包到一个或多个文件中，加载时一次性读取，编译时间较长。

**总结**：Vite 在开发阶段的启动速度和响应速度都明显优于 Webpack，特别是对于大型项目。

### 2. **热更新 (HMR)**

- **Vite**：
    - **极速热更新**：由于 Vite 只重新编译发生变化的模块，并且直接通过 ESM 提供给浏览器，因此热更新非常快，能实时反映代码改动，页面刷新也更少，用户体验更好。

- **Webpack**：
    - **热更新较慢**：Webpack 在热更新时需要重新打包相关模块，并更新整个 bundle，速度相对较慢，特别是在大型项目中表现较为明显。

**总结**：Vite 的热更新速度通常快于 Webpack，提供了更流畅的开发体验。

### 3. **打包速度**

- **Vite**：
    - **生产环境打包使用 Rollup**：Vite 在生产环境下依赖 Rollup 进行打包。Rollup 作为一种较为轻量的打包工具，适合对代码进行高效的 Tree-shaking 和按需加载，生成较为精简的打包结果，但对于极其复杂的构建场景可能不如 Webpack 灵活。

- **Webpack**：
    - **高效增量打包**：Webpack 在生产模式下具有更灵活和强大的打包能力，支持各种复杂的插件和 loader，可以进行高度定制的优化。例如代码拆分、懒加载、动态导入等功能支持非常全面。
    - **慢速但灵活**：Webpack 在复杂项目中的打包时间可能会较长，但可以通过多种插件和配置进行优化，比如 `TerserPlugin`、`MiniCssExtractPlugin`、`Code Splitting` 等。

**总结**：Vite 的打包速度在简单和中型项目中可能更快，而 Webpack 在大型项目或需要高度定制化时更具优势。

### 4. **生态系统与插件**

- **Vite**：
    - **轻量但快速发展的生态系统**：Vite 的插件系统借鉴了 Rollup，但相对来说生态系统还在快速发展中。对于某些特定场景（例如复杂的插件需求），可能不如 Webpack 生态成熟。不过，Vite 的插件生态正在不断增长，常见功能如样式处理、图片压缩等已有较多插件支持。
    - **兼容 Rollup 插件**：Vite 的插件系统是基于 Rollup 的，因此许多现有的 Rollup 插件可以直接用于 Vite。

- **Webpack**：
    - **成熟且庞大的生态系统**：Webpack 经过多年的发展，拥有庞大而成熟的插件和 loader 生态，几乎可以覆盖所有构建场景的需求，无论是代码拆分、模块懒加载、图片压缩，还是复杂的打包优化方案，Webpack 都能找到合适的解决方案。
    - **可定制性强**：Webpack 的插件和 loader 机制非常灵活，几乎可以满足所有场景下的自定义需求。

**总结**：Webpack 具有更加成熟和广泛的生态系统，而 Vite 的生态系统较轻量，但其 Rollup 兼容性使其仍然具备一定的灵活性。

### 5. **配置复杂度**

- **Vite**：
    - **简单配置**：Vite 的配置文件较为简洁，基于现代 ESM 标准。默认情况下，Vite 提供了很多开箱即用的功能，如 JSX 支持、TypeScript 支持等，开发者无需为简单项目做过多配置。

- **Webpack**：
    - **配置复杂**：Webpack 的配置相对复杂，特别是在处理大型项目时，配置文件可能非常冗长，并且需要使用大量的插件和 loader 来处理不同的资源类型和优化场景。

**总结**：Vite 在配置上更加简单直观，而 Webpack 虽然强大但在配置上更加繁琐，特别是在复杂项目中。

### 6. **浏览器支持**

- **Vite**：
    - **现代浏览器优先**：Vite 是为现代浏览器设计的，使用原生 ESM 支持。如果需要支持旧版浏览器（如 IE），则需要额外的配置和插件，如使用 `@vitejs/plugin-legacy`。

- **Webpack**：
    - **支持广泛的浏览器**：Webpack 默认支持广泛的浏览器，包括旧版浏览器，如 IE 11。它可以通过 Babel、Polyfill 等工具轻松实现旧浏览器的兼容。

**总结**：Webpack 对浏览器兼容性支持更为广泛，而 Vite 针对现代浏览器优化，如果需要支持旧浏览器需要额外的插件。

### 7. **适用场景**

- **Vite**：
    - 适合中小型项目，尤其是需要快速启动和热更新的项目。
    - 适用于现代前端框架（如 Vue 3、React）开发，特别是单页面应用（SPA）。
    - 对于不需要复杂构建逻辑的项目非常友好，配置简单且性能出色。

- **Webpack**：
    - 适合大型项目，尤其是需要复杂打包、优化和兼容性的项目。
    - 更适合多页面应用（MPA）和需要复杂插件或自定义配置的大型企业级应用。
    - 适用于需要广泛兼容浏览器支持（包括旧版浏览器）的项目。

### 总结

| 特性             | Vite                              | Webpack                           |
|------------------|-----------------------------------|-----------------------------------|
| **开发启动速度** | 快速，按需编译，适合大型项目        | 较慢，需要打包整个项目              |
| **热更新速度**   | 快，基于原生 ESM                  | 较慢，尤其在大型项目中              |
| **打包速度**     | 生产模式使用 Rollup，速度较快       | 灵活，支持复杂场景，但较慢            |
| **生态系统**     | 轻量，基于 Rollup，快速发展中       | 成熟，插件丰富，支持复杂场景          |
| **配置复杂度**   | 简单，开箱即用                    | 配置复杂，尤其是在复杂项目中          |
| **浏览器支持**   | 现代浏览器优先，需插件支持旧浏览器   | 广泛支持，包括旧版浏览器              |
| **适用场景**     | 适合中小型项目和现代框架开发         | 适合大型项目和复杂构建场景            |

Vite 提供了快速的开发体验，尤其在开发时的启动速度和热更新表现优异；而 Webpack 由于其庞大的生态系统和灵活的插件机制，在大型和复杂项目中仍然是非常强大的工具。

## 配置示例
```js

import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

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
