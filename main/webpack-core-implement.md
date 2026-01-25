## Webpack5 核心打包原理代码实现

> 本文档是 [webpack.md](./webpack.md) 的补充文档，包含 webpack5 核心打包原理的完整代码实现。

### 1. 核心文件结构

```
mini-webpack/
├── lib/
│   ├── index.js        # 入口文件
│   ├── compiler.js     # 编译器核心
│   ├── parser.js       # AST 解析器
│   └── template.js     # 输出模板
└── webpack.config.js   # 配置文件
```

### 2. 入口文件 (index.js)

```js
const Compiler = require('./compiler');

/**
 * webpack 入口函数
 * @param {Object} options - webpack 配置对象
 * @returns {Compiler} 编译器实例
 */
function webpack(options) {
    // 1. 初始化阶段：合并配置参数
    // 将用户配置与默认配置合并
    const mergedOptions = {
        entry: './src/index.js',
        output: {
            path: 'dist',
            filename: 'bundle.js'
        },
        module: { rules: [] },
        plugins: [],
        ...options
    };

    // 2. 创建 Compiler 实例
    // Compiler 是 webpack 的核心引擎，负责整个编译流程
    const compiler = new Compiler(mergedOptions);

    // 3. 注册插件
    // 遍历 plugins 数组，调用每个插件的 apply 方法
    // 插件通过 apply 方法注册到 compiler 的生命周期钩子上
    if (mergedOptions.plugins && Array.isArray(mergedOptions.plugins)) {
        mergedOptions.plugins.forEach(plugin => {
            plugin.apply(compiler);
        });
    }

    return compiler;
}

module.exports = webpack;
```

### 3. 编译器核心 (compiler.js)

