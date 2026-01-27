## Webpack 5

### 1\. 概述 
+   `webpack` 是一个模块打包工具，你可以使用webpack管理你的**模块依赖**，并**编绎输出**模块们所需的静态文件。

#### Webpack5 重要更新

| 特性 | 说明 |
| --- | --- |
| **持久化缓存** | `cache: { type: 'filesystem' }` 内置文件缓存，大幅提升二次构建速度，替代 `hard-source-webpack-plugin` |
| **Asset Modules** | 内置资源模块，替代 `url-loader`、`file-loader`、`raw-loader` |
| **模块联邦** | `ModuleFederationPlugin` 支持跨应用共享模块 |
| **Tree Shaking 优化** | 支持嵌套 tree shaking、内部模块 tree shaking |
| **output.clean** | 内置清理输出目录，替代 `clean-webpack-plugin` |
| **fullhash** | `[hash]` 更名为 `[fullhash]`，`contenthash` 计算更精确 |
| **移除 Node.js Polyfill** | 不再自动引入 Node.js 核心模块的 polyfill |
| **默认压缩器** | 内置 `TerserPlugin`，无需额外安装 |
| **this.getOptions()** | Loader 中使用 `this.getOptions()` 替代 `loader-utils` 的 `getOptions` |
| **ESLint** | `eslint-loader` 废弃，使用 `eslint-webpack-plugin` |
| **devServer 配置** | `contentBase` → `static`，`clientLogLevel` → `client.logging` 等 |

### 2\. 核心打包原理实现

#### 2.1 打包流程概述

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  初始化阶段  │ -> │  编译阶段    │ -> │  构建阶段    │ -> │  生成阶段    │ -> │  输出阶段    │
│  (初始化参数  │    │ (创建编译器  │    │ (递归解析    │    │ (生成chunk  │    │ (写入文件   │
│   合并配置)  │    │  注册插件)   │    │  构建模块)   │    │  生成代码)   │    │  系统)      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

**1. 初始化阶段**
- 读取并合并配置参数：从 `webpack.config.js`、命令行参数、默认配置中合并得到最终配置
- 创建 `Compiler` 对象：实例化编译器，初始化文件系统、缓存等基础设施
- 加载所有插件：遍历 `plugins` 数组，调用每个插件的 `apply(compiler)` 方法注册钩子
- 触发钩子：`environment` → `afterEnvironment` → `initialize`

**2. 编译阶段**
- 触发 `beforeRun` 和 `run` 钩子，正式启动编译流程
- 创建 `Compilation` 对象：每次编译都会创建新的 Compilation，包含本次编译的所有信息
- 触发 `compile` → `thisCompilation` → `compilation` 钩子
- 确定入口：根据 `entry` 配置找到所有入口文件

**3. 构建阶段（make）**
- 从入口文件开始，调用 loader 对模块进行转换（如 babel-loader 转换 ES6+）
- 使用 `@babel/parser` 将代码解析为 AST（抽象语法树）
- 遍历 AST 找出该模块依赖的其他模块（`import`、`require`）
- 递归处理所有依赖模块，构建完整的**模块依赖图（Module Graph）**
- 触发钩子：`buildModule` → `succeedModule`（每个模块构建时）

**4. 生成阶段（seal）**
- 根据入口和模块依赖关系，将模块组装成 `Chunk`（代码块）
- 应用代码分割规则（`splitChunks`），生成多个 Chunk
- 对每个 Chunk 生成最终代码：添加 webpack runtime、模块包装函数
- 执行优化：Tree Shaking 标记、Scope Hoisting、代码压缩等
- 触发钩子：`seal` → `optimize` → `optimizeChunks` → `afterOptimizeChunks`

**5. 输出阶段（emit）**
- 触发 `emit` 钩子，此时可以最后修改输出内容
- 根据 `output` 配置，确定输出路径和文件名
- 将 Chunk 转换为文件，写入文件系统
- 触发 `afterEmit` → `done` 钩子，编译完成

#### 2.2 代码实现

> 完整的 webpack5 核心打包原理代码实现请查看：**[webpack-core-implement.md](./webpack-core-implement.md)**
>
> 包含以下内容：
> - 入口文件 (index.js) - webpack 函数实现
> - 编译器核心 (compiler.js) - Compiler 类完整实现
> - AST 解析器 (parser.js) - 依赖解析与代码转换
> - 使用示例 - 如何运行 mini-webpack

