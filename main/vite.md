# Vite 5.4.9 概述

## Vite 与 webpack 优缺点

| 特性             | Vite                                        | Webpack               |
|----------------|---------------------------------------------|-----------------------|
| **开发启动速度**     | 快,使用原生 ES 模块（ESM）和 `esbuild` 预构建依赖，按需编译启动内容 | 较慢，需要先打包整个项目再启动       |
| **热更新速度**      | 快，基于原生 ESM,按需更新实际变化的模块                      | 较慢，尤其在大型项目中, 重新打包整个模块 |
| **打包速度**       | 生产模式使用 Rollup，速度较快                          | 灵活扩展性强，支持复杂场景，但较慢     |
| **生态系统**       | 较少，基于 Rollup                                | 成熟，插件丰富，支持复杂场景        |
| **配置复杂度**      | 简单，开箱即用,扩展性相对不足                             | 配置复杂，尤其是在复杂项目中, 扩展性强  |
| **浏览器支持**      | 现代浏览器优先，需插件支持旧浏览器                           | 广泛支持新旧版浏览器            |
| **开发与生产构建一致性** | 开发环境使用 ESBUILD，生产环境使用 Rollup,构建一致性并不稳定      | 构建一致性是稳定的             |
| **适用场景**       | 适合中小型项目和现代框架开发                              | 适合大型项目和复杂构建场景         |

## 配置示例

### defineConfig 函数参数
```js

import { defineConfig, loadEnv } from 'vite';
import vue from '@vitejs/plugin-vue';

// 使用defineConfig 不用 jsdoc 注解也可以获取ts类型提示
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


```

### defineConfig 对象参数