```js
const path = require('path');
const fs = require('fs');
const parser = require('./parser');
const { SyncHook, AsyncSeriesHook } = require('tapable');

/**
 * Compiler 类 - webpack 编译器核心
 * 负责整个编译流程的调度和执行
 */
class Compiler {
    constructor(options) {
        this.options = options;
        // 入口文件路径
        this.entry = options.entry;
        // 输出配置
        this.output = options.output;
        // 模块规则（loader 配置）
        this.rules = options.module?.rules || [];
        // 存储所有模块
        this.modules = new Map();
        // 存储所有 chunk
        this.chunks = new Map();
        // 存储最终输出的资源
        this.assets = {};
        // 当前工作目录
        this.context = process.cwd();

        // ========== Tapable 钩子系统 ==========
        // webpack5 使用 tapable 实现插件机制
        this.hooks = {
            // 初始化完成
            initialize: new SyncHook(),
            // 编译开始前
            beforeRun: new AsyncSeriesHook(['compiler']),
            // 开始编译
            run: new AsyncSeriesHook(['compiler']),
            // 编译完成
            done: new AsyncSeriesHook(['stats']),
            // 创建 compilation 对象时
            compilation: new SyncHook(['compilation', 'params']),
            // 生成资源到 output 目录之前
            emit: new AsyncSeriesHook(['compilation']),
            // 生成资源到 output 目录之后
            afterEmit: new AsyncSeriesHook(['compilation']),
        };
    }

    /**
     * 启动编译流程
     * @param {Function} callback - 编译完成的回调函数
     */
    run(callback) {
        // 触发 beforeRun 钩子
        this.hooks.beforeRun.callAsync(this, (err) => {
            if (err) return callback(err);

            // 触发 run 钩子
            this.hooks.run.callAsync(this, (err) => {
                if (err) return callback(err);

                // 开始编译
                this.compile((err, compilation) => {
                    if (err) return callback(err);

                    // 触发 emit 钩子（输出资源前）
                    this.hooks.emit.callAsync(compilation, (err) => {
                        if (err) return callback(err);

                        // 输出文件
                        this.emitAssets(compilation);

                        // 触发 afterEmit 钩子
                        this.hooks.afterEmit.callAsync(compilation, (err) => {
                            if (err) return callback(err);

                            // 触发 done 钩子
                            const stats = this.getStats();
                            this.hooks.done.callAsync(stats, (err) => {
                                callback(err, stats);
                            });
                        });
                    });
                });
            });
        });
    }

    /**
     * 编译主流程
     * @param {Function} callback - 回调函数
     */
    compile(callback) {
        // 创建 Compilation 对象
        const compilation = {
            modules: this.modules,
            chunks: this.chunks,
            assets: this.assets,
            options: this.options
        };

        // 触发 compilation 钩子
        this.hooks.compilation.call(compilation, {});

        try {
            // 1. 从入口开始构建模块依赖图
            this.buildModuleGraph();

            // 2. 生成 chunk
            this.generateChunks();

            // 3. 生成最终代码
            this.generateCode();

            callback(null, compilation);
        } catch (err) {
            callback(err);
        }
    }

    /**
     * 构建模块依赖图
     * 从入口文件开始，递归解析所有依赖
     */
    buildModuleGraph() {
        // 获取入口文件的绝对路径
        const entryPath = path.resolve(this.context, this.entry);
        
        // 从入口开始递归构建
        this.buildModule(entryPath, true);
    }

    /**
     * 构建单个模块
     * @param {string} modulePath - 模块的绝对路径
     * @param {boolean} isEntry - 是否为入口模块
     * @returns {Object} 模块对象
     */
    buildModule(modulePath, isEntry = false) {
        // 如果模块已经构建过，直接返回（避免循环依赖）
        if (this.modules.has(modulePath)) {
            return this.modules.get(modulePath);
        }

        // 1. 读取模块源代码
        let sourceCode = fs.readFileSync(modulePath, 'utf-8');

        // 2. 应用匹配的 loader 进行转换
        // loader 从右到左、从下到上执行
        sourceCode = this.applyLoaders(modulePath, sourceCode);

        // 3. 使用 AST 解析模块，提取依赖
        const { code, dependencies } = parser.parse(sourceCode, {
            filename: modulePath,
            sourceType: 'module'
        });

        // 4. 创建模块对象
        const moduleId = this.getModuleId(modulePath);
        const module = {
            id: moduleId,
            path: modulePath,
            code: code,
            dependencies: [],
            isEntry: isEntry
        };

        // 5. 先将模块加入 modules（处理循环依赖）
        this.modules.set(modulePath, module);

        // 6. 递归处理依赖模块
        dependencies.forEach(dep => {
            // 解析依赖的绝对路径
            const depPath = this.resolvePath(modulePath, dep);
            
            // 递归构建依赖模块
            const depModule = this.buildModule(depPath);
            
            // 记录依赖关系
            module.dependencies.push({
                request: dep,           // 原始请求路径
                moduleId: depModule.id  // 解析后的模块 ID
            });
        });

        return module;
    }

    /**
     * 应用 loader 转换源代码
     * @param {string} modulePath - 模块路径
     * @param {string} sourceCode - 源代码
     * @returns {string} 转换后的代码
     */
    applyLoaders(modulePath, sourceCode) {
        let result = sourceCode;

        // 遍历所有 loader 规则
        this.rules.forEach(rule => {
            // 检查模块路径是否匹配规则
            if (rule.test && rule.test.test(modulePath)) {
                // 获取 loader 列表（可能是单个或数组）
                const loaders = Array.isArray(rule.use) ? rule.use : [rule.use];
                
                // 从右到左执行 loader
                for (let i = loaders.length - 1; i >= 0; i--) {
                    const loaderConfig = loaders[i];
                    const loaderPath = typeof loaderConfig === 'string' 
                        ? loaderConfig 
                        : loaderConfig.loader;
                    const loaderOptions = typeof loaderConfig === 'object' 
                        ? loaderConfig.options 
                        : {};

                    // 加载并执行 loader
                    const loader = require(loaderPath);
                    
                    // 创建 loader 上下文（模拟 webpack 的 loaderContext）
                    const loaderContext = {
                        resourcePath: modulePath,
                        // webpack5: 使用 this.getOptions() 替代 loader-utils
                        getOptions: () => loaderOptions,
                        async: () => {
                            // 简化实现，返回同步回调
                            return (err, content) => {
                                if (err) throw err;
                                result = content;
                            };
                        },
                        callback: (err, content) => {
                            if (err) throw err;
                            result = content;
                        }
                    };

                    // 执行 loader，绑定上下文
                    result = loader.call(loaderContext, result);
                }
            }
        });

        return result;
    }

    /**
     * 解析模块路径
     * @param {string} currentPath - 当前模块路径
     * @param {string} request - 请求的模块路径
     * @returns {string} 解析后的绝对路径
     */
    resolvePath(currentPath, request) {
        const currentDir = path.dirname(currentPath);
        
        // 相对路径
        if (request.startsWith('./') || request.startsWith('../')) {
            let resolved = path.resolve(currentDir, request);
            
            // 尝试添加扩展名
            const extensions = ['.js', '.jsx', '.ts', '.tsx', '.json'];
            for (const ext of extensions) {
                if (fs.existsSync(resolved + ext)) {
                    return resolved + ext;
                }
            }
            
            // 尝试解析 index 文件
            if (fs.existsSync(resolved) && fs.statSync(resolved).isDirectory()) {
                for (const ext of extensions) {
                    const indexPath = path.join(resolved, 'index' + ext);
                    if (fs.existsSync(indexPath)) {
                        return indexPath;
                    }
                }
            }
            
            return resolved;
        }
        
        // node_modules 模块（简化实现）
        return require.resolve(request, { paths: [currentDir] });
    }

    /**
     * 获取模块 ID（相对路径）
     * @param {string} modulePath - 模块绝对路径
     * @returns {string} 模块 ID
     */
    getModuleId(modulePath) {
        return './' + path.relative(this.context, modulePath).replace(/\\/g, '/');
    }

    /**
     * 生成 chunk
     * 根据入口和代码分割规则生成 chunk
     */
    generateChunks() {
        // 创建入口 chunk
        const entryChunk = {
            name: 'main',
            modules: [],
            files: []
        };

        // 将所有模块添加到入口 chunk（简化实现）
        this.modules.forEach(module => {
            entryChunk.modules.push(module);
        });

        this.chunks.set('main', entryChunk);
    }

    /**
     * 生成最终代码
     * 将模块打包成可执行的 bundle
     */
    generateCode() {
        this.chunks.forEach((chunk, chunkName) => {
            // 生成模块映射对象
            const modulesCode = chunk.modules.map(module => {
                return `
    // ${module.id}
    "${module.id}": function(module, exports, __webpack_require__) {
${module.code}
    }`;
            }).join(',\n');

            // 获取入口模块 ID
            const entryModule = chunk.modules.find(m => m.isEntry);
            const entryModuleId = entryModule ? entryModule.id : './src/index.js';

            // 生成 bundle 代码（webpack5 runtime 简化版）
            const bundleCode = `