#### 2.3 核心概念总结

| 概念 | 说明 |
| --- | --- |
| **Compiler** | 编译器，webpack 的核心引擎，负责整个编译生命周期 |
| **Compilation** | 编译实例，每次文件变化都会创建新的 Compilation |
| **Module** | 模块，对应一个源文件，包含代码和依赖信息 |
| **Chunk** | 代码块，由多个模块组成，最终生成一个输出文件 |
| **Asset** | 输出资源，最终写入文件系统的内容 |
| **Tapable** | 钩子系统，实现插件机制的核心库 |
| **Loader** | 模块转换器，将非 JS 文件转换为 webpack 可处理的模块 |
| **Plugin** | 插件，通过钩子介入编译流程的各个阶段 |

### 3\. Loader

**作用**：Loader 是模块转换器，用于将非 JavaScript 文件（如 CSS、图片、TypeScript 等）转换为 webpack 能够处理的有效模块。

**特点**：
- 本质是一个函数，接收源文件内容，返回转换后的内容
- 执行顺序：`从右到左，从下到上`
- 支持链式调用，多个 loader 可以串联使用
- webpack5 使用 `this.getOptions()` 获取配置（替代 `loader-utils`）

**简单示例**：

```js
// 自定义 loader：将文件内容转为大写
module.exports = function(source) {
    return source.toUpperCase();
};

// webpack.config.js 中使用
module: {
    rules: [
        {
            test: /\.txt$/,
            use: ['./loaders/uppercase-loader.js']
        }
    ]
}
```

### 4\. Plugin

**作用**：Plugin 用于扩展 webpack 功能，可以介入编译流程的各个阶段，执行更广泛的任务（打包优化、资源管理、环境变量注入等）。

**特点**：
- 本质是一个具有 `apply` 方法的类
- 通过 Tapable 钩子系统注册到 webpack 生命周期
- 可以访问 `compiler` 和 `compilation` 对象

**简单示例**：

```js
// 自定义 plugin：打包完成后输出文件列表
class FileListPlugin {
    apply(compiler) {
        compiler.hooks.emit.tapAsync('FileListPlugin', (compilation, callback) => {
            let fileList = '## 打包文件列表\n\n';
            
            for (let filename in compilation.assets) {
                fileList += `- ${filename}\n`;
            }
            
            compilation.assets['fileList.md'] = {
                source: () => fileList,
                size: () => fileList.length
            };
            
            callback();
        });
    }
}

// webpack.config.js 中使用
plugins: [
    new FileListPlugin()
]
```

### 5\. 配置举例

