# Vite 8.x 概述

## Vite 8 新特性

### 重大更新

1. **Rolldown 打包器（基于 Rust）**
   - Vite 8 默认使用 Rolldown 作为生产环境打包器
   - Rolldown 是基于 Rust 编写的高性能打包器，与 Rollup API 兼容
   - 构建速度显著提升，大型项目构建时间可减少 50% 以上
   - 开发和生产环境构建一致性大幅改善

2. **Environment API（稳定版）**
   - 支持多环境构建（客户端、SSR、Edge 等）
   - 每个环境可以有独立的配置和优化策略
   - 框架开发者可以更灵活地控制不同环境的行为

3. **Node.js 版本要求**
   - 最低要求 Node.js 22.x 或 20.19+
   - 不再支持 Node.js 18

4. **默认浏览器兼容性目标**
   - 默认目标更改为 "Baseline Widely Available"
   - 确保支持主流浏览器中已广泛可用的 Web 平台特性

5. **Module Runner API**
   - 替代原有的 `ssrLoadModule`
   - 提供更强大的模块执行能力

### 配置变更

```js
// Vite 8 新增/变更的配置
export default defineConfig({
  // Environment API 配置
  environments: {
    client: {
      build: {
        outDir: 'dist/client',
      },
    },
    ssr: {
      build: {
        outDir: 'dist/server',
      },
    },
  },
  
  // 新的默认值
  json: {
    stringify: true, // Vite 8 默认为 true
  },
})
```

---

## Vite 与 webpack 优缺点

| 特性             | Vite 8                                      | Webpack               |
|----------------|---------------------------------------------|-----------------------|
| **开发启动速度**     | 极快，使用原生 ES 模块（ESM）和 `esbuild` 预构建依赖，按需编译 | 较慢，需要先打包整个项目再启动       |
| **热更新速度**      | 极快，基于原生 ESM，按需更新实际变化的模块                    | 较慢，尤其在大型项目中，重新打包整个模块 |
| **打包速度**       | 生产模式使用 Rolldown（Rust），速度极快                 | 灵活扩展性强，支持复杂场景，但较慢     |
| **生态系统**       | 成熟，兼容 Rollup 插件生态                          | 成熟，插件丰富，支持复杂场景        |
| **配置复杂度**      | 简单，开箱即用，Environment API 增强扩展性              | 配置复杂，尤其是在复杂项目中，扩展性强  |
| **浏览器支持**      | 现代浏览器优先，需插件支持旧浏览器                           | 广泛支持新旧版浏览器            |
| **开发与生产构建一致性** | Rolldown 统一构建，一致性显著改善                       | 构建一致性是稳定的             |
| **适用场景**       | 适合各种规模项目和现代框架开发                             | 适合大型项目和复杂构建场景         |

## 配置示例

### defineConfig 函数参数

> 基础常用配置选项,不需要深入就看这块就好