```js
import { createLogger, defineConfig, loadEnv } from 'vite';
import vue from '@vitejs/plugin-vue';
import { visualizer } from 'rollup-plugin-visualizer'


// 消息处理自定义函数logger
// interface Logger {
//   info(msg: string, options?: LogOptions): void
//           warn(msg: string, options?: LogOptions): void
//           warnOnce(msg: string, options?: LogOptions): void
//           error(msg: string, options?: LogErrorOptions): void
//           clearScreen(type: LogType): void
//           hasErrorLogged(error: Error | RollupError): boolean
//   hasWarned: boolean
// }
const logger = createLogger()
const loggerWarn = logger.warn
logger.warn = (msg, options) => {
  // 忽略空 CSS 文件的警告
  if (msg.includes('vite:css') && msg.includes(' is empty')) return
  loggerWarn(msg, options)
}

//  对象参数
export default defineConfig({
    // 项目根目录路径（index.html 文件所在的位置）
    // 默认值 process.cwd() ,当前路径的相对路径
    root: '/',
  
    // 应用的基础公共路径, 通常用于子路径部署。默认为 '/'，如果部署到子目录则需要指定，如 '/my-app/'
    base: '/',
  
    // 当前模式,在配置中指明将会把 serve 和 build 时的模式 都 覆盖掉。也可以通过命令行 --mode 选项来重写
    mode: 'development', // 生产环境 production

  // 用于指定当前项目的类型或应用类型,影响 Vite 如何处理应用入口以及一些默认行为
  appType:'spa', // 默认 'spa'(单页面文件) | 'mpa'(包含 HTML 中间件) | 'custom'(ssr | 完全自定义 HTML 模板)

  // 处理特定类型的静态资源,直接使用不会被插件转换
  assetsInclude: ['**/*.gltf',/\.txt$/, /\.md$/, /\.csv$/, /\.png$/],

  // 静态资源服务的文件夹名称, 会在打包时直接复制到生产环境包中不做任何处理
  publicDir: 'public',

  // 用于定义全局常量，静态替换构建时的值
  define: {
    __APP_VERSION__: JSON.stringify('1.0.0'), // 定义全局变量，可以在应用中使用
    'process.env.NODE_ENV': JSON.stringify('production'), // 伪造 `process.env` 中的值，常用于设置环境
  },
  
    // 插件配置，用于添加 Vite 插件
    plugins: [
      vue(), // 支持 Vue 单文件组件 (SFC)
      visualizer({ // 类似 webpack-bundle-analyzer ,可视化分析打包文件大小
        open: true, // 构建完成后自动打开报告页面
        gzipSize: true, // 显示 gzip 后的大小
        brotliSize: true // 显示 brotli 后的大小
      })
    ],

    // 解析模块时的行为配置
    resolve: {
      alias: {  // 路径别名
        '@': '/src', // 路径别名配置，@ 代表 /src 目录
        'components': '/src/components', // 其他自定义别名
      },
      dedupe: ['vue'], // 防止依赖模块重复安装，强制引用同一依赖
      /*
      * 浏览器 vs Node.js 环境：许多库会针对浏览器和 Node.js 提供不同的模块版本。
        开发 vs 生产模式：同一模块可能提供未压缩的开发版本和压缩后的生产版本。
        特定平台构建：根据特定平台（如 Electron、React Native 等）提供不同的实现,可以根据平台选择适当的模块版本
      * 
      * */
      conditions: ['import','module','browser','default', 'production', 'development'], // 一个依赖提供多个版本时,使用conditions根据不同环境使用不同版本的模块,按照顺序进行优先加载
      mainFields: ['browser', 'module', 'jsnext:main', 'jsnext'], // package.json 中，在解析依赖时,优先解析的顺序 browser(浏览器) module(node.js)
      extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'], // 导入时想要省略的扩展名列表,加载文件忽略后缀名
      preserveSymlinks: false, // 模块解析符号链接，将符号链接解析为它们指向的实际路径.
      // 为true时保留模块符号链接, 主要作用于 Monorepo 环境下，多个包可能通过符号链接进行解析, 而不是使用实际路径.
    },
  
    html:{ // 定制入口 HTML 文件的处理，比如注入环境变量、动态插入标签等
      cspNonce: 'unique-nonce' // 一个在生成脚本或样式标签时会用到的 nonce 值占位符,还会生成一个带有 nonce 值的 meta 标签。符合CSP 规则,确保页面安全
      /*
      * <!DOCTYPE html>
        <html>
          <head>
            <title>My App</title>
            <meta nonce="unique-nonce"></meta>
            <style nonce="unique-nonce">
              body { background-color: #fff; }
            </style>
          </head>
          <body>
            <script nonce="unique-nonce">
              console.log('This script is allowed by CSP because it has a valid nonce.');
            </script>
          </body>
        </html>
      * 
      * */
    },
  
    // CSS 相关配置
    css: {
      modules: { // 选项将被传递给 postcss-modules
        scopeBehaviour: 'local', // 样式的作用域范围，默认 'local'，可选 'global'
        globalModulePaths: '', // 全局模块路径
        exportGlobals: 'globalName', // 导出全局变量名
        generateScopedName: 'ScopedName', // 生产作用域名
        hashPrefix: 'hashName', // hash前缀
        localsConvention: 'camelCaseOnly', // camelCase 将类名转换为 驼峰 格式 | camelCaseOnly 将类名转换为 驼峰 格式 | dashes 将类名转换为 烤串 格式 | dashesOnly 将类名转换为 烤串 格式
      },
      postcss: { // 内联的 PostCSS 配置（格式同 postcss.config.js），或者一个（默认基于项目根目录的）自定义的 PostCSS 配置路径
        plugins: [require('autoprefixer')], // 配置 PostCSS 插件，例如 autoprefixer
      },
      preprocessorOptions: { // 指定传递给 CSS 预处理器的配置选项
        // 配置 CSS 预处理器选项
        less: { // 传递给less的选项
          javascriptEnabled: true, // 启用 Less 中的 JavaScript 支持
          modifyVars: { '@primary-color': '#1DA57A' }, // 自定义全局样式变量
        },
        stylus:{ // 仅支持 define，可以作为对象传递
          define: {
            $specialColor: new stylus.nodes.RGBA(51, 197, 255, 1),
          },
        },
        scss: { // 传递给scss预处理器的选项
          additionalData: `@import "src/styles/variables.scss";`, // 为每段样式内容添加额外的代码. 该行意思为每个样式文件导入全局scss
          api: 'modern-compiler', // 或 "modern"，"legacy"
          importers: [
            // ...
          ],
        },
      },
      // 以下为css实验性功能,可能移除
      devSourcemap: true, // 开发模式下是否生成 CSS source map，默认 false
      preprocessorMaxWorkers: 0, // CSS 预处理器会尽可能在 worker 线程中运行
      transformer: 'postcss', // 'postcss' | 'lightningcss' , 选择用于 CSS 处理的引擎类型
      lightningcss: {} // 配置 Lightning CSS
    },

  // 配置开发服务器选项
  server: {
    host: '0.0.0.0', // 设置为 '0.0.0.0' 允许外部设备访问本地开发服务器
    port: 3000, // 指定开发服务器监听的端口
    strictPort: true, // 如果指定端口被占用，直接退出而不是尝试其他端口
    https: false, // 是否启用 HTTPS，默认为 false
    open: true, // 启动开发服务器时自动打开浏览器
    // 代理配置，解决开发环境下的跨域问题
    proxy: {
      '/api': {
        target: 'http://localhost:5000', // 目标服务器地址
        changeOrigin: true, // 修改请求头中的 origin
        rewrite: (path) => path.replace(/^\/api/, ''), // 重写路径，去除 `/api`
        secure: false, // 是否验证 SSL 证书
      },
      '/socket.io': {
        target: 'ws://localhost:5174',
        ws: true,
        rewriteWsOrigin: true,
      },
    },
    cors: true, // 是否允许跨域请求，默认为 false
    headers: {}, // 指定服务器响应的header
    // 热模块替换 (HMR) 相关配置
    hmr: true,
    // hmr: {
    //     overlay: true, // 是否显示 HMR 错误覆盖层
    //     protocol: 'ws', // HMR 使用的通信协议
    //     port: 24678, // 指定 HMR 监听的端口
    //     clientPort: 3000, // 指定客户端连接的端口
    // },

    // 提前转换和缓存文件以进行预热,提高初始页面加载速度
    warmup:{
      // 文件路径数组或相对于 root 的 fast-glob 通配
      clientFiles: ['./src/components/*.vue', './src/utils/big-utils.js'], // clientFiles 是仅在客户端使用的文件
      ssrFiles: ['./src/server/modules/*.js'], // ssrFiles 是仅在服务端渲染中使用的文件
    },

    // 文件监听配置
    // Vite 服务器的文件监听器默认会监听 root 目录，同时会跳过 .git/、node_modules/，以及 Vite 的 cacheDir 和 build.outDir 这些目录
    watch: {
      ignored: ['!**/node_modules/**'], // 忽略哪些文件或目录的变化
      usePolling: true, // 启用轮询模式检查文件变动
      interval: 300, // 轮询的时间间隔（ms）
    },
    middlewareMode: false, // 参数: ssr' | 'html',以中间件模式创建 Vite 服务器。
    fs: {
      strict: true, // 禁止 Vite 访问 server.fs.allow 中列出的路径以及项目根目录之外的路径
      allow: ['项目根目录'], // 允许访问项目外部的文件夹路径数组
      deny: ['.env', '.env.*', '*.{crt,pem}'], // 比 server.fs.allow 选项的优先级更高,完全禁止访问的文件名
    },
    origin: 'http://127.0.0.1:8080', // 服务器资源的origin
    sourcemapIgnoreList(sourcePath, sourcemapPath) {
      // sourcePath 资源路径
      // sourcemapPath 映射路径
      // 默认值，将把所有路径中含有 node_modules 的文件添加到忽略列表中
      return sourcePath.includes('node_modules')
    }
  },

  // esbuild配置 参数: 对象 | false
  // 配置 esbuild 的行为，esbuild 在 Vite 中用于依赖预构建以及部分代码的快速转译
  // 在开发和构建过程中，确保代码能快速转译并满足目标平台的要求，同时可以定制某些转换行为。
  esbuild: {
    jsxFactory: 'React.createElement',  // 对应 React 的 JSX 创建元素的方法
    jsxFragment: 'React.Fragment',      // 对应 React 的 Fragment
    include: [/\.tsx/,/\.jsx/,/\.ts/], // 只编译这些内容
    exclude: [/\.js$/], // 排除这些内容
    jsxInject: `import React from 'react'`, // 自动为每一个被 esbuild 转换的文件注入 JSX helper
    target: 'es2015', // 配置目标为 ES2015，保证转换的代码能在大多数现代浏览器中运行
    legalComments: 'none',  // 删除所有注释, eof将注释移至末尾
    minify: true,          // 启用代码压缩
    pure: ['console.log'], // 移除指定的函数调用，通常用于优化生产代码。比如去掉所有 console.log 调用
    loader: 'ts',  // 将 TypeScript 直接编译为 JavaScript，速度更快
  },

  // 配置依赖预构建（pre-bundling）的选项,优化开发环境中依赖的加载和热更新效率，避免因为大量依赖导致启动变慢
  optimizeDeps: {
    entries: [ // 手动设置入口文件,确保加载所有依赖,不设置会自动排除node_modules、build.outDir等文件夹
      'src/main.js', // 手动指定要扫描的入口文件
      'src/components/**/*.vue', // 也可以使用通配符指定多个文件
      'src/pages/*.js',
      '!node_modules/*' // 排除node_modules
    ],
    include: ['vue', 'vue-router'], // 不在 node_modules 中的，链接的包不会被预构建,设置可强制预构建的依赖
    exclude: ['some-large-lib'], // 排除不需要优化的依赖
    esbuildOptions: { // 开发环境传递给esbuild的选项,所有esbuild可以配置这里都可以配置
      // 1. 指定编译的 JavaScript 目标版本为 ES2020
      // 这样可以确保依赖项被编译成 ES2020 兼容的代码，适用于现代浏览器。
      target: 'es2020',
      // 2. 定义全局常量
      // 这些定义在代码中可以作为全局宏使用。比如 __DEV__ 可以在开发环境中使用，
      // 而 __VERSION__ 可以用来输出当前版本号。
      define: {
        __DEV__: 'true',                       // 定义是否处于开发环境
        __VERSION__: JSON.stringify('1.0.0')   // 定义当前应用版本号
      },
      // 3. 自定义 esbuild 插件
      // 通过 esbuild 插件，可以实现对模块的自定义解析行为，比如这里的例子替换了
      // 'env' 模块路径为 'env.js'。
      plugins: [
        {
          name: 'custom-plugin', // 插件名称
          setup(build) {
            // 自定义模块解析规则
            // 当导入的模块是 'env' 时，将其路径重定向为 'env.js'
            build.onResolve({ filter: /^env$/ }, args => {
              return { path: path.resolve(__dirname, 'env.js') };
            });
          }
        }
      ],
      // 4. 启用源码映射
      // 在开发中启用 sourcemap，这样在浏览器调试时可以追踪到原始源码的位置。
      sourcemap: true,
      // 5. 自定义 JSX 编译
      // 如果使用 JSX，可以自定义 JSX 的工厂函数和片段。
      // 例如，在使用 Preact 时可能需要这样配置。
      jsxFactory: 'h',             // 将 JSX 编译为 'h' 函数 (如 Preact)
      jsxFragment: 'Fragment'       // JSX 片段编译为 'Fragment'
    },
    force: true, // 强制预构建,忽略缓存
  },

  // 构建后预览配置，适用于本地预览生产构建结果
  // 使用 vite preview 命令启动,用于快速预览生产构建的结果，并进行最后的测试或展示
  preview: {
    host: '0.0.0.0',
    port: 5000, // 预览服务器的端口，默认为 5000
    strictPort: true, // 如果端口不可用则直接退出
    https: false, // 是否启用 HTTPS
    open: false, // 是否自动打开浏览器
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
    },
    cors: true,
    headers: {}
  },
  
  // 配置生产构建的相关选项
  build: {
    target: ['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14'], // 构建兼容目标，支持 es2020 或更高，具体视浏览器支持情况而定
    modulePreload: { polyfill: true }, // 默认情况下，一个 模块预加载 polyfill 会被自动注入
    outDir: 'dist', // 打包后文件输出目录，默认为 'dist'
    assetsDir: 'assets', // 静态资源文件夹名，默认为 'assets'   图片等静态资源
    assetsInlineLimit: 4096, // 小于此大小的静态资源(png等图片)转为内联 base64，单位字节，默认为 4096 (4KB)
    cssCodeSplit: true, // 启用/禁用 CSS 代码拆分，默认为 true
    cssTarget: ['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14'], // 默认值与 build.target 一致
    cssMinify: 'esbuild', // 默认值与 build.minify 一致, 参数: boolean | 'esbuild' | 'lightningcss'
    sourcemap: false, // 是否生成 sourcemap 文件，默认为 false，适合生产环境调试. 参数:boolean | 'inline' | 'hidden'
    rollupOptions: { // 自定义底层的 Rollup 打包配置
      input: {
        main: './index.html', // 构建入口文件
        nested: './src/nested.html', // 可以有多个入口文件
      },
      output: {
        // 输出文件配置
        dir: 'dist', // 输出目录
        entryFileNames: '[name].[hash].js', // 入口文件名格式
        chunkFileNames: '[name].[hash].js', // 非入口 chunk 的文件名格式
        assetFileNames: '[name].[contenthash].[ext]', // 静态资源文件名格式
        manualChunks(id) { // 此处进行代码分割,对 node_modules 中的模块进行分割
          if (id.includes('node_modules')) {
            return 'vendor'; // 手动将 node_modules 中的代码打包到 vendor 文件中
          }
        },
      },
      // 摇树 默认true, 接受 true/false/对象 参数
      treeshake: {
          // 比如：告诉 Rollup 对象属性读取不产生副作用
          propertyReadSideEffects: false,
      }
    },
    commonjsOptions:{}, // 传递给 @rollup/plugin-commonjs 插件的选项
    dynamicImportVarsOptions:{}, // 传递给 @rollup/plugin-dynamic-import-vars 的选项
    lib: {}, // 构建为库的配置项,不为正常项目
    manifest: false, // 是否生成包含了没有被 hash 过的资源文件名和 hash 后版本的映射 manifest.json 文件
    ssrManifest: false, // 是否生成 SSR 的 manifest 文件，以确定生产中的样式链接与资产预加载指令,当该值为一个字符串时，它将作为 manifest 文件的名字
    ssr: false, // 生成面向 SSR 的构建。此选项的值可以是字符串，用于直接定义 SSR 的入口，也可以为 true，但这需要通过设置 rollupOptions.input 来指定 SSR 的入口。
    emitAssets: false, // 在非客户端的构建过程中，静态资源并不会被输出，因为我们默认它们会作为客户端构建的一部分被输出, 开启可强制输出这些资源。
    ssrEmitAssets: false, // 在 SSR 构建期间，静态资源不会被输出，因为它们通常被认为是客户端构建的一部分, 开启可强制输出这些资源。
    minify: 'terser', // 压缩方式，可选 'esbuild' (默认) | 'terser' | true
    terserOptions: {
        // 例如：去除 console 和 debugger
        drop_console: true,
        drop_debugger: true,
    }, // 传递给 Terser 的更多 minify 选项
    write: true, // 设置为 false 来禁用将构建后的文件写入磁盘。这常用于 编程式地调用 build() 在写入磁盘之前，需要对构建后的文件进行进一步处理。
    emptyOutDir: true, // 打包前是否清空输出目录，默认为 true
    copyPublicDir: true, // 默认情况下，Vite 会在构建阶段将 publicDir 目录中的所有文件复制到 outDir 目录中。可以通过设置该选项为 false 来禁用该行为。
    reportCompressedSize: true, // 启用/禁用 gzip 压缩大小报告。压缩大型输出文件可能会很慢，因此禁用该功能可能会提高大型项目的构建性能。
    chunkSizeWarningLimit: 500, // 触发输出警告的 chunk 文件大小（KB），默认为 500KB
    watch: null, // 设置为 {} 则会启用 rollup 的监听器。对于只在构建阶段或者集成流程使用的插件很常用
  },


    // 存储缓存文件的目录, 以下为默认值
    cacheDir: 'node_modules/.vite',

    // 用于加载 .env 文件的目录,可以是一个绝对路径，也可以是相对于项目根的路径
    // 假设你的环境变量文件放在 config/env/ 目录下,值就为'./config/env'
    // 值为 'root' 时,直接使用配置中 root 的内容
    envDir: 'root',

    // 以 envPrefix 开头的环境变量会通过 import.meta.env 暴露在你的客户端源码中
    // envPrefix 不应被设置为空字符串 ''，这将暴露你所有的环境变量，导致敏感信息的意外泄漏
    /* 想暴露一个不含前缀的变量，可以使用 define 选项
    * define: {
      'import.meta.env.ENV_VARIABLE': JSON.stringify(process.env.ENV_VARIABLE)
      }
    * */
    envPrefix:'VITE_',

    // JSON 相关配置
    json: {
      namedExports: true, // 支持从 JSON 文件中导出按名导入，默认 true
      stringify: false, // 若设置为 true，导入的 JSON 会被转换为 export default JSON.parse("..."),并禁止namedExports
    },
      
    // 调整 Vite 的日志输出级别
    logLevel: 'info', // 日志级别：'info' | 'warn' | 'error' | 'silent'

    // 使用自定义 logger 处理各种消息
    customLogger: logger,

    // 设为 false 可以避免 Vite 清屏而错过在终端中打印某些关键信息。
    clearScreen: true,
  
    ssr:{ // ssr 环境配置
      external: true, // 指定的依赖项和它们传递的依赖项进行外部化，以供服务端渲染（SSR）使用.
      noExternal: [], // 防止列出的依赖项在服务端渲染（SSR）时被外部化，这些依赖项将会在构建过程中被打包. 设为true则所有依赖都不会外部化
      target: 'node', // 'node' | 'webworker' , SSR 服务器的构建目标
      resolve: {
        conditions:['import','module','browser','default', 'production', 'development'], // ssr 导入内部化包入口的解析条件
        externalConditions:['import','module','browser','default', 'production', 'development'] // ssr 导入外部化包入口的解析条件
      },
    },
  
    // Web Worker 配置
    worker: {
      format: 'iife', // worker类型，默认为 'iife' (可选 'es')
      plugins: [], //  worker 打包使用的 Vite 插件
      rollupOptions: {} // worker 打包使用的 Rollup 配置项
    },
});

```