```js
const { resolve } = require('path')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');
// webpack5 内置持久化缓存，不再需要 hard-source-webpack-plugin
const TerserWebpackPlugin = require('terser-webpack-plugin')
// webpack5 使用 css-minimizer-webpack-plugin 替代 optimize-css-assets-webpack-plugin
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
const CompressionWebpackPlugin = require('compression-webpack-plugin')
const ImageMinimizerWebpackPlugin = require('image-minimizer-webpack-plugin')
// webpack5 使用 eslint-webpack-plugin 替代 eslint-loader
const ESLintPlugin = require('eslint-webpack-plugin')
// webpack5 模块联邦
const { ModuleFederationPlugin } = require('webpack').container;
const zlib = require('zlib');

/*
  tree shaking：去除无用代码
    前提：1. 必须使用ES6模块化  2. 开启production环境 （就自动会把无用代码去掉）
    作用: 减少代码体积

    在package.json中配置
      "sideEffects": false 表示所有代码都没有副作用（都可以进行tree shaking）
        这样会导致的问题：可能会把css / @babel/polyfill （副作用）文件干掉
      所以可以配置："sideEffects": ["*.css", "*.less"] 不会对css/less文件tree shaking处理
*/
/**
 * webpack5 特性：
 * - tree shaking 优化
 * - 增加 cache 属性实现持久化缓存
 * - [hash] 更名为 [fullhash]
 * - 不再自动引入 Node.js 模块的 polyfill
 * - 内置 terser 进行代码压缩
 * - contenthash 计算更精确，添加注释和修改变量不会影响 contenthash 值
 * */


// 定义nodejs环境变量：决定使用browserslist的哪个环境
process.env.NODE_ENV = 'production'

module.exports = {
    entry: './src/index.js', // 可以是字符串地址、数组地址、对象地址
    /*
  entry: 入口起点
    1. string --> './src/index.js'
      单入口
      打包形成一个chunk。 输出一个bundle文件。
      此时chunk的名称默认是 main
    2. array  --> ['./src/index.js', './src/add.js']
      多入口
      所有入口文件最终只会形成一个chunk, 输出出去只有一个bundle文件。
        --> 只有在HMR功能中让html热更新生效~
    3. object
      多入口
      有几个入口文件就形成几个chunk，输出几个bundle文件
      此时chunk的名称是 key

      --> 特殊用法
        {
          // 所有入口文件最终只会形成一个chunk, 输出出去只有一个bundle文件。
          index: ['./src/index.js', './src/count.js'],
          // 形成一个chunk，输出一个bundle文件。
          add: './src/add.js'
        }
*/
    output: {
        // 使用contenthash配合cache-control能有效配合浏览器缓存
        // chunkhash hash 分别代表对应块hash与文件打包hash，chunkhash可能相同，
        // 而hash则是每次打包都不同，不管内容是否有变化
        // webpack5: [hash] 已更名为 [fullhash]
        filename: 'js/build_[contenthash:10].js',
        path: resolve(__dirname, 'build'),
        // webpack5: 替代 clean-webpack-plugin，每次构建前清理输出目录
        clean: true,
        // 所有资源引入公共路径前缀 --> 'imgs/a.jpg' --> '/imgs/a.jpg'
        // 通过cross-env设置打包环境变量，根据变量打包时进行动态选择，各个配置项参数可以动态修改publicPath，实现不同项目使用不同的资源路径
        // package.json: scripts: { "dev": "cross-env NODE_ENV=development webpack-dev-server --config webpack.config.js", }
        publicPath: '/',
        chunkFilename: 'js/[name]_chunk_[contenthash:10].js', // 非入口chunk的名称
        // webpack5: library 配置方式更新
        library: {
            name: '[name]',
            type: 'window' // 替代 libraryTarget，可选值: 'var', 'module', 'assign', 'this', 'window', 'self', 'global', 'commonjs', 'umd' 等
        }
    },
    module: {
        // 防止 webpack 解析那些任何与给定正则表达式相匹配的文件。
        // 忽略的文件中不应该含有import,require,define的调用，或任何其他导入机制。
        // 忽略大型的 library 可以提高构建性能。
        // externals共用会导致externals失效，因为会不转换require(),导致文件无法读取
        noParse:/jquery/,
        rules: [
            {
                test: /\.less$/,
                use: [
                    // 'style-loader', // 以style标签行内样式的方式,插入head标签中
                    MiniCssExtractPlugin.loader, // 打包成css单文件
                    'css-loader', // 加载读取css文件内容
                    {
                        // 还需要在 package.json 中定义 browserslist
                        // 为css样式添加兼容前缀
                        // webpack5 postcss-loader 配置方式更新
                        loader: 'postcss-loader',
                        options: {
                            postcssOptions: {
                                plugins: [
                                    'postcss-preset-env'
                                ]
                            }
                        }
                    },
                    // 处理less文件,转换为css文件
                    'less-loader'
                ]
            },
            {
                oneOf:[
                    {
                        test: /\.js$/,
                        // webpack5 内置缓存，不再需要 cache-loader
                        use: [
                            'thread-loader', // 多线程编译
                            {
                                loader: 'babel-loader',
                                options: {
                                    // webpack5 babel-loader 内置缓存支持
                                    cacheDirectory: true,
                                    cacheCompression: false, // 关闭缓存压缩，提升构建速度
                                    presets: [
                                        ['@babel/preset-env', {
                                            useBuiltIns: 'usage', // 按需加载
                                            corejs: { version: 3 }, // 指定 core-js 版本
                                            targets: { // 指定向后兼容到什么版本
                                                chrome: '60',
                                                firefox: '50',
                                                safari: '10',
                                                ie: '9',
                                                edge: '17'
                                            }
                                        }]
                                    ]
                                }
                            }
                        ],
                        include: /src/,
                        exclude: /node_modules/ 
                    },
                    // 配置移至 plugins 中
                    {
                        // webpack5 Asset Modules 替代 url-loader
                        test: /\.(jpg|png|gif|jpeg|webp|svg)$/,
                        type: 'asset',
                        parser: {
                            dataUrlCondition: {
                                maxSize: 8 * 1024 // 小于 8kb 转为 base64
                            }
                        },
                        generator: {
                            filename: 'imgs/[name]_[contenthash:10][ext]'
                        }
                    },
                    {
                        test: /\.html$/,
                        loader: 'html-loader' // 将 HTML 导出为字符串。当编译器需要时，将压缩 HTML 字符串
                    },
                    {
                        // webpack5 Asset Modules 替代 file-loader
                        exclude: /\.(js|css|html|less|jpg|png|gif|jpeg|webp|svg)$/,
                        type: 'asset/resource',
                        generator: {
                            filename: 'media/[name]_[contenthash:10][ext]'
                        }
                    },
                    {
                        // webpack5 Asset Modules 替代 raw-loader
                        test: /\.txt$/i,
                        type: 'asset/source'
                    },
                ]
            }
        ]
    },
    plugins: [
        // 压缩 gzip / Brotli算法, 都需要配置nginx 对应模块开启对应压缩功能
        // Brotli更快，压缩更小，但是兼容性不如gzip
        new CompressionWebpackPlugin({
            algorithm: 'brotliCompress', // 使用 Brotli 算法
            test: /\.(js|css|html|svg)$/, // 仅压缩这些文件类型
            compressionOptions: {
                params: {
                    [zlib.constants.BROTLI_PARAM_QUALITY]: 11, // 最高压缩级别
                },
            },
            threshold: 10240, // 只压缩大于 10KB 的文件
            minRatio: 0.8, // 只有压缩比 < 0.8 才生成压缩文件
            filename: '[path][base].br', // 输出文件名格式
        }),
        new MiniCssExtractPlugin({ // 将css打包成单文件
            filename: 'css/built_[contenthash:10].css'
        }),
        new HtmlWebpackPlugin({ //  自动引入各个打包文件
            template: './src/index.html', // 使用模板
            minify: {
                collapseWhitespace: true, // 移除空格
                removeComments: true // 移除注释
            }
        }),
        // 将某个文件打包输出到build目录下，并在html中自动引入该资源
        new AddAssetHtmlWebpackPlugin({
            filepath: resolve(__dirname, 'dll/jquery.js')
        }),
        // 通过 cache 配置项实现，见文件底部 cache 配置
        
        // webpack5 eslint-webpack-plugin 替代 eslint-loader
        new ESLintPlugin({
            context: resolve(__dirname, 'src'),
            extensions: ['js', 'jsx', 'ts', 'tsx'],
            fix: true,
            cache: true,
            cacheLocation: resolve(__dirname, 'node_modules/.cache/eslint/')
        }),
        // webpack5
        // 定义引入的模块联邦插件设置
        new ModuleFederationPlugin({
            // 模块联邦名字，提供给其他模块使用
            name: 'app1',
            // 打包后的名称,以及外部访问的资源入口
            // 目录路径相对于 output.path
            // 定义用于存储模块联邦信息的文件的名称，默认为`remoteEntry.js`
            filename: 'App1RemoteEntry.js',
            // type 是变量的类型，分别有 'var'、'module'、'assign'、'assign-properties'、'this'、'window'、'self'、'global' 或 'commonjs'
            // var 变量将挂载在 window 对象上. module 以 ES6 模块的形式导出. assign 以 CommonJS 模块的形式导出.
            // name 是变量的名称。这将导致生成的远程引用代码中使用 remote_app 作为全局变量名称
            library: { type: "var", name: "remote_app" },
            // 定义远程模块的名称和加载方式的映射
            remotes: {
                /**
                 *  App2 引用其他应用模块的资源别名
                 *  app2 是 APP2 的模块联邦名字
                 *  http://localhost:3001 是 APP2 运行的地址
                 *  App2RemoteEntry.js 是 APP2 提供的外部访问的资源名字
                 *  可以访问到 APP2 通过 exposes 暴露给外部的资源
                 */
                App2: 'app2@http://localhost:3001/App2RemoteEntry.js',
                RemoteA: `RemoteA@${process.env.A_URL}/remoteEntry.js`,
                // 使用函数来动态定义远程模块的加载路径
                remoteApp: () => {
                    if (process.env.NODE_ENV === 'production') {
                        return 'remote_app@http://example.com/remoteEntry.js';
                    } else {
                        return 'remote_app_dev@http://localhost:3001/remoteEntry.js';
                    }
                },
                // 使用返回 Promise 的函数来异步加载远程模块，这对于需要进行异步加载和初始化的场景非常有用
                remote_App: () => import('remote_app@http://example.com/remoteEntry.js'),
            },
            // 暴露给外部的模块/文件夹/动态加载
            // 键表示其他模块中引用的路径，值表示当前模块中实际的模块路径
            exposes: {
                './Button': './src/components/Button',
                './Header': './src/components/Header',
                // 动态加载
                './DynamicComponent': () => {
                    if ('环境变量') {
                        return './src/components/DynamicComponent1';
                    } else {
                        return './src/components/DynamicComponent2';
                    }
                },
            },
            // 共享模块,避免重复加载都有用到的模块，如lodash
            // 优先用远程的依赖，如果远程版本不对或没有，再用本地版本
            // shared 依赖不能 tree sharking
            // 数组写法,设置的共享模块的配置项全是默认值
            shared: [
                'react',
                'react-dom'
            ],
            /** 对象写法
             * shared: {
             *     react: {
             *         // 定义共享模块的作用域，默认为 'default',共享模块在全局作用域内可见和访问
             *         // 其他自定义值,则是独立的局部作用域,避免命名冲突
             *         // 自定义 'test', 则加载时 import("test/共享模块名")
             *         shareScope: 'default',
             *
             *         // 如果设置为 true，则共享模块将被导入到当前模块中；如果设置为 false，则共享模块将不会被导入,需外部下载脚本，而是由远程模块提供.
             *         // 默认为 true, Webpack 将会将其打包到当前模块中，使其在运行时可用
             *         import: true,
             *
             *         // 指定共享模块是否应该被视为单例模块，默认为 false,为每个模块创建一个实例
             *         // 设置为true,共享模块将在所有使用它的模块中共享同一个实例. 如框架包 react
             *         singleton: true,
             *
             *         // 设置为true,表示共享模块将在主应用程序启动时立即加载（eager loading）
             *         // 默认值false, 表示共享模块将会按需加载（lazy loading），即在模块被使用时才会被加载
             *         eager: true,
             *
             *         // 加载共享模块的版本需要等于或大于定义的版本号要求,不符合则会警告报错，可以是一个字符串或对象，默认为 undefined
             *         requiredVersion: {
             *           react: '^16.0.0',
             *           'react-dom': '^16.0.0'
             *         },
             *
             *         // 锁定共享模块的版本号,只允许用对应版本否则警告报错，可以是一个字符串或对象，默认为 undefined
             *         version: {
             *           react: '16.13.1',
             *           'react-dom': '16.13.1'
             *         }
             *     }
             * }
             *
             * */
        }),
        // 定义app2导出设置
        // new ModuleFederationPlugin({
        //     // 模块联邦名字，提供给其他模块使用
        //     name: 'app2',
        //     // 提供给外部访问的资源入口
        //     filename: 'App2RemoteEntry.js',
        //     // 引用的外部资源列表
        //     remotes: {},
        //     // 暴露给外部的资源列表
        //     exposes: {
        //         /**
        //          *  ./Header 是让外部应用使用时基于这个路径拼接引用路径，如：App2/Header
        //          *  ./src/Header.js 是当前应用的要暴露给外部的资源模块路径
        //          */
        //         './Header': './src/Header.js',
        //     },
        //     // 共享模块，值当前被 exposes 的模块需要使用的共享模块，如lodash
        //     shared: {},
        // }),
],
    devServer: {
        port: 8080, // 端口
        hot: true, // 打开HMR 热模块更新
        compress: true, // gzip
        open: true, // 自动打开浏览器
        proxy: [
            {
                context: ['/api'],
                target: 'https://localhost:8080', // 请求访问地址
                changeOrigin: true, // 修改源访问
                pathRewrite: {
                    '^/api': '/api' // 替换请求地址
                }
            }
        ],
        /*跨域问题：同源策略中不同的协议、端口号、域名就会产生跨域。正常的浏览器和服务器之间有跨域，但是服务器之间没有跨域。
    代码通过代理服务器运行，所以浏览器和代理服务器之间没有跨域，浏览器把请求发送到代理服务器上，代理服务器替你转发到另外一个服务器上
    服务器之间没有跨域，所以请求成功。代理服务器再把接收到的响应响应给浏览器。这样就解决开发环境下的跨域问题
    */
        // webpack5 devServer 配置更新
        client: {
            logging: 'none', // 替代 clientLogLevel
            overlay: false, // 如果出错了，不要全屏提示
            progress: true, // 显示编译进度
        },
        // webpack5: contentBase 改为 static
        static: {
            directory: resolve(__dirname, 'build'),
            watch: true // 替代 watchContentBase
        },
        // webpack5: watchOptions 移到顶层或 static.watch 配置
        watchFiles: {
            paths: ['src/**/*'],
            options: {
                ignored: /node_modules/
            }
        },
    },
    devtool: 'eval-source-map',
    mode: 'production', // development
    // 代码分割，会把node_modules中的文件打包到一起，如果是多入口则，
    // 会将公共引用打包到一个文件，共同引用。
    // externals防止将某些 import 的包(package)打包到 bundle 中，
    // 而是在运行时(runtime)再去从外部获取这些扩展依赖(external dependencies)
    externals: {
        // 拒绝jQuery被打包进来(通过cdn引入，速度会快一些)
        // 忽略import的库名 -- 导出的全局变量名称
        jquery: 'jQuery'
    },
    // 解析模块的规则
    resolve: {
        // 配置解析模块路径别名: 优点：当目录层级很复杂时，简写路径；缺点：路径不会提示
        alias: {
            $css: resolve(__dirname, 'src/css')
        },
        // 配置省略文件路径的后缀名（引入时就可以不写文件后缀名了）*-
        extensions: ['.js', '.json', '.jsx', '.css'],
        // 告诉 webpack 解析模块应该去找哪个目录
        modules: [resolve(__dirname, '../../node_modules'), 'node_modules'],
        // mainFields用来告诉webpack使用第三方模块中的哪个字段来导入模块；
        // 第三方模块中都会有package.json文件用来描述模块，有多个特殊环境字段用来告诉webpack导入文件的位置
        // 默认["browser", "module", "main"]，node环境是["module", "main"]
        // 设置为单个可减少字段搜索
        mainFields: ["main"]
    },
    optimization: {
        splitChunks: {
            // 代码分割时默认对异步代码生效，all：所有代码有效，initial：同步代码有效
            chunks: 'all',
            /* 以下都是默认值，可以不写
            minSize: 30 * 1024, // 分割的chunk最小为30kb（大于30kb的才分割）
            maxSize: 0, // 最大没有限制
            minChunks: 1, // 要提取的chunk最少被引用1次
            maxAsyncRequests: 5, // 按需加载时并行加载的文件的最大数量为5
            maxInitialRequests: 3, // 入口js文件最大并行请求数量
            automaticNameDelimiter: '~', // 名称连接符
            cacheGroups: { // 分割chunk的组
              vendors: {
                // node_modules中的文件会被打包到vendors组的chunk中，--> vendors~xxx.js
                // 满足上面的公共规则，大小超过30kb、至少被引用一次
                test: /[\\/]node_modules[\\/]/,
                // 优先级越大越先打包
                priority: -10
              },
              commons: {
                  name: 'chunk-commons',
                // 要提取的chunk最少被引用2次
                  minChunks: 2,
                  priority: 5,
                  chunks: 'initial',
                // 如果当前要打包的模块和之前已经被提取的模块是同一个，就会复用，而不是重新打包
                  reuseExistingChunk: true
                },
                // 当想要首屏优化需要剥离一些包的时候,可以通过test与优先级来匹配各个包,让他们独立分割出来
               indexchunk: {
                  test: /[\\/]node_modules[\\/]|各种包路径正则/,
                  name: 'index-chunk',
                  priority: 10,
                  chunks: 'all',
                  // 强制执行此方案,无视splitChunks.minSize、splitChunks.minChunks、splitChunks.maxAsyncRequests 和 splitChunks.maxInitialRequests 选项，并始终为此缓存组创建 chunk
                  enforce: true,
                },
            }*/

        },
        /*
        chunkFilename中设置hash会导致一个问题：修改a文件导致b文件contenthash变化
        （因为在index.js中import a.js，index.js中记录了a.js的hash值，而a.js改变，其contenthash改变，
        导致index.js文件内容中记录的contenthash也改变，重新打包后index.js的contenthash也会变，这样就会使缓存失效）
        解决办法：runtimeChunk --> 将当前模块记录其他模块的hash单独打包为一个文件 runtime
        */
        // 默认值为false
        // true：对于每个entry会生成runtime~${entrypoint.name}的文件。
        // name:{}：自定义runtime文件的name,也可以是字符串value
        runtimeChunk: {
            name: entrypoint => `runtime~${entrypoint.name}`
        },
        // 默认为true，效果就是压缩js代码
        // webpack5 默认使用 TerserWebpackPlugin
        minimizer: [
            // 配置生产环境的压缩方案：js/css
            // webpack5 TerserWebpackPlugin 配置更新，移除了 cache 和 sourceMap 选项
            new TerserWebpackPlugin({
                // 开启多进程打包
                parallel: true,
                // webpack5 使用 terserOptions 配置压缩选项
                terserOptions: {
                    compress: {
                        drop_console: true, // 移除 console
                        drop_debugger: true // 移除 debugger
                    },
                    format: {
                        comments: false // 移除注释
                    }
                },
                extractComments: false // 不将注释提取到单独文件
            }),
            // webpack5 压缩css
            new CssMinimizerPlugin({
                parallel: true // 开启多进程压缩
            }),
            // webpack5 压缩图片 (配置方式已更新)
            new ImageMinimizerWebpackPlugin({
                minimizer: {
                    implementation: ImageMinimizerWebpackPlugin.imageminMinify,
                    options: {
                        plugins: [
                            ['gifsicle', { interlaced: true }],
                            ['jpegtran', { progressive: true }],
                            ['optipng', { optimizationLevel: 5 }],
                            [
                                'svgo',
                                {
                                    plugins: [
                                        {
                                            name: 'preset-default',
                                            params: {
                                                overrides: {
                                                    removeViewBox: false,
                                                },
                                            },
                                        },
                                    ],
                                },
                            ],
                        ],
                    },
                },
            }),
        ],
        // production 默认为true
        // 标识tree shaking代码，死代码
        // 结合TerserWebpackPlugin，可清除死代码
        // 只导出外部使用的模块成员 负责标记枯树叶
        usedExports: true,
        // 开启副作用，优先在packetjson中开启
        // sideEffects: true,
    },

    // https://blog.csdn.net/qq_39207948/article/details/124802300
    // webpack5 设置cache能够提升构建速度
    cache: {
        // type: 'memory', // 临时内存缓存
        type: 'filesystem', // 文件缓存
        // 使用这些项和所有依赖项的哈希值来使文件系统缓存失效
        // 设置以下配置,来获取最新配置以及所有依赖项
        buildDependencies: {
            config: [__filename]
        },
        name: 'dev_cache', // 缓存文件名称
        cacheDirectory: 'node_modules/.cache/webpack', // 指定缓存目录
    }
}
```

