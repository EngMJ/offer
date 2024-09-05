## webpack

### 1\. 概述 
+   `webpack` 是一个模块打包工具，你可以使用webpack管理你的**模块依赖**，并**编绎输出**模块们所需的静态文件。

### [2\. 核心打包原理实现](https://github.com/EngMJ/Webpack4/tree/master/webpackGenerate)

    核心文件如下:
+ [webpackGenerate/lib/index.js](https://github.com/EngMJ/Webpack4/blob/master/webpackGenerate/lib/index.js)
+ [webpackGenerate/lib/compiler.js](https://github.com/EngMJ/Webpack4/blob/master/webpackGenerate/lib/compiler.js)
+ [webpackGenerate/lib/parser.js](https://github.com/EngMJ/Webpack4/blob/master/webpackGenerate/lib/parser.js)

### 3\. Loader

+ 3.1 `loader`就是一个`node`模块，一个函数。当某种资源需要用这个`loader`转换时，这个函数会被调用。并且，这个函数可以通过提供给它的`this`上下文访问`Loader API`。

+ 3.2 解析顺序：`从下向上，从右向左`

**编写一个babel-loader**:

```js
// 定义
const { getOptions } = require('loader-utils');
const { validate } = require('schema-utils');
const babel = require('@babel/core');
const util = require('util');

const babelSchema = require('./babelSchema.json');

// babel.transform用来编译代码的方法
// 是一个普通异步方法
// util.promisify将普通异步方法转化成基于promise的异步方法
const transform = util.promisify(babel.transform);

module.exports = function (content, map, meta) {
    // 获取loader的options配置
    const options = getOptions(this) || {};
    // 校验babel的options的配置
    validate(babelSchema, options, {
        name: 'Babel Loader'
    });

    // 创建异步
    const callback = this.async();

    // 使用babel编译代码
    transform(content, options)
        .then(({code, map}) => callback(null, code, map, meta))
        .catch((e) => callback(e))

}
//使用
{
    test: /\.txt$/,
    loader: 'my-babel-loader'
}
```

### 4\. plugins
插件可以执行各种任务，从打包优化到代码压缩、从环境变量注入到静态资源管理.

**编写一个CopyWebpackPlugin:**
```js
const path = require('path');
const fs = require('fs');
const {promisify} = require('util')

const { validate } = require('schema-utils');
const globby = require('globby');
const webpack = require('webpack');

const schema = require('./schema.json');

const readFile = promisify(fs.readFile);
const {RawSource} = webpack.sources

class CopyWebpackPlugin {
  constructor(options = {}) {
    // 验证options是否符合规范
    validate(schema, options, {
      name: 'CopyWebpackPlugin'
    })

    this.options = options;
  }

  apply(compiler) {
    // 初始化compilation
    compiler.hooks.thisCompilation.tap('CopyWebpackPlugin', (compilation) => {
      // 添加资源的hooks
      compilation.hooks.additionalAssets.tapAsync('CopyWebpackPlugin', async (cb) => {
        // 将from中的资源复制到to中，输出出去
        const { from, ignore } = this.options;
        const to = this.options.to ? this.options.to : '.';

        // context就是webpack配置
        // 运行指令的目录
        const context = compiler.options.context; // process.cwd()
        // 将输入路径变成绝对路径
        const absoluteFrom = path.isAbsolute(from) ? from : path.resolve(context, from);

        // 1. 过滤掉ignore的文件
        // globby(要处理的文件夹，options)
        const paths = await globby(absoluteFrom, { ignore });

        console.log(paths); // 所有要加载的文件路径数组

        // 2. 读取paths中所有资源
        const files = await Promise.all(
          paths.map(async (absolutePath) => {
            // 读取文件
            const data = await readFile(absolutePath);
            // basename得到最后的文件名称
            const relativePath = path.basename(absolutePath);
            // 和to属性结合
            // 没有to --> reset.css
            // 有to --> css/reset.css
            const filename = path.join(to, relativePath);

            return {
              // 文件数据
              data,
              // 文件名称
              filename
            }
          })
        )

        // 3. 生成webpack格式的资源
        const assets = files.map((file) => {
          const source = new RawSource(file.data);
          return {
            source,
            filename: file.filename
          }
        })

        // 4. 添加compilation中，输出出去
        assets.forEach((asset) => {
          compilation.emitAsset(asset.filename, asset.source);
        })

        cb();
      })
    })
  }

}

module.exports = CopyWebpackPlugin;
```

### 5\. 配置举例

```js
const { resolve } = require('path')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');
const hardSourceWebpackPlugin = require('hard-source-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin')
const ScriptExtHtmlWebpackPlugin = require('script-ext-html-webpack-plugin')
// webpack5 模块联邦
const { ModuleFederationPlugin } = require('webpack').container;

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
 * webpack 5 tree Shaking优化,增加cache属性,将hash转变为fullhash,不再为 Node.js 模块自动引用 polyfill、也内置了terser进行代码压缩
 * fullhash 是根据打包中的所有文件计算出来的 hash 值，在一次打包中，所有的资源出口文件的filename获得的[hash]都是一样的
 * contenthash webpack4对于添加注释和修改变量其实，是会影响它的一个contenthash值的计算. webpack 5不会影响,contenthash值不变
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
        // chunkshash hash 分别代表对应块hash与文件打包hash，chunkhash可能相同，
        // 而hash则是每次打包都不同，不管内容是否有变化
        filename: 'js/build_[contenthash:10].js',
        path: resolve(__dirname, 'build'),
        // 所有资源引入公共路径前缀 --> 'imgs/a.jpg' --> '/imgs/a.jpg'
        // 通过cross-env设置打包环境变量, 在根据变量打包时进行动态选择,各个配置项参数可以动态修改publicPath, 实现不同项目使用不同的资源路径
        // package.json: scripts: { "dev": "cross-env NODE_ENV=development webpack-dev-server --config webpack.config.js", }
        publicPath: '/',
        chunkFilename: 'js/[name]_chunk_[contenthash:10].js', // 非入口chunk的名称
        library: '[name]', // 打包整个库后向外暴露的变量名
        libraryTarget: 'window' // 变量名添加到哪个变量上 browser
        // libraryTarget: 'global' // 变量名添加到哪个变量上 node
        // libraryTarget: 'commonjs' // 变量名添加到哪个变量上 conmmonjs模块 exports
    },
    module: {
        // 防止 webpack 解析那些任何与给定正则表达式相匹配的文件。
        // 忽略的文件中不应该含有import,require,define的调用，或任何其他导入机制。
        // 忽略大型的 library 可以提高构建性能。
        // externals共用会导致externals失效，因为会不转换require(),导致文件无法读取
        noParse:/jquery/,
        rules: [
            {
                test: /\.less/,
                use: [
                    // 'style-loader', // 以style标签行内样式的方式,插入head标签中
                    MiniCssExtractPlugin.loader, // 打包成css单文件
                    'css-loader', // 加载读取css文件内容
                    {
                        // 还需要子pakege.json中定义browserslist
                        // 为css样式添加兼容前缀
                        loader: 'postcss-loader',
                        options: {
                            ident:'postcss',
                            plugins: () => [require('postcss-preset-env')()]
                        }
                    },
                    {
                        // 处理less文件,转换为css文件
                        loders: 'less-loader',
                        options: {
                        }
                    }
                ]
            },
            {
                oneOf:[
                    {
                        test: /\.js$/,
                        use: [
                            'cache-loader', // 缓存对应loader所转换的文件
                            'thread-loader',
                            {
                                loader: 'babel-loader',
                                options: {
                                    // cacheDirectory: true,
                                    presets: [
                                        '@babel/preset-env',
                                        {
                                            useBuiltIns: 'usage', // 按需加载
                                            corejs: { version:3 }, // 制定core-js版本
                                            target: { // 制定向后兼容到什么版本
                                                chrome: '60',
                                                fireox: '50',
                                                safari: '10',
                                                ie: '9',
                                                edge: '17'
                                            }
                                        }
                                    ]
                                }
                            }
                        ],
                        includes: /src/,
                        excludes: /node_modules/
                    },
                    {
                        test: /\.js$/,
                        use: [
                            'cache-loader', // 缓存
                            'thread-loader', // 多线程
                            {
                                loader: 'eslint-loader',
                                options: {
                                    fix: true
                                }
                            }
                        ],
                        enforce: 'pre', // 先执行
                        // enforce: 'post', // 延后执行
                        includes: /src/
                    },
                    {
                        test: /\.(jpg|png|gif)$/,
                        use: 'url-loader',
                        options: {
                            limit: 8 * 1024, // 大小小于这个的文件全部转换为base64编码方式加载
                            name: '[name]_[contenthash:10].[ext]',
                            outputPath: 'imgs',
                            esModule: false // 因为要配合html-loder的commenjs风格的转换，所以关闭esmodule风格
                        }
                        // use: 'asset/inline'    webpack5,不再使用url-loader
                    },
                    {
                        test: /\.html$/,
                        loader: 'html-loader' // 将 HTML 导出为字符串。当编译器需要时，将压缩 HTML 字符串
                    },
                    {
                        exclude: /\.(js|css|html|less|jpg|png|gif)$/,
                        loader: 'file-loader',
                        options: {
                            outputPath: 'media'
                        }
                        // use: 'asset/resource'    webpack5,不再使用file-loader
                    },
                    {
                        test: /\.text$/i,
                        use: 'raw-loader' // 将文件加载为 字符串文本
                        // use: 'asset/source'    webpack5,不再使用raw-loader
                    },
                ]
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({ // 将css打包成单文件
            filename: 'css/built[contenthash:10].css'
        }),
        new OptimizeCssAssetsWebpackPlugin(), // 压缩css
        new HtmlWebpackPlugin({ //  自动引入各个打包文件
            template: './src/index.html', // 使用模板
            minify: {
                collapseWhitespace: true, // 移除空格
                removeComments: true // 移除注释
            }
        }),
        // 告诉webpack哪些库不参与打包，同时使用时的名称也得变
        new webpack.DllReferencePlugin({
            manifest: resolve(__dirname, 'dll/manifest.json')
        }),
        // 将某个文件打包输出到build目录下，并在html中自动引入该资源
        new AddAssetHtmlWebpackPlugin({
            filepath: resolve(__dirname, 'dll/jquery.js')
        }),
        // 缓存模块，提升打包速度
        new hardSourceWebpackPlugin(
            // {
            //     //设置缓存目录的路径
            //     //相对路径或者绝对路径
            //     cacheDirectory: 'node_modules/.cache/hard-source/[confighash]',
            //     //构建不同的缓存目录名称
            //     //也就是cacheDirectory中的[confighash]值
            //     configHash: function(webpackConfig) {
            //         return require('node-object-hash')({sort: false}).hash(webpackConfig);
            //     },
            //     //环境hash
            //     //当loader、plugin或者其他npm依赖改变时进行替换缓存
            //     environmentHash: {
            //         root: process.cwd(),
            //         directories: [],
            //         files: ['package-lock.json', 'yarn.lock'],
            //     },
            //     //自动清除缓存
            //     cachePrune: {
            //         //缓存最长时间（默认2天）
            //         maxAge: 2 * 24 * 60 * 60 * 1000,
            //         //所有的缓存大小超过size值将会被清除
            //         //默认50MB
            //         sizeThreshold: 50 * 1024 * 1024
            //     },
            // }
        ),
        // 将runtimechunk文件，以行内形式打包进index.html,减少文件请求
        new ScriptExtHtmlWebpackPlugin({
            inline: /runtime~.+\.js$/  //正则匹配runtime文件名
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
                RemoteA: `RemoteA@${env.A_URL}/remoteEntry.js`,
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
            // 控制远程模块的加载方式
            // 选项的值可以是 'async'、'sync' 或 'prefetch'，分别代表异步加载、同步加载和预加载
            remotesType: 'async',
            // 暴露给外部的模块/文件夹/动态加载
            // 键表示其他模块中引用的路径，值表示当前模块中实际的模块路径
            exposes: {
                Button: './src/components/Button',
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
             *         // 设置为true,表示共享模块将在主应用程序启动时立即加载,
             *         // 默认值false, 表示共享模块将会按需加载，即在模块被使用时才会被加载
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
            // 选项的值可以是 'eager' 或 'lazy'，分别代表主应用程序启动时立即加载和按需加载
            sharedType: 'lazy',
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
        proxy: {
            '/api':{
                target: 'https://localhost:8080', // 请求访问地址
                changeOrigin: true, // 修改源访问
                pathRewrite: {
                    '^/api': '/api' // 替换请求地址
                }
            }
        },
        /*跨域问题：同源策略中不同的协议、端口号、域名就会产生跨域。正常的浏览器和服务器之间有跨域，但是服务器之间没有跨域。
    代码通过代理服务器运行，所以浏览器和代理服务器之间没有跨域，浏览器吧请求发送到代理服务器上，代理服务器替你转发到另外一个服务器上
    服务器之间没有跨域，所以请求成功。代理服务器再把接收到的响应响应给浏览器。这样就解决开发环境下的跨域问题

    */
        // 不要显示启动服务器日志信息
        clientLogLevel: 'none',
        // 除了一些基本信息外，其他内容都不要显示
        quiet: true,
        // 如果出错了，不要全屏提示
        overlay: false,
        // 运行代码所在的目录
        contentBase: resolve(__dirname, 'build'),
        // 监视contentBase目录下的所有文件，一旦文件变化就会reload
        watchContentBase: true,
        watchOptions: {
            // 忽略文件
            ignored: /node_modules/
        },
    },
    devtool: 'eval-source-map',
    model: 'production', // development
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
            // 代码分割时默认对异步代码生效，all：所有代码有效，inital：同步代码有效
            chunks: 'all',
            /* 以下都是默认值，可以不写
            miniSize: 30 * 1024, // 分割的chunk最小为30kb（大于30kb的才分割）
            maxSize: 0, // 最大没有限制
            minChunks: 1, // 要提取的chunk最少被引用1次
            maxAsyncRequests: 5, // 按需加载时并行加载的文件的最大数量为5
            maxInitialRequests: 3, // 入口js文件最大并行请求数量
            automaticNameDelimiter: '~', // 名称连接符
            name: true, // 可以使用命名规则
            usedExports: true, // 打包时只包含应用中真正使用的模块,不打包使用的模块
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
        // webpack4默认的压缩为uglifyjs-webpack-plugin
        // webpack5默认TerserWebpackPlugin
        minimizer: [
            // 配置生产环境的压缩方案：js/css
            new TerserWebpackPlugin({
                // 开启缓存
                cache: true,
                // 开启多进程打包
                parallel: true,
                // 启用sourceMap(否则会被压缩掉)
                sourceMap: false
            })
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
        name: 'dev_cache' // 缓存文件名称
    }
}
```

### 6\. 打包加速的方法

+   devtool 的 sourceMap较为耗时
+   开发环境不做无意义的操作：代码压缩、目录内容清理、计算文件hash、提取CSS文件等
+   第三方依赖外链script引入：vue、ui组件、JQuery等
+   HotModuleReplacementPlugin：热更新增量构建
+   DllPlugin& DllReferencePlugin：动态链接库，提高打包效率，仅打包一次第三方模块，每次构建只重新打包业务代码。
+   thread-loader,happypack：多线程编译，加快编译速度
+   noParse：不需要解析某些模块的依赖
+   babel-loader开启缓存cache
+   splitChunks（老版本用CommonsChunkPlugin）：提取公共模块，将符合引用次数(minChunks)的模块打包到一起，利用浏览器缓存
+   Tree Shaking 摇树：基于ES6提供的模块系统对代码进行静态分析, 并在压缩阶段将代码中的死代码（dead code)移除，减少代码体积。

### 7\. 打包体积 优化思路

+   webpack-bundle-analyzer插件可以可视化的查看webpack打包出来的各个文件体积大小，以便我们定位大文件，进行体积优化
+   提取第三方库或通过引用外部文件的方式引入第三方库
+   代码压缩插件`UglifyJsPlugin`
+   服务器启用gzip压缩
+   按需加载资源文件 `require.ensure`\=
+   剥离`css`文件，单独打包
+   去除不必要插件，开发环境与生产环境用不同配置文件
+   SpritesmithPlugin雪碧图，将多个小图片打包成一张，用background-image，backgroud-pisition，width，height控制显示部分
+   url-loader 文件大小小于设置的尺寸变成base-64编码文本，大与尺寸由file-loader拷贝到目标目录


### 8\. 常用插件简述

+   webpack-dev-server
+   clean-webpack-plugin：编译前清理输出目录
+   CopyWebpackPlugin：复制文件
+   HotModuleReplacementPlugin：热更新
+   ProvidePlugin：全局变量设置
+   DefinePlugin：定义全局常量
+   splitChunks（老版本用CommonsChunkPlugin）：提取公共模块，将符合引用次数的模块打包到一起
+   mini-css-extract-plugin（老版本用ExtractTextWebpackPlugin）：css单独打包
+   TerserPlugin（老版本用UglifyJsPlugin）：压缩代码
+   progress-bar-webpack-plugin：编译进度条
+   DllPlugin& DllReferencePlugin：提高打包效率，仅打包一次第三方模块
+   webpack-bundle-analyzer：可视化的查看webpack打包出来的各个文件体积大小
+   thread-loader,happypack：多进程编译，加快编译速度

### 9\. Tree Shaking 摇树

**背景：** 项目中，有一个入口文件，相当于一棵树的主干，入口文件有很多依赖的模块，相当于树枝。实际情况中，虽然依赖了某个模块，但其实只使用其中的某些功能。通过 tree-shaking，将没有使用的模块摇掉，这样来达到删除无用代码的目的。

**思路：** 基于ES6提供的模块系统对代码进行静态分析,并将代码中的死代码（dead code)移除的一种技术。因此，利用Tree Shaking技术可以很方便地实现我们代码上的优化，减少代码体积。

Tree Shaking 摇树 是借鉴了 rollup 的实现。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22aace03c6b74574991c2a8e15463fd7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

**摇树删除代码的原理：** webpack基于ES6提供的模块系统，对代码的依赖树进行静态分析，把import & export标记为3类：

+   所有import标记为/\* harmony import \*/
+   被使用过的export标记为/harmony export(\[type\])/，其中\[type\]和webpack内部有关，可能是binding，immutable等；
+   没有被使用的export标记为/\* unused harmony export \[FuncName\] \*/，其中\[FuncName\]为export的方法名，之后使用Uglifyjs（或者其他类似的工具）进行代码精简，把没用的都删除。

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
        "@babel/polly-fill",
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
    mode: 'develpoment',
    optimization: {
        // 优化导出的模块
        usedExports: true
    },
}
// 在生产模式下默认开启 usedExports: true ，打包压缩时就会将没用到的代码移除
{
    mode: 'production',
    //  这个属性的作用就是集中配置webpack内部的优化功能
    optimizition: {
        // 只导出外部使用的模块成员 负责标记枯树叶
        usedExports: true,
        minimize: true, // 自动压缩代码 负责摇掉枯树叶
        /**
         * webpack打包默认会将一个模块单独打包到一个闭包中
         * webpack3中新增的API 将所有模块都放在一个函数中 ，尽可能将所有模块合并在一起，
         * 提升效率，减少体积  达到作用域提升的效果
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
收到配置preset-env的modules： false,确保不会开启自动转换的插件(在最新版本的babel-loader中自动帮我们关闭了转换成commonjs规范的功能)

```js
presets: [
    ['@babel/preset-env', {module: 'commonjs'}]
]
```