/******/ (() => { // webpackBootstrap
/******/     "use strict";
/******/     
/******/     // ========== webpack 模块缓存 ==========
/******/     var __webpack_module_cache__ = {};
/******/     
/******/     // ========== webpack require 函数 ==========
/******/     function __webpack_require__(moduleId) {
/******/         // 检查模块是否在缓存中
/******/         var cachedModule = __webpack_module_cache__[moduleId];
/******/         if (cachedModule !== undefined) {
/******/             return cachedModule.exports;
/******/         }
/******/         
/******/         // 创建新模块并放入缓存
/******/         var module = __webpack_module_cache__[moduleId] = {
/******/             id: moduleId,
/******/             loaded: false,
/******/             exports: {}
/******/         };
/******/         
/******/         // 执行模块函数
/******/         __webpack_modules__[moduleId].call(
/******/             module.exports,
/******/             module,
/******/             module.exports,
/******/             __webpack_require__
/******/         );
/******/         
/******/         // 标记模块已加载
/******/         module.loaded = true;
/******/         
/******/         // 返回模块导出
/******/         return module.exports;
/******/     }
/******/     
/******/     // ========== 模块定义 ==========
/******/     var __webpack_modules__ = {
${modulesCode}
/******/     };
/******/     
/******/     // ========== webpack5 ESM 兼容性处理 ==========
/******/     // 定义 __esModule 标记
/******/     __webpack_require__.r = (exports) => {
/******/         if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/             Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/         }
/******/         Object.defineProperty(exports, '__esModule', { value: true });
/******/     };
/******/     
/******/     // 定义导出属性的 getter
/******/     __webpack_require__.d = (exports, definition) => {
/******/         for (var key in definition) {
/******/             if (__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
/******/                 Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
/******/             }
/******/         }
/******/     };
/******/     
/******/     // hasOwnProperty 简写
/******/     __webpack_require__.o = (obj, prop) => Object.prototype.hasOwnProperty.call(obj, prop);
/******/     
/******/     // ========== 启动入口模块 ==========
/******/     var __webpack_exports__ = __webpack_require__("${entryModuleId}");
/******/     
/******/ })();
`;

            // 确定输出文件名
            const filename = this.output.filename.replace('[name]', chunkName);
            
            // 存储到 assets
            this.assets[filename] = bundleCode;
            chunk.files.push(filename);
        });
    }

    /**
     * 输出资源文件
     * @param {Object} compilation - 编译对象
     */
    emitAssets(compilation) {
        // 确保输出目录存在
        const outputPath = path.resolve(this.context, this.output.path);
        if (!fs.existsSync(outputPath)) {
            fs.mkdirSync(outputPath, { recursive: true });
        }

        // 写入所有资源文件
        Object.keys(this.assets).forEach(filename => {
            const filePath = path.join(outputPath, filename);
            fs.writeFileSync(filePath, this.assets[filename]);
            console.log(`Emit: ${filename}`);
        });
    }

    /**
     * 获取编译统计信息
     * @returns {Object} 统计信息
     */
    getStats() {
        return {
            modules: Array.from(this.modules.values()).map(m => ({
                id: m.id,
                size: m.code.length
            })),
            chunks: Array.from(this.chunks.values()).map(c => ({
                name: c.name,
                files: c.files,
                modules: c.modules.length
            })),
            assets: Object.keys(this.assets).map(name => ({
                name,
                size: this.assets[name].length
            })),
            time: Date.now()
        };
    }
}

module.exports = Compiler;
```