### 6\. 打包加速的方法

+   devtool 的 sourceMap较为耗时
+   开发环境不做无意义的操作：代码压缩、目录内容清理、计算文件hash、提取CSS文件等
+   noParse：不需要解析某些模块的依赖
+   第三方依赖外链script引入：vue、ui组件、JQuery等
+   babel-loader开启缓存 `cacheDirectory: true`
+   splitChunks：提取公共模块，将符合引用次数(minChunks)的模块打包到一起，利用浏览器缓存
+   thread-loader：多线程编译，加快编译速度（webpack5 中 happypack 已弃用，推荐使用 thread-loader）
+   **webpack5 持久化缓存**：通过 `cache: { type: 'filesystem' }` 配置，大幅提升二次构建速度
+   Tree Shaking 摇树：基于ES6提供的模块系统对代码进行静态分析，并在压缩阶段将代码中的死代码（dead code）移除，减少代码体积

### 7\. 打包体积 优化思路

+   webpack-bundle-analyzer插件可以可视化的查看webpack打包出来的各个文件体积大小，以便我们定位大文件，进行体积优化
+   提取第三方库或通过引用外部文件的方式引入第三方库
+   压缩插件`TerserWebpackPlugin` `CssMinimizerPlugin` `ImageMinimizerWebpackPlugin`
+   服务器启用gzip / brotli 压缩
+   按需加载资源文件 `import()`
+   剥离`css`文件，单独打包
+   去除不必要插件，开发环境与生产环境用不同配置文件
+   SpritesmithPlugin雪碧图，将多个小图片打包成一张，用background-image，background-position，width，height控制显示部分
+   webpack5 Asset Modules：使用 `type: 'asset'` 替代 url-loader，小于阈值转 base64，大于阈值输出文件