```js
// ==================== 依赖导入 ====================
import { defineConfig } from 'vite'; // Vite 配置定义函数
import vue from '@vitejs/plugin-vue'; // Vue 3 单文件组件支持
import tailwindcss from '@tailwindcss/vite'; // Tailwind CSS 原子化样式框架
import UnpluggingAutoImport from 'unplugin-auto-import/vite'; // 自动导入 API，无需手动 import
import Components from 'unplugin-vue-components/vite'; // Vue 组件自动注册
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'; // Ant Design Vue 组件解析器
import { resolve } from 'path'; // 路径解析方法
import { VantResolver } from '@vant/auto-import-resolver'; // Vant 移动端组件解析器
import vueJsxPlugin from '@vitejs/plugin-vue-jsx'; // Vue JSX 语法支持
import legacy from '@vitejs/plugin-legacy'; // 旧版浏览器兼容插件，生成 polyfill
import { visualizer } from 'rollup-plugin-visualizer'; // 构建产物可视化分析工具
import viteCompression from 'vite-plugin-compression'; // 文件压缩插件（gzip/brotli）

// ==================== Vite 主配置 ====================
export default defineConfig(async ({ command }) => {
   // 判断是否为调试构建（保留 console、sourcemap 等）
   const isDebugBuild = process.env.DEBUG_BUILD === 'true';

   // ==================== 服务器默认配置 ====================
   /**
    * 服务器构建的默认配置
    * 包含 API 基础地址、WebSocket 地址和开发服务器配置
    */
   let serverConfig = {
      baseUrl: 'http://pc.cc/', // http API 地址
      marketWsUrl: 'wss://pc.cc', // WebSocket 地址
      dev: {
         port: 9966, // 开发服务器端口
         host: true, // 允许外部访问（局域网调试）
         open: true, // 启动时自动打开浏览器
      },
   };

   // 优先使用品牌配置，否则回退到服务器配置
   const config = serverConfig;

   // ==================== 插件配置 ====================
   /**
    * Vite 插件数组
    * 插件按顺序执行，某些插件有执行顺序要求
    */
   const plugins = [
      // Vue 3 单文件组件（.vue）编译支持
      vue(),

      // Vue JSX/TSX 语法支持
      // 允许在 Vue 组件中使用 JSX 语法
      vueJsxPlugin(),

      // Tailwind CSS 插件
      // 提供原子化 CSS 类名支持
      tailwindcss(),

      // 自动导入插件
      // 自动导入 Vue、Vue Router 等 API，无需手动 import
      UnpluggingAutoImport({
         dts: true, // 生成 TypeScript 类型声明文件
         imports: [
            'vue', // Vue 3 组合式 API（ref, reactive, computed 等）
            'vue-router', // 路由相关 API（useRouter, useRoute 等）
            'vue-i18n', // 国际化 API（useI18n 等）
            'pinia', // 状态管理 API（defineStore 等）
            '@vueuse/core', // VueUse 工具函数库
         ],
         resolvers: [
            AntDesignVueResolver(), // Ant Design Vue 组件 API
            VantResolver(), // Vant 组件 API
         ],
         // 自动扫描并导入以下目录中的模块
         dirs: [
            './src/apis', // API 接口
            './src/config', // 配置文件
            './src/utils', // 工具函数
            './src/store', // Pinia 状态仓库
            './src/hooks', // 组合式函数
            './src/i18n', // 国际化
            './src/components', // 公共组件
         ],
      }),

      // 组件自动注册插件
      // 自动注册组件，无需手动 import 和注册
      Components({
         resolvers: [
            // Ant Design Vue 组件解析器
            AntDesignVueResolver({
               importStyle: false, // 不自动导入样式（使用全局样式）
            }),
            // Vant 移动端组件解析器
            VantResolver({
               importStyle: true, // 自动导入组件样式
            }),
         ],
      }),

      /**
       * Rollup 可视化分析插件
       * 仅在设置环境变量 ENABLE_VISUALIZER=true 时启用
       * 生成交互式的构建产物分析报告
       */
      visualizer({
         open: true, // 构建完成后自动打开分析报告
         gzipSize: true, // 显示 gzip 压缩后的大小
         brotliSize: true, // 显示 brotli 压缩后的大小
         filename: 'dist/stats.html', // 分析报告输出路径
      }),
   ];

   // ==================== 压缩插件（服务器构建）====================
   /**
    * 文件压缩插件
    * 生成 .gz 和 .br 压缩文件，配合 nginx 提升传输效率
    */
   if (command === 'build') {
      plugins.push(
              // Gzip 压缩（通用性好，所有浏览器支持）
              viteCompression({
                 algorithm: 'gzip', // 压缩算法
                 ext: '.gz', // 压缩文件扩展名
                 threshold: 1024, // 压缩阈值：只压缩大于 1KB 的文件
                 deleteOriginFile: false, // 保留原文件（nginx 可以选择性提供）
                 verbose: true, // 在控制台显示压缩信息
              }),
              // Brotli 压缩（压缩率更高，现代浏览器支持）
              viteCompression({
                 algorithm: 'brotliCompress', // Brotli 压缩算法
                 ext: '.br', // 压缩文件扩展名
                 threshold: 1024, // 压缩阈值：只压缩大于 1KB 的文件
                 deleteOriginFile: false, // 保留原文件
                 verbose: true, // 显示压缩信息
              }),
      );
   }

   // ==================== 旧版浏览器兼容插件（服务器构建）====================
   /**
    * Legacy 插件
    * 仅在服务器构建时启用（Web 用户可能使用旧版浏览器）
    * 注意：此插件会显著增加打包体积（生成 polyfill 和 legacy 代码块）
    */
   if (command === 'build') {
      plugins.unshift(
              // unshift: 添加到插件列表最前面
              legacy({
                 targets: ['chrome >= 64', 'safari >= 12'], // 目标浏览器版本
                 modernPolyfills: true, // 为现代浏览器添加必要的 polyfill
                 // 明确指定目标浏览器，避免覆盖 build.target 的警告
                 renderLegacyChunks: true, // 生成旧版代码块
              }),
      );
   }

   // ==================== 返回 Vite 配置对象 ====================
   return {
      // ==================== 基础路径 ====================
      /**
       * 公共基础路径
       * Web: 使用绝对路径 '/'，从服务器根目录加载
       */
      base: '/',

      // ==================== 全局常量定义 ====================
      /**
       * 定义在编译时替换的全局常量
       * 这些值会在构建时被静态替换
       */
      define: {
         'import.meta.env.VITE_SKIN': JSON.stringify(process.env.VITE_SKIN), // 皮肤/主题
         'import.meta.env.VITE_API_BASE_URL': JSON.stringify(config.baseUrl), // API 基础地址
         'import.meta.env.VITE_MARKET_WS_URL': JSON.stringify(config.marketWsUrl), // WebSocket 地址
         'process.env.DEBUG_BUILD': JSON.stringify(process.env.DEBUG_BUILD || 'false'), // 调试模式标识
      },

      // ==================== 开发服务器配置 ====================
      /**
       * 开发服务器配置
       * 用于本地开发时的热更新、代理等
       */
      server: {
         open: config.dev?.open ?? serverConfig.dev.open, // 启动时是否自动打开浏览器
         host: config.dev?.host ?? serverConfig.dev.host, // 监听地址（true 表示监听所有地址）
         port: config.dev?.port ?? serverConfig.dev.port, // 监听端口
         // 代理配置：解决开发时的跨域问题
         proxy: {
            // API 请求代理
            '/api': {
               target: config.baseUrl, // 代理目标地址
               changeOrigin: true, // 修改请求头中的 Origin
            },
            // WebSocket 代理
            '/ws': {
               target: config.marketWsUrl,
               changeOrigin: true,
               ws: true, // 启用 WebSocket 代理
            },
         },
      },

      // ==================== 构建优化配置 ====================
      build: {
         // 输出目录
         outDir: 'dist',
         // 静态资源目录（相对于 outDir）
         assetsDir: 'assets',

         /**
          * 资源内联阈值（字节）
          * 小于此阈值的资源会被内联为 base64
          * 默认 4KB，可减少 HTTP 请求数
          */
         assetsInlineLimit: 4096,

         /**
          * Source Map 配置
          * 调试模式：生成内联 sourcemap 便于调试
          * 生产模式：关闭 sourcemap（之前的 sourcemap 文件很大）
          */
         sourcemap: isDebugBuild ? 'inline' : false,

         /**
          * 构建目标
          * 指定最终代码需要兼容的浏览器/运行时版本
          * 服务器构建：由 legacy 插件控制（需要兼容旧浏览器）
          */
         target: undefined,

         /**
          * 代码压缩配置
          * 调试模式：不压缩，便于阅读调试
          * 生产模式：使用 terser 压缩（比 esbuild 压缩率更高）
          */
         minify: isDebugBuild ? false : 'terser',

         /**
          * Terser 压缩选项
          * 调试模式：不配置（不压缩）
          * 生产模式：精细控制压缩行为
          */
         terserOptions: isDebugBuild
                 ? undefined
                 : {
                    compress: {
                       /**
                        * Console 处理策略
                        * 不直接 drop_console，而是通过 pure_funcs 选择性移除
                        * 保留 console.error 和 console.warn 用于生产环境错误追踪
                        */
                       drop_console: false,
                       drop_debugger: true, // 移除 debugger 语句
                       passes: 2, // 压缩遍数（更多遍数=更好压缩，但更慢）
                       /**
                        * 纯函数列表
                        * 这些函数调用会被视为无副作用而移除
                        * 保留 error/warn 用于错误追踪
                        */
                       pure_funcs: [
                          'console.log', // 移除调试日志
                          'console.info', // 移除信息日志
                          'console.debug', // 移除调试日志
                          // 保留 console.error 和 console.warn 用于生产环境错误追踪
                       ],
                       dead_code: true, // 移除不可达代码
                       unused: true, // 移除未使用的变量
                    },
                    mangle: {
                       properties: false, // 不混淆属性名（避免破坏某些依赖属性名的库）
                    },
                    format: {
                       comments: false, // 移除所有注释
                    },
                 },

         /**
          * CSS 代码分割
          * true: 将 CSS 提取到单独的文件中
          * 大型 CSS 会被分割为独立的 chunk
          */
         cssCodeSplit: true,

         /**
          * Rollup 打包配置
          * 用于细粒度控制代码分割策略
          */
         rollupOptions: {
            output: {
               /**
                * 手动代码分割
                * 将第三方库按类型分割到不同的 chunk
                * 好处：
                * 1. 缓存优化：库代码变化少，可长期缓存
                * 2. 按需加载：大型库可以懒加载
                * 3. 并行加载：多个小 chunk 可并行下载
                */
               manualChunks(id) {
                  // 警告: src 目录代码一拆就炸裂，慎改
                  // 也不能拆 Vue 相关的库，交由 Vite 处理

                  // 第三方库打包策略

                  // 1. UI 框架（Ant Design Vue + Vant）
                  // 这两个库体积较大，单独分包
                  if (id.includes('ant-design-vue') || id.includes('vant')) {
                     return 'ui';
                  }

                  // 2. 图表库（独立分包）
                  // ECharts 体积很大，必须单独分包
                  if (id.includes('echarts')) {
                     return 'chart';
                  }

                  // 3. 工具类库
                  // 这些是常用的工具函数库，合并打包
                  if (
                          id.includes('lodash-es') || // 工具函数
                          id.includes('dayjs') || // 日期处理
                          id.includes('crypto-js') || // 加密
                          id.includes('qs') || // URL 参数序列化
                          id.includes('@vueuse') // Vue 组合式工具函数
                  ) {
                     return 'utils';
                  }

                  // 4. 其他第三方库
                  // 较小的第三方库合并打包
                  if (
                          id.includes('swiper') || // 轮播组件
                          id.includes('aos') || // 滚动动画
                          id.includes('axios') || // HTTP 客户端
                          id.includes('mitt') || // 事件总线
                          id.includes('currency.js') || // 货币格式化
                          id.includes('qrcode') // 二维码生成
                  ) {
                     return 'others';
                  }

                  // 5. 默认 vendor（最终兜底）
                  // 其他所有 node_modules 中的库
                  if (id.includes('node_modules')) {
                     return 'vendor';
                  }
               },
               /**
                * 实验性最小 chunk 大小
                * 尝试合并小于 50KB 的业务代码 chunk
                * 避免产生过多碎片文件，减少 HTTP 请求
                * 注意：此选项不影响 manualChunks 中定义的第三方库
                */
               experimentalMinChunkSize: 50 * 1024,
            },
         },
      },

      // ==================== 开发依赖预构建优化 ====================
      /**
       * 依赖预构建配置
       * 仅影响开发服务器，不影响生产构建
       * Vite 会预先将 CommonJS/UMD 依赖转换为 ESM
       */
      optimizeDeps: {
         /**
          * 排除预构建的依赖
          * 这些大型库不会在开发时预构建，而是按需加载
          * 可以加快开发服务器启动速度
          */
         exclude: [
            'echarts', // 图表库（体积大，按需加载）
            'vant', // 移动端 UI 库（如果按需引入，可排除）
         ],
         /**
          * 强制预构建的依赖
          * 这些核心依赖会被预先构建，确保开发时快速加载
          */
         include: [
            'vue', // Vue 核心
            'vue-router', // 路由
            'pinia', // 状态管理
            '@vueuse/core', // 工具函数
            'ant-design-vue',
         ],
         /**
          * Rolldown 配置选项（Vite 8 使用 Rolldown 替代 esbuild）
          * 用于控制预构建时的代码转换行为
          */
         rolldownOptions: {
            // 确保 CommonJS 模块被正确转换为 ES 模块
            format: 'esm',
         },
      },

      // 插件列表
      plugins,

      // ==================== 路径解析配置 ====================
      resolve: {
         /**
          * 路径别名配置
          * 允许使用 @ 符号代替 src 目录
          * 例如：import xxx from '@/components/xxx'
          */
         alias: {
            '@': resolve(__dirname, 'src'), // @ 指向 src 目录
         },
      },
   };
});


```