### 4. AST 解析器 (parser.js)

```js
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generator = require('@babel/generator').default;
const t = require('@babel/types');

/**
 * 解析模块代码
 * 使用 Babel 解析 AST，提取依赖并转换代码
 * 
 * @param {string} code - 源代码
 * @param {Object} options - 配置选项
 * @returns {Object} { code: 转换后的代码, dependencies: 依赖数组 }
 */
function parse(code, options = {}) {
    // 存储依赖路径
    const dependencies = [];

    // 1. 将代码解析为 AST（抽象语法树）
    const ast = parser.parse(code, {
        sourceType: 'module',  // 支持 ES Module
        plugins: [
            'jsx',             // 支持 JSX
            'typescript',      // 支持 TypeScript
            'classProperties', // 支持类属性
            'dynamicImport'    // 支持动态导入
        ]
    });

    // 2. 遍历 AST，收集依赖并转换代码
    traverse(ast, {
        // 处理 import 语句: import xxx from 'xxx'
        ImportDeclaration({ node }) {
            const source = node.source.value;
            dependencies.push(source);
        },

        // 处理 require 调用: require('xxx')
        CallExpression(path) {
            const { node } = path;
            
            // 检查是否为 require 调用
            if (node.callee.name === 'require' && 
                node.arguments.length === 1 &&
                t.isStringLiteral(node.arguments[0])) {
                
                const source = node.arguments[0].value;
                dependencies.push(source);
                
                // 将 require 转换为 __webpack_require__
                node.callee.name = '__webpack_require__';
            }
        },

        // 处理动态导入: import('xxx')
        Import(path) {
            const parent = path.parentPath.node;
            if (t.isCallExpression(parent) && 
                parent.arguments.length === 1 &&
                t.isStringLiteral(parent.arguments[0])) {
                
                const source = parent.arguments[0].value;
                dependencies.push(source);
                
                // 转换为 __webpack_require__.e 动态加载
                // 这里简化处理，实际 webpack 会生成更复杂的代码
            }
        },

        // 处理 export default
        ExportDefaultDeclaration(path) {
            const { node } = path;
            const declaration = node.declaration;

            // 转换 export default xxx 为 module.exports = xxx
            if (t.isIdentifier(declaration) || 
                t.isFunctionDeclaration(declaration) ||
                t.isClassDeclaration(declaration) ||
                t.isExpression(declaration)) {
                
                // 生成: __webpack_require__.r(__webpack_exports__);
                // 生成: __webpack_exports__["default"] = xxx;
            }
        },

        // 处理 export { xxx }
        ExportNamedDeclaration(path) {
            const { node } = path;
            
            if (node.source) {
                // export { xxx } from 'xxx' - 重导出
                dependencies.push(node.source.value);
            }
            
            // 转换命名导出为 __webpack_exports__.xxx = xxx
        }
    });

    // 3. 将 AST 转换回代码
    // 同时转换 ES Module 语法为 CommonJS
    const { code: transformedCode } = generator(ast, {
        comments: false,  // 移除注释
        compact: false    // 不压缩（便于阅读）
    });

    // 4. 包装模块代码（转换 import/export）
    const wrappedCode = transformESModule(transformedCode, dependencies);

    return {
        code: wrappedCode,
        dependencies
    };
}

/**
 * 转换 ES Module 语法
 * 将 import/export 转换为 webpack runtime 兼容的格式
 * 
 * @param {string} code - 原始代码
 * @param {Array} dependencies - 依赖列表
 * @returns {string} 转换后的代码
 */
function transformESModule(code, dependencies) {
    let result = code;

    // 简化实现：替换 import 语句
    // 实际 webpack 使用更复杂的 AST 转换
    
    // import xxx from 'xxx' -> const xxx = __webpack_require__('xxx')
    result = result.replace(
        /import\s+(\w+)\s+from\s+['"]([^'"]+)['"]/g,
        'const $1 = __webpack_require__("$2")'
    );

    // import { xxx } from 'xxx' -> const { xxx } = __webpack_require__('xxx')
    result = result.replace(
        /import\s+\{([^}]+)\}\s+from\s+['"]([^'"]+)['"]/g,
        'const {$1} = __webpack_require__("$2")'
    );

    // export default xxx -> module.exports = xxx
    result = result.replace(
        /export\s+default\s+/g,
        'module.exports = '
    );

    // export { xxx } -> exports.xxx = xxx
    result = result.replace(
        /export\s+\{([^}]+)\}/g,
        (match, exports) => {
            return exports.split(',')
                .map(e => e.trim())
                .map(e => `exports.${e} = ${e}`)
                .join(';\n');
        }
    );

    // export const/let/var xxx -> exports.xxx = xxx
    result = result.replace(
        /export\s+(const|let|var)\s+(\w+)/g,
        '$1 $2; exports.$2'
    );

    return result;
}

module.exports = { parse };
```

### 5. 使用示例

```js
// webpack.config.js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                use: ['babel-loader']
            }
        ]
    },
    plugins: []
};

// 运行打包
const webpack = require('./lib/index');
const config = require('./webpack.config');

const compiler = webpack(config);

compiler.run((err, stats) => {
    if (err) {
        console.error(err);
        return;
    }
    console.log('Build completed!');
    console.log(stats);
});
```

### 6. 核心概念总结

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