### 8\. 常用插件简述

+   HtmlWebpackPlugin：生成html文件，并自动引入打包输出的资源
+   CopyWebpackPlugin：复制文件
+   ProvidePlugin：全局变量设置
+   DefinePlugin：定义全局常量
+   mini-css-extract-plugin：css单独打包
+   TerserPlugin：压缩代码（webpack5 内置）
+   CompressionWebpackPlugin：gzip / Brotli 压缩静态文件
+   webpack-bundle-analyzer：可视化的查看webpack打包出来的各个文件体积大小

### 9\. Tree Shaking 摇树

**背景：** 项目中，有一个入口文件，相当于一棵树的主干，入口文件有很多依赖的模块，相当于树枝。实际情况中，虽然依赖了某个模块，但其实只使用其中的某些功能。通过 tree-shaking，将没有使用的模块摇掉，这样来达到删除无用代码的目的。

**思路：** 基于ES6提供的模块系统对代码进行静态分析,并将代码中的死代码（dead code)移除的一种技术。因此，利用Tree Shaking技术可以很方便地实现我们代码上的优化，减少代码体积。

Tree Shaking 摇树 是借鉴了 rollup 的实现。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22aace03c6b74574991c2a8e15463fd7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

**摇树删除代码的原理：** webpack基于ES6提供的模块系统，对代码的依赖树进行静态分析，把import & export标记为3类：

