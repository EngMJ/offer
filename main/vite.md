## Vite 5.x 概述

## 优缺点

## webpack区别

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