### defineConfig 对象参数

> 完整配置选项

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
    appType: 'spa', // 默认 'spa'(单页面文件) | 'mpa'(包含 HTML 中间件) | 'custom'(ssr | 完全自定义 HTML 模板)

    // 处理特定类型的静态资源,直接使用不会被插件转换
    assetsInclude: ['**/*.gltf', /\.txt$/, /\.md$/, /\.csv$/, /\.png$/],

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
        conditions: ['import', 'module', 'browser', 'default', 'production', 'development'], // 一个依赖提供多个版本时,使用conditions根据不同环境使用不同版本的模块,按照顺序进行优先加载
        mainFields: ['browser', 'module', 'jsnext:main', 'jsnext'], // package.json 中，在解析依赖时,优先解析的顺序 browser(浏览器) module(node.js)
        extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'], // 导入时想要省略的扩展名列表,加载文件忽略后缀名
        preserveSymlinks: false, // 模块解析符号链接，将符号链接解析为它们指向的实际路径.
        // 为true时保留模块符号链接, 主要作用于 Monorepo 环境下，多个包可能通过符号链接进行解析, 而不是使用实际路径.
    },

    html: { // 定制入口 HTML 文件的处理，比如注入环境变量、动态插入标签等
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
                modifyVars: {'@primary-color': '#1DA57A'}, // 自定义全局样式变量
            },
            stylus: { // 仅支持 define，可以作为对象传递
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
        warmup: {
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
        include: [/\.tsx/, /\.jsx/, /\.ts/], // 只编译这些内容
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
        // Vite 8: esbuildOptions 已废弃，改用 rolldownOptions（Rolldown 替代 esbuild 进行依赖优化）
        rolldownOptions: { // 开发环境传递给 Rolldown 的选项
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
            // 3. 自定义 Rolldown 插件
            // 通过插件可以实现对模块的自定义解析行为
            plugins: [
                {
                    name: 'custom-plugin', // 插件名称
                    resolveId(source) {
                        // 自定义模块解析规则
                        // 当导入的模块是 'env' 时，将其路径重定向为 'env.js'
                        if (source === 'env') {
                            return path.resolve(__dirname, 'env.js');
                        }
                    }
                }
            ],
            // 4. 启用源码映射
            // 在开发中启用 sourcemap，这样在浏览器调试时可以追踪到原始源码的位置。
            sourcemap: true,
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
        target: ['es2020', 'edge111', 'firefox114', 'chrome111', 'safari16.4'], // Vite 8 默认使用 Baseline Widely Available 目标
        modulePreload: {polyfill: true}, // 默认情况下，一个 模块预加载 polyfill 会被自动注入
        outDir: 'dist', // 打包后文件输出目录，默认为 'dist'
        assetsDir: 'assets', // 静态资源文件夹名，默认为 'assets'   图片等静态资源
        assetsInlineLimit: 4096, // 小于此大小的静态资源(png等图片)转为内联 base64，单位字节，默认为 4096 (4KB)
        cssCodeSplit: true, // 启用/禁用 CSS 代码拆分，默认为 true
        cssTarget: ['es2020', 'edge111', 'firefox114', 'chrome111', 'safari16.4'], // 默认值与 build.target 一致
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
        commonjsOptions: {}, // 传递给 @rollup/plugin-commonjs 插件的选项
        dynamicImportVarsOptions: {}, // 传递给 @rollup/plugin-dynamic-import-vars 的选项
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
      stringify: true, // Vite 8 默认为 true，导入的 JSON 会被转换为 export default JSON.parse("...")，如需按名导入设为 false
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

---

## 从旧版本迁移到 Vite 8

### 从 Vite 7 迁移

1. **Node.js 版本升级**
   ```bash
   # 确保 Node.js 版本 >= 22.x 或 20.19+
   node -v
   ```

2. **更新依赖**
   ```bash
   npm install vite@latest
   ```

3. **配置调整**
   ```js
   // vite.config.js
   export default defineConfig({
     // json.stringify 默认值已改为 true
     // 如需保持旧行为，显式设置为 false
     json: {
       stringify: false,
     },
   })
   ```

4. **Rolldown 打包器**
   - Vite 8 默认使用 Rolldown，大部分 Rollup 插件仍然兼容
   - 如遇兼容性问题，检查插件是否支持 Rolldown

### 从 Vite 5/6 迁移

1. **Environment API 迁移**
   ```js
   // 旧版本
   export default defineConfig({
     ssr: {
       target: 'node',
     },
   })
   
   // Vite 8 推荐
   export default defineConfig({
     environments: {
       ssr: {
         build: {
           ssr: true,
         },
       },
     },
   })
   ```

2. **ssrLoadModule 迁移**
   ```js
   // 旧版本
   const module = await vite.ssrLoadModule('/src/entry-server.js')
   
   // Vite 8 使用 Module Runner
   const runner = createModuleRunner(environment)
   const module = await runner.import('/src/entry-server.js')
   ```

### 破坏性变更清单

| 变更项 | 旧行为 | 新行为 |
|--------|--------|--------|
| Node.js 要求 | >= 18 | >= 20.19 或 >= 22 |
| 默认打包器 | Rollup | Rolldown |
| 依赖优化配置 | `optimizeDeps.esbuildOptions` | `optimizeDeps.rolldownOptions` |
| `json.stringify` | 默认 false | 默认 true |
| SSR API | `ssrLoadModule` | Module Runner API |
| 浏览器目标 | 自定义默认值 | Baseline Widely Available (Chrome 111+, Firefox 114+, Safari 16.4+) |