+   所有import标记为/\* harmony import \*/
+   被使用过的export标记为/harmony export(\[type\])/，其中\[type\]和webpack内部有关，可能是binding，immutable等；
+   没有被使用的export标记为/\* unused harmony export \[FuncName\] \*/，其中\[FuncName\]为export的方法名，之后使用 TerserPlugin（webpack5 默认）进行代码精简，把没用的都删除。

**为何基于es6模块实现（ES6 module 特点：）：**

+   只能作为模块顶层的语句出现
+   import的模块名只能是字符串常量
+   import binding是immutable的

**条件：**

1.  首先源码必须遵循 ES6 的模块规范 (import & export)，如果是 CommonJS 规范 (require) 则无法使用。
2.  编写的模块代码不能有副作用，如果在代码内部改变了外部的变量则不会被移除。

**配置方法：**

1.  在package.json里添加一个属性：

```js
{
    // sideEffects如果设为false，webpack就会认为所有没用到的函数都是没副作用的，即删了也没关系。
    "sideEffects": false,
    // 设置黑名单，用于防止误删代码
    "sideEffects": [
        // 数组里列出黑名单，禁止shaking下列代码
        "@babel/polyfill",
        "*.less",
        // 其它有副作用的模块
        "./src/some-side-effectful-file.js"
    ],
}
```