## 常用配置
1. 复制静态资源(图片等不需要转换的文件): publicDir / assetsInclude / build.assetsDir / build.copyPublicDir
2. 图片转base64: build.assetsInlineLimit
3. 配置路径别名: resolve.alias
4. 配置环境变量: define
5. 配置sourceMap: esbuild.sourcemap(开发环境) / build.sourcemap(生产环境)
6. 配置插件: plugins
7. gzip / brotli 压缩: vite-plugin-compression
8. 模块可视化插件: rollup-plugin-visualizer
9. 配置导出的 HTML 内容: html
10. 配置css预处理器: css.preprocessorOptions
11. css单独打包: build.cssCodeSplit
12. 配置服务器: server
13. 配置热模块替换: server.hmr
14. 配置 esbuild: esbuild
15. 配置依赖预构建: optimizeDeps
16. 配置构建项目: build
17. js兼容编译: build.target
18. 代码分割: build.rollupOptions.manualChunks
19. 配置代码压缩: build.minify / build.terserOptions
20. 配置摇树: build.rollupOptions.treeshake
21. 配置输出文件名/缓存(contentHash): build.rollupOptions.output.entryFileNames
22. 配置缓存: cacheDir
23. 配置 SSR: ssr
24. 配置 Web Worker: worker
25. 配置日志: logLevel / customLogger / clearScreen

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

### 11. **`rollup-plugin-visualizer`**
- **用途**: 类似 webpack-bundle-analyzer ,生成可视化的打包分析报告，帮助优化项目的打包体积。
- **功能**: 生成交互式的打包分析图表，展示项目中各个模块的大小和依赖关系。

```bash
# 安装插件
npm install rollup-plugin-visualizer --save-dev
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import visualizer from 'rollup-plugin-visualizer';

// 定义 Vite 配置
export default defineConfig({
  plugins: [
    // 添加可视化打包分析插件
    visualizer(),
  ],
});
```
