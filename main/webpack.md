## webpack

+   `webpack` 是一个模块打包工具，你可以使用webpack管理你的**模块依赖**，并**编绎输出**模块们所需的静态文件。
+   它能够很好地**管理、打包**Web开发中所用到的HTML、Javascript、CSS以及各种静态文件（图片、字体等），让开发过程更加高效。
+   对于不同类型的资源，webpack有对应的模块加载器`loader`。
+   webpack模块打包器会分析模块间的**依赖关系**，最后生成优化且合并后的静态资源。
+   其**插件**功能提供了处理各种文件过程中的各个生命周期钩子，使开发者能够利用插件功能开发很多自定义的功能。
### 2\. 核心打包原理简述

### 3\. Loader

**编写一个loader**:

`loader`就是一个`node`模块，它输出了一个函数。当某种资源需要用这个`loader`转换时，这个函数会被调用。并且，这个函数可以通过提供给它的`this`上下文访问`Loader API`。 `reverse-txt-loader`。

**解析顺序**：`从下向上，从右向左`

```js
// 定义
module.exports = function(src) {
    //src是原文件内容（abcde），下面对内容进行处理，这里是反转
    var result = src.split('').reverse().join('');
    //返回JavaScript源码，必须是String或者Buffer
    return `module.exports = '${result}'`;
}
//使用
{
    test: /\.txt$/,
    loader: 'reverse-txt-loader'
}
```

### 4\. 配置举例

**§ base**

```js
module.exports = {
    entry: {
        app: './src/main.js'
    },
    output: {
        path: path.resolve(__dirname, '../dist'),
        filename: '[name].js',
        publicPath: ''
    },
    resolve: {
        extensions: ['.js', '.vue', '.json', '.css', '.less'],
        alias: {
            '@': resolve('src'),
            src: resolve('src'),
            static: resolve('static')
        }
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader',
                options: vueLoaderConfig
            },
            {
                test: /\.js$/,
                loader: 'babel-loader',
                include: [resolve('src'), resolve('test')]
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                loader: 'url-loader',
                query: {
                    limit: 1,
                    name: utils.assetsPath(`img/[name].[ext]`)
                },
                include: /nobase64/
            },
            {
                test: /\.svg(\?\S*)?$/,
                loader: 'svg-sprite-loader',
                query: {
                    prefixize: true,
                    name: '[name]-[hash]'
                },
                include: [resolve('src')],
                exclude: /node_modules|bower_components/
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new ProgressBarPlugin({
            format: 'build [:bar] ' + chalk.green.bold(':percent') + ' (:elapsed seconds) : (:msg)',
            clear: false,
            width: 60
        })
    ]
};

```

**§ dev**

```js
var path = require('path');
var utils = require('./utils');
var webpack = require('webpack');
var config = require('../config');
var merge = require('webpack-merge');
var baseWebpackConfig = require('./webpack.base.conf');
var CopyWebpackPlugin = require('copy-webpack-plugin');
var MyPlugin = require('./htmlPlugin');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var MiniCssExtractPlugin = require('mini-css-extract-plugin');
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');

module.exports = merge(baseWebpackConfig, {
    mode: 'development',
    externals: {
        vue: 'Vue'
    },
    devtool: '#source-map',
    output: {
        path: config.build.assetsRoot,
        filename: utils.assetsPath(`js/[name].js`),
        chunkFilename: utils.assetsPath(`js/[name].js`)
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new FriendlyErrorsPlugin(),
        new CopyWebpackPlugin([
            {
                from: path.resolve(__dirname, `../static`),
                to: path.resolve(config.dev.assetsPublicPath, `/dist/static`)
            }
        ]),
        new HtmlWebpackPlugin({
            alwaysWriteToDisk: true,
            filename: 'index.html',
            template: path.resolve(__dirname, `../src/index.html`), // 模板路径
            inject: false,
            excludeChunks: [],
            baseUrl: '',
            currentEnv: 'development',
            envName: 'local',
            curentBranch: ''
        }),
        new HtmlHardDisk()
    ],
    optimization: {
        /*
            作用域提升插件
            [注意] 这个插件在 mode: production 时时默认开启的
            这样配置时为了在 development 时也开启
    https://webpack.js.org/configuration/optimization/#optimizationconcatenatemodules
      */
        concatenateModules: true,
        splitChunks: {
            cacheGroups: {
                commons: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all'
                },
                styles: {
                    name: 'styles',
                    test: /\.css$/,
                    chunks: 'all',
                    enforce: true
                }
            }
        },
        runtimeChunk: {
            name: 'manifest'
        },
        minimizer: [
            // 代码压缩UglifyJsPlugin的升级版
            new TerserPlugin({
                cache: true,
                parallel: true,
                terserOptions: {
                    sourceMap: true,
                    warnings: false,
                    compress: {
                        // warnings: false
                    },
                    ecma: 6,
                    mangle: true
                },
                sourceMap: true
            }),
            new OptimizeCSSPlugin({
                cssProcessorOptions: {
                    autoprefixer: {
                        browsers: 'last 2 version, IE > 8'
                    }
                }
            })
        ]
    }

}
```

**§ prod**

```js
var path = require('path');
var utils = require('./utils');
var webpack = require('webpack');
var merge = require('webpack-merge');
var baseWebpackConfig = require('./webpack.base.conf');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin');
var CopyWebpackPlugin = require('copy-webpack-plugin');
var HtmlHardDisk = require('html-webpack-harddisk-plugin');
var MiniCssExtractPlugin = require('mini-css-extract-plugin');
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');

module.exports = merge(baseWebpackConfig, {
    mode: 'production',
    externals: {
        vue: 'Vue'
    },
    output: {
        path: config.build.assetsRoot,
        filename: utils.assetsPath(`js/[name].js`),
        chunkFilename: utils.assetsPath(`js/[name]-[chunkhash].js`)
    },
    plugins: [
        // http://vuejs.github.io/vue-loader/en/workflow/production.html
        new webpack.DefinePlugin({
            'process.env': env
        }),
        // extract css into its own file
        new MiniCssExtractPlugin({
            filename: utils.assetsPath('css/[name].css'),
            allChunks: true
        }),
        // copy custom static assets
        new CopyWebpackPlugin([
            {
                from: path.resolve(__dirname, `../static`),
                to: path.resolve(__dirname, `../dist/static`)
            }
        ]),
        new HtmlWebpackPlugin({
            alwaysWriteToDisk: true,
            filename: 'index.html',
            template: path.resolve(__dirname, `../src/index.html`), // 模板路径
            inject: false,
            baseUrl: '',
            currentEnv: 'production',
            envName: 'prod',
            curentBranch: ''
        }),
        new BundleAnalyzerPlugin()
    ],
    optimization: {
        minimizer: [
            // 代码压缩UglifyJsPlugin的升级版
            new TerserPlugin({
                //  cache: true,
                parallel: true,
                terserOptions: {
                    warnings: false,
                    compress: {
                    },
                    //  ecma: 6,
                    mangle: true
                },
                sourceMap: true
            }),
            new OptimizeCSSPlugin({
                cssProcessorOptions: {
                    autoprefixer: {
                        browsers: 'last 2 version, IE > 8'
                    }
                }
            })
        ],
        splitChunks: {
            cacheGroups: {
                commons: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all'
                }
            }
        },
        runtimeChunk: {
            name: 'manifest'
        }
    }
};
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

### 8\. Tree Shaking 摇树

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

### 9\. 常用插件简述

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