tree-shaking 摇掉代码中未使用的代码 在生产模式下自动开启

tree-shaking并不是webpack中的某一个配置选项，是一组功能搭配使用后的优化效果，会在生产模式下自动启动

```js
// 在开发模式下，设置 usedExports: true ，打包时只会标记出哪些模块没有被使用，不会删除，因为可能会影响 source-map的标记位置的准确性。
{
    mode: 'development',
    optimization: {
        // 优化导出的模块
        usedExports: true
    },
}
// 在生产模式下默认开启 usedExports: true ，打包压缩时就会将没用到的代码移除
{
    mode: 'production',
    //  这个属性的作用就是集中配置webpack内部的优化功能
    optimization: {
        // 只导出外部使用的模块成员 负责标记枯树叶
        usedExports: true,
        minimize: true, // 自动压缩代码 负责摇掉枯树叶
        /**
         * 作用域提升（Scope Hoisting）
         * 将所有模块尽可能合并到一个函数中，减少闭包数量
         * 提升运行效率，减少代码体积
         */
        concatenateModules: true,
    },

}
```

**使用摇树的注意事项：**

1.  使用 ES6 模块语法编写代码
2.  工具类函数尽量以单独的形式输出，不要集中成一个对象或者类
3.  声明 sideEffects
4.  自己在重构代码时也要注意副作用

**tree-shaking & babel** 使用babel-loader处理js代码会导致tree-shaking失效的原因：

+   treeshaking 使用的前提必须是ES module组织的代码，也就是说交给ESMOdule处理的代码必须是ESM。当我们使用babel-loader处理js代码之后就有可能将ESM 转换 成commonjs规范（preset-env插件工作的时候就会将esm => coommonjs）

解决办法：  
手动配置 preset-env 的 modules: false，确保不会开启自动转换的插件（在最新版本的 babel-loader 中已自动关闭转换成 commonjs 规范的功能）

```js
presets: [
    ['@babel/preset-env', { modules: false }]
]
```
