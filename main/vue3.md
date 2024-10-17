# Vue3.5.11 API 参考

## 单文件组件 SFC

**SFC结构示例:**

```vue
<template>
  <!-- 注释 -->
  <div class="example">{{ msg }}</div>
</template>

<!--组合式-->
<script setup>
  import { ref } from 'vue'
</script>

<!--选项式-->
<!--<script>-->
<!--  export default {-->
<!--    data() {-->
<!--      return {-->
<!--        msg: 'Hello world!'-->
<!--      }-->
<!--    }-->
<!--  }-->
<!--</script>-->

<style scoped lang="less">
  .example {
    color: red;
  }
</style>

<doc>
  自定义区块, 如'doc'可以写文档的块等
</doc>

```

### `<template>` 

#### Class 与 Style 绑定
1. :class 和 :style 都可以接受字符串/对象/数组 值
2. :style 会自动添加浏览器特殊css前缀

#### 插值表达式内受限的全局访问
- 模板中的表达式访问到有限的全局对象。会暴露常用内置全局对象，如 Math 和 Date。

- 没有显式包含在列表中的全局对象将不能在模板内表达式中访问，可以在 app.config.globalProperties 全局显式添加。

```vue
<template>
   // class="active", 对象参数
   <div :class="{ active: true }"></div>
   // class="a b", 数组参数
   <div :class="['a', 'b']"></div>
   // class="a b", 三元表达式
   <div :class="[true ? 'a' : '', 'b']"></div>
   // 组件使用class会直接继承到根元素, 也可以使用$attrs.class 自定义使用的地方
   <MyComponent :class="{ active: isActive }" />
   // style对象值
   <div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
   // style数组会将属性合并
   <div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
</template>

```

### `<script setup>`
- 组件中`<script setup>` 中的顶层的导入、声明的变量和函数可在模板中直接使用。
- 使用 `<script setup>` 的单文件组件会自动根据文件名生成对应的 name 选项，无需再手动声明.

- 组件引用

```vue
<script setup>
  import * as Form from './form-components' // 组件集合加载, 命名空间组件
  import { FooBar as FooBarChild } from './components' // 递归组件调用
</script>

<template>
  <Form.Input> // 组件集合加载使用, 命名空间组件
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

- 顶层await,代码会被编译成 async setup()

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

- 与普通的 `<script>` 一起使用,两边无法互相访问,谨慎使用
```vue
<script>
// 普通 <script>，在模块作用域下执行 (仅一次)
runSideEffectOnce()

// 声明额外的选项
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// 在 setup() 作用域中执行 (对每个实例皆如此)
</script>
```

- `<script setup>` 不能使用 src 属性


### `<style>`

- 作用域
```vue
<style>
/* 全局样式 */
</style>

<style scoped>
/* 作用域样式样式,会为每个选择器添加 组件生成的随机的属性选择器 */
</style>
```

- 选择器
```vue
<!---->
<style scoped>
/*深度选择器,会影响子组件*/
.a :deep(.b) { /*编译为 .a[data-v-f3f3eg9] .b {} */ 
  /* ... */
}

/*插槽选择器, 影响插槽内的样式内容*/
:slotted(div) {
  color: red;
}

/*全局选择器, 影响全局, 即使在scope中依然可以影响全局*/
:global(.red) {
  color: red;
}
</style>
```

- CSS Modules & useCssModule()

```vue
<template>
  <p :class="$style.red">This should be red</p>
  <p :class="classes.red">red</p>
</template>

<script setup>
  import { useCssModule } from 'vue'

  // 默认情况下，返回 <style module> 的 class
  useCssModule()

  // 具名情况下，返回 <style module="classes"> 的 class
  useCssModule('classes')
  
</script>

<!--添加module属性, 组件中可以通过 $style 进行访问-->
<style module>
  .red {
    color: red;
  }
</style>

<!--可以自定义注入名称-->
<style module="classes">
.red {
  color: red;
}
</style>

```

- CSS v-bind

```vue
<!--组合式-->
<script setup>
  import { ref } from 'vue'
  const theme = ref({
      color: 'red',
  })
</script>

<template>
  <p>hello</p>
</template>

<style scoped>
  p {
    color: v-bind('theme.color');
  }
</style>
```

```vue
<!--选项式-->
<template>
  <div class="text">hello</div>
</template>

<script>
export default {
  data() {
    return {
      color: 'red'
    }
  }
}
</script>

<style>
.text {
  color: v-bind(color);
}
</style>
```

## app实例

### createApp()

创建一个应用实例。

- **示例**

  第一个参数是根组件。第二个参数可选，它是要传递给根组件的 props。
  可以直接内联根组件：

  ```js
  import { createApp } from 'vue'

  const app = createApp({
    /* 根组件选项 */
  })
  ```

  也可以使用从别处导入的组件：

  ```js
  import { createApp } from 'vue'
  import App from './App.vue'

  const app = createApp(App)
  ```

### createSSRApp()

创建SSR模式的应用实例。用法与 `createApp()` 完全相同。

### app.mount()

将应用实例挂载在一个容器元素中。

- **详细信息**

  参数可以是一个实际的 DOM 元素或一个 CSS 选择器 (使用第一个匹配到的元素)。返回根组件的实例。

  如果该组件有模板或定义了渲染函数，它将替换容器内所有现存的 DOM 节点。否则在运行时编译器可用的情况下，容器元素的 `innerHTML` 将被用作模板。

  在 SSR 激活模式下，它将激活容器内现有的 DOM 节点。如果出现了[激活不匹配](/guide/scaling-up/ssr#hydration-mismatch)，那么现有的 DOM 节点将会被修改以匹配客户端的实际渲染结果。

  对于每个应用实例，`mount()` 仅能调用一次。

- **示例**

  ```js
  import { createApp } from 'vue'
  const app = createApp(/* ... */)

  app.mount('#app')
  ```

  也可以挂载到一个实际的 DOM 元素。

  ```js
  app.mount(document.body.firstChild)
  ```

### app.unmount()

卸载一个已挂载的应用实例。卸载一个应用会触发该应用组件树内所有组件的卸载生命周期钩子。

- **类型**

  ```ts
  interface App {
    unmount(): void
  }
  ```

### app.onUnmount() 3.5+

注册一个回调函数，在应用卸载时调用。

- **类型**

  ```ts
  interface App {
    onUnmount(callback: () => any): void
  }
  ```

### app.component()

注册全局组件

- **示例**

  ```js
  import { createApp } from 'vue'

  const app = createApp({})

  // 注册一个选项对象
  app.component('my-component', {
    /* ... */
  })

  // 得到一个已注册的组件
  const MyComponent = app.component('my-component')
  ```

### app.directive()

注册全局指令

- **示例**

  ```js
  import { createApp } from 'vue'

  const app = createApp({
    /* ... */
  })

  // 注册（对象形式的指令）
  app.directive('my-directive', {
    /* 自定义指令钩子 */
  })

  // 注册（函数形式的指令）
  app.directive('my-directive', () => {
    /* ... */
  })

  // 得到一个已注册的指令
  const myDirective = app.directive('my-directive')
  ```

### app.use() 插件

安装一个插件

- **详细信息**

  第一个参数应是插件本身，可选的第二个参数是要传递给插件的选项。

  插件可以是一个带 `install()` 方法的对象，亦或直接是一个将被用作 `install()` 方法的函数。插件选项 (`app.use()` 的第二个参数) 将会传递给插件的 `install()` 方法。

  若 `app.use()` 对同一个插件多次调用，该插件只会被安装一次。

- **示例**

  ```js
  
  import { createApp } from 'vue'
  
  const app = createApp({})
  
  const myPlugin = {
     install(app, options) {
        // app 实例
        // 插件作用:
        // 通过 app.component() 和 app.directive() 注册一到多个全局组件或自定义指令。
        // 通过 app.provide() 使一个资源可被注入进整个应用。
        // 向 app.config.globalProperties 中添加一些全局实例属性或方法
     }
  }
  
  app.use(myPlugin, {
   // '参数option'
  })
  
  ```


### app.mixin()

注册全局 mixin.
Mixin 在 Vue 3 支持主要是为了向后兼容，不推荐使用,使用组合式函数代替.

- **类型**

  ```ts
  interface App {
    mixin(mixin: ComponentOptions): this
  }
  ```

### app.provide()

全局注入

- **详细信息**

  第一个参数应当是注入的 key，第二个参数则是提供的值。返回应用实例本身。

- **示例**

  ```js
  import { createApp } from 'vue'

  const app = createApp(/* ... */)

  app.provide('key', 'value')
  ```

  在应用的某个组件中：

  <div class="composition-api">

  ```js
  import { inject } from 'vue'

  export default {
    setup() {
      console.log(inject('key')) // 'hello'
    }
  }
  ```

  </div>
  <div class="options-api">

  ```js
  export default {
    inject: ['key'],
    created() {
      console.log(this.message) // 'hello'
    }
  }
  ```

  </div>

### app.runWithContext()

- 仅在 3.3+ 中支持

使用当前应用作为注入上下文执行回调函数, 即使没有当前活动的组件实例,回调函数中`inject()` 调用也可以从当前应用提供的值中查找注入。回调的返回值也将被返回。

- **示例**

  ```js
  import { inject } from 'vue'

  app.provide('id', 1)

  const injected = app.runWithContext(() => {
    return inject('id')
  })

  console.log(injected) // 1
  ```

### app.version

提供当前应用所使用的 Vue 版本号。

- **示例**

  在一个插件中对版本作判断：

  ```js
  export default {
    install(app) {
      const version = Number(app.version.split('.')[0])
      if (version < 3) {
        console.warn('This plugin requires Vue 3')
      }
    }
  }
  ```

### app.config

应用实例的配置项

```js
import { createApp } from 'vue'

const app = createApp(/* ... */)

console.log(app.config)
```

### app.config.errorHandler

用于为应用内抛出的未捕获错误指定一个全局处理函数。

- **详细信息**

  它可以从下面这些来源中捕获错误：

   - 组件渲染器
   - 事件处理器
   - 生命周期钩子
   - `setup()` 函数
   - 侦听器
   - 自定义指令钩子
   - 过渡 (Transition) 钩子

- **示例**

  ```js
  app.config.errorHandler = (err, instance, info) => {
    // error 错误对象
    // instance 产生错误的组件实例
    // info 错误信息
    // 处理错误，例如：报告给一个服务
  }
  ```

### app.config.warnHandler

用于为 Vue 的运行时警告指定一个自定义处理函数。 
警告仅会在开发阶段显示，因此在生产环境中，这条配置将被忽略。

- **示例**

  ```js
  app.config.warnHandler = (msg, instance, trace) => {
    // 警告处理器将接受警告信息作为其第一个参数，来源组件实例为第二个参数，以及组件追踪字符串作为第三个参数。
    // `trace` 是组件层级结构的追踪
  }
  ```

### app.config.performance

设置此项为 `true` 可以在浏览器开发工具的“性能/时间线”页中启用对组件初始化、编译、渲染和修补的性能表现追踪。
仅在开发模式和支持 performance API 的浏览器中工作。

- **类型**：`boolean`

### app.config.compilerOptions

配置运行时编译器的选项。也可通过compilerOptions 选项在每个组件的基础上覆盖这些选项。

此配置项仅在完整构建版本中可用。仅含运行时的 Vue 文件版本，必须通过构建工具的相关配置传递给 `@vue/compiler-dom`。

- `vue-loader`通过 `compilerOptions` loader 的选项传递

- `vite`通过 `@vitejs/plugin-vue` 的选项传递

#### app.config.compilerOptions.isCustomElement

用于指定一个检查方法来识别原生自定义元素。

- **类型** `(tag: string) => boolean`

- **详细信息**

  如果该标签需要当作原生自定义元素则应返回 `true`。对匹配到的标签，Vue 会将其渲染为原生元素而非将其视为一个 Vue 组件来解析。

  原生 HTML 和 SVG 标签不需要在此函数中进行匹配，Vue 的解析器会自动识别它们。

- **示例**

  ```js
  // 将所有标签前缀为 `ion-` 的标签视为自定义元素
  app.config.compilerOptions.isCustomElement = (tag) => {
    return tag.startsWith('ion-')
  }
  ```

#### app.config.compilerOptions.whitespace

用于调整模板中空格的处理行为。

- **类型** `'condense' | 'preserve'`

- **默认** `'condense'`

- **详细信息**

  Vue 移除/缩短了模板中的空格以求更高效的模板输出。默认的策略是“缩短”，表现行为如下：

   1. 元素中开头和结尾的空格字符将被缩短为一个空格。
   2. 包含换行的元素之间的空白字符会被删除。
   3. 文本节点中连续的空白字符被缩短成一个空格。

  设置该选项为 `'preserve'` 则会禁用 (2) 和 (3) 两项。

- **示例**

  ```js
  app.config.compilerOptions.whitespace = 'preserve'
  ```

#### app.config.compilerOptions.delimiters

用于调整模板内文本插值的分隔符。

- **类型** `[string, string]`

- **默认** `{{ "['\u007b\u007b', '\u007d\u007d']" }}`

- **详细信息**

  此项通常是为了避免与同样使用 mustache 语法的服务器端框架发生冲突。

- **示例**

  ```js
  // 分隔符改为ES6模板字符串样式
  app.config.compilerOptions.delimiters = ['${', '}']
  ```

#### app.config.compilerOptions.comments

用于调整是否移除模板中的 HTML 注释。

- **类型** `boolean`

- **默认** `false`

- **详细信息**

  默认情况下，Vue 会在生产环境移除所有注释，设置该项为 `true` 会强制 Vue 在生产环境也保留注释。在开发过程中，注释是始终被保留的。这个选项通常在 Vue 与其他依赖 HTML 注释的库一起使用时使用。

- **示例**

  ```js
  app.config.compilerOptions.comments = true
  ```

### app.config.globalProperties

注册能够被应用内所有组件实例访问到的全局属性的对象, 如果全局属性与组件自己的属性冲突，组件自己的属性将具有更高的优先级。
这是对 Vue 2 中 `Vue.prototype` 使用方式的一种替代.

- **用法**

  ```js
  app.config.globalProperties.msg = 'hello'
  ```

  这使得 `msg` 在应用的任意组件模板上都可用，并且也可以通过任意组件实例的 `this` 访问到：

  ```js
  export default {
    mounted() {
      console.log(this.msg) // 'hello'
    }
  }
  ```

### app.config.optionMergeStrategies

属性冲突时,用于定义组件选项的合并策略的对象。

- **示例**

  ```js
  const app = createApp({
    // 自身的选项
    msg: 'Vue',
    // 来自 mixin 的选项
    mixins: [
      {
        msg: 'Hello '
      }
    ],
    mounted() {
      // 在 this.$options 上暴露被合并的选项
      console.log(this.$options.msg)
    }
  })

  // 在 `app.config.optionMergeStrategies` 对象上以选项的名称作为 key，可以为一个自定义选项注册分配一个合并策略函数
  // 为 `msg` 定义一个合并策略函数
  app.config.optionMergeStrategies.msg = (parent, child) => {
    // 合并策略函数分别接受在父实例和子实例上定义的该选项的值作为第一和第二个参数。
    return (parent || '') + (child || '')
  }

  app.mount('#app')
  // 打印 'Hello Vue'
  ```

### app.config.idPrefix 3.5+

配置此应用中通过 useId() 生成的所有 ID 的前缀。

- **类型** `string`

- **默认值** `undefined`

- **示例**

  ```js
  app.config.idPrefix = 'my-app'
  ```

  ```js
  // 在组件中：
  const id1 = useId() // 'my-app:0'
  const id2 = useId() // 'my-app:1'
  ```

### app.config.throwUnhandledErrorInProduction 3.5+

强制在生产模式下抛出未处理的错误。

- **类型** `boolean`

- **默认值** `false`

- **详情**

  默认情况下，在 Vue 应用中抛出但未显式处理的错误在开发和生产模式下有不同的行为：

   - 在开发模式下，错误会被抛出并可能导致应用崩溃。这是为了使错误更加突出，以便在开发过程中被注意到并修复。

   - 在生产模式下，错误只会被记录到控制台以尽量减少对最终用户的影响。然而，这可能会导致只在生产中发生的错误无法被错误监控服务捕获。

  通过将 `app.config.throwUnhandledErrorInProduction` 设置为 `true`，即使在生产模式下也会抛出未处理的错误。

## 全局 API

### version

暴露当前所使用的 Vue 版本。

- **类型** `string`

- **示例**

  ```js
  import { version } from 'vue'

  console.log(version)
  ```

### nextTick()

等待下一次 DOM 更新刷新的工具方法。

- **类型**

  ```ts
  function nextTick(callback?: () => void): Promise<void>
  ```

- **详细信息**

  `nextTick()` 可以在状态改变后立即使用，以等待 DOM 更新完成。你可以传递一个回调函数作为参数，或者 await 返回的 Promise。

- **示例**

  ```vue
  <script setup>
  import { ref, nextTick } from 'vue'

  const count = ref(0)

  async function increment() {
    count.value++

    // DOM 还未更新
    console.log(document.getElementById('counter').textContent) // 0

    // 1. promise方式
    await nextTick()
    // DOM 此时已经更新
    console.log(document.getElementById('counter').textContent) // 1
  
    // 2. 回调方式
    // nextTick(()=>{
    //     console.log(document.getElementById('counter').textContent) // 1
    // })
  }
  </script>

  <template>
    <button id="counter" @click="increment">{{ count }}</button>
  </template>
  ```

### defineComponent() typescript组件类型推导

在定义 Vue 组件时提供类型推导的辅助函数。

- **类型**

  ```ts
  // 选项语法
  function defineComponent(
    component: ComponentOptions
  ): ComponentConstructor

  // 函数语法 (需要 3.3+)
  function defineComponent(
    setup: ComponentOptions['setup'],
    extraOptions?: ComponentOptions
  ): () => any
  ```

- **详细信息**

  第一个参数是一个组件选项对象,返回值将是该选项对象本身，该函数实际上没有任何操作，仅用于提供类型推导。

  注意返回值的类型有一点特别：它会是一个构造函数类型，它的实例类型是根据选项推断出的组件实例类型。这是为了能让该返回值在 TSX 中用作标签时提供类型推导支持。

  你可以像这样从 `defineComponent()` 的返回类型中提取出一个组件的实例类型 (与其选项中的 `this` 的类型等价)：

  ```ts
  const Foo = defineComponent(/* ... */)

  type FooInstance = InstanceType<typeof Foo>
  ```

  #### 函数签名

  - 仅在 3.3+ 中支持

  `defineComponent()` 还有一种备用签名，旨在与组合式 API 和渲染函数或 JSX一起使用。

  与传递选项对象不同的是，它需要传入一个函数。这个函数的工作方式与组合式 API 的 `setup()` 函数相同：它接收 props 和 setup 上下文。返回值应该是一个渲染函数——支持 `h()` 和 JSX：

  ```js
  import { ref, h } from 'vue'

  const Comp = defineComponent(
    (props) => {
      // 就像在 <script setup> 中一样使用组合式 API
      const count = ref(0)

      return () => {
        // 渲染函数或 JSX
        return h('div', count.value)
      }
    },
    // 其他选项，例如声明 props 和 emits。
    {
      props: {
        /* ... */
      }
    }
  )
  ```

  此签名的主要用例是使用 TypeScript (特别是使用 TSX)，因为它支持泛型：

  ```tsx
  const Comp = defineComponent(
    <T extends string | number>(props: { msg: T; list: T[] }) => {
      // 就像在 <script setup> 中一样使用组合式 API
      const count = ref(0)

      return () => {
        // 渲染函数或 JSX
        return <div>{count.value}</div>
      }
    },
    // 目前仍然需要手动声明运行时的 props
    {
      props: ['msg', 'list']
    }
  )
  ```

  在将来，我们计划提供一个 Babel 插件，自动推断并注入运行时 props (就像在单文件组件中的 `defineProps` 一样)，以便省略运行时 props 的声明。

  #### webpack Treeshaking 的注意事项

  因为 `defineComponent()` 是一个函数调用，所以它可能被某些构建工具认为会产生副作用，如 webpack。即使一个组件从未被使用，也有可能不被 tree-shake。

  为了告诉 webpack 这个函数调用可以被安全地 tree-shake，我们可以在函数调用之前添加一个 `/*#__PURE__*/` 形式的注释：

  ```js
  export default /*#__PURE__*/ defineComponent(/* ... */)
  ```

  使用 Vite 可以智能地确定 `defineComponent()` 实际上并没有副作用，所以无需手动注释。


## 响应式 API

### 响应式值改变触发监听方式

以下改变可触发监听:
1. 数组 下标修改,push/pop/shift/unshift/splice/sort/reverse,重新赋值
2. 对象 key修改, 重新赋值
3. 原始类型 重新赋值

### ref & shallowRef
组合式api推荐声明响应式状态方式,将值包装在特殊对象内实现响应式监听.包装对象能够将所有类型值转换为响应式,并能更好保持监听.
使用shallowRef实现浅相应.

```vue
<script setup>
   import { ref, shallowRef } from 'vue'

   const count = ref(0)

   console.log(count) // { value: 0 }
   console.log(count.value) // 0

   function increment() {
        count.value++
    }
   
    // 浅响应式 shallowRef, 只有对 .value 的访问是响应式的
   // 主要作用减少监听开销
   const state = shallowRef({ count: 1 })
   // 不会触发更改
   state.value.count = 2
   // 会触发更改
   state.value = { count: 2 }
    
</script>

<template>
   <button @click="increment">
    {{ count }} // 自动解包,即为count.value
    </button>
</template>
```

### triggerRef()

强制触发依赖于一个浅层 ref 的副作用，这通常在对浅引用的内部值进行深度变更后使用。

- **示例**

  ```js
  const shallow = shallowRef({
    greet: 'Hello, world'
  })

  // 触发该副作用第一次应该会打印 "Hello, world"
  watchEffect(() => {
    console.log(shallow.value.greet)
  })

  // 这次变更不应触发副作用，因为这个 ref 是浅层的
  shallow.value.greet = 'Hello, universe'

  // 打印 "Hello, universe"
  triggerRef(shallow)
  ```

### customRef()

创建一个自定义的 ref，显式声明对其依赖追踪和更新触发的控制方式。

- **类型**

  ```ts
  function customRef<T>(factory: CustomRefFactory<T>): Ref<T>

  type CustomRefFactory<T> = (
    track: () => void,
    trigger: () => void
  ) => {
    get: () => T
    set: (value: T) => void
  }
  ```

- **详细信息**

  `customRef()` 预期接收一个工厂函数作为参数，这个工厂函数接受 `track` 和 `trigger` 两个函数作为参数，并返回一个带有 `get` 和 `set` 方法的对象。

  一般来说，`track()` 应该在 `get()` 方法中调用，而 `trigger()` 应该在 `set()` 中调用。然而事实上，你对何时调用、是否应该调用他们有完全的控制权。

- **示例**

  创建一个防抖 ref，即只在最近一次 set 调用后的一段固定间隔后再调用：

  ```js
  import { customRef } from 'vue'

  export function useDebouncedRef(value, delay = 200) {
    let timeout
    return customRef((track, trigger) => {
      return {
        get() {
          track()
          return value
        },
        set(newValue) {
          clearTimeout(timeout)
          timeout = setTimeout(() => {
            value = newValue
            trigger()
          }, delay)
        }
      }
    })
  }
  ```

  在组件中使用：

  ```vue
  <script setup>
  import { useDebouncedRef } from './debouncedRef'
  const text = useDebouncedRef('hello')
  </script>

  <template>
    <input v-model="text" />
  </template>
  ```

  [在演练场中尝试一下](https://play.vuejs.org/#eNplUkFugzAQ/MqKC1SiIekxIpEq9QVV1BMXCguhBdsyaxqE/PcuGAhNfYGd3Z0ZDwzeq1K7zqB39OI205UiaJGMOieiapTUBAOYFt/wUxqRYf6OBVgotGzA30X5Bt59tX4iMilaAsIbwelxMfCvWNfSD+Gw3++fEhFHTpLFuCBsVJ0ScgUQjw6Az+VatY5PiroHo3IeaeHANlkrh7Qg1NBL43cILUmlMAfqVSXK40QUOSYmHAZHZO0KVkIZgu65kTnWp8Qb+4kHEXfjaDXkhd7DTTmuNZ7MsGyzDYbz5CgSgbdppOBFqqT4l0eX1gZDYOm057heOBQYRl81coZVg9LQWGr+IlrchYKAdJp9h0C6KkvUT3A6u8V1dq4ASqRgZnVnWg04/QWYNyYzC2rD5Y3/hkDgz8fY/cOT1ZjqizMZzGY3rDPC12KGZYyd3J26M8ny1KKx7c3X25q1c1wrZN3L9LCMWs/+AmeG6xI=)

  :::warning 谨慎使用
  当使用 customRef 时，我们应该谨慎对待其 getter 的返回值，尤其是在每次运行 getter 时都生成新对象数据类型的情况下。当这样的 customRef 作为 prop 传递时，将影响父组件和子组件之间的关系。

  父组件的渲染函数可能会被其他的响应式状态变化触发。在重新渲染过程中，我们会重新评估 customRef 的值，并返回一个新的对象数据类型作为子组件的 prop。这个 prop 会与其上一个值进行比较，由于两者不同，子组件中 customRef 的响应式依赖将被触发。与此同时，因为没有调用 customRef 的 setter，父组件中的响应式依赖不会运行。

  [在演练场中尝试一下](https://play.vuejs.org/#eNqFVEtP3DAQ/itTS9Vm1ZCt1J6WBZUiDvTQIsoNcwiOkzU4tmU7+9Aq/71jO1mCWuhlN/PyfPP45kAujCk2HSdLsnLMCuPBcd+Zc6pEa7T1cADWOa/bW17nYMPPtvRsDT3UVrcww+DZ0flStybpKSkWQQqPU0IVVUwr58FYvdvDWXgpu6ek1pqSHL0fS0vJw/z0xbN1jUPHY/Ys87Zkzzl4K5qG2zmcnUN2oAqg4T6bQ/wENKNXNk+CxWKsSlmLTSk7XlhedYxnWclYDiK+MkQCoK4wnVtnIiBJuuEJNA2qPof7hzkEoc8DXgg9yzYTBBFgNr4xyY4FbaK2p6qfI0iqFgtgulOe27HyQRy69Dk1JXY9C03JIeQ6wg4xWvJCqFpnlNytOcyC2wzYulQNr0Ao+Mhw0KnTTEttl/CIaIJiMz8NGBHFtYetVrPwa58/IL48Zag4N0ssquNYLYBoW16J0vOkC3VQtVqk7cG9QcHz1kj0QAlgVYkNMFk6d0bJ1pbGYKUkmtD42HmvFfi94WhOEiXwjUnBnlEz9OLTJwy5qCo44D4O7en71SIFjI/F9VuG4jEy/GHQKq5hQrJAKOc4uNVighBF5/cygS0GgOMoK+HQb7+EWvLdMM7weVIJy5kXWi0Rj+xaNRhLKRp1IvB9hxYegA6WJ1xkUe9PcF4e9a+suA3YwYiC5MQ79KlFUzw5rZCZEUtoRWuE5PaXCXmxtuWIkpJSSr39EXXHQcWYNWfP/9A/uV3QUXJjueN2E1ZhtPnSIqGS+er3T77D76Ox1VUn0fsd4y3HfewCxuT2vVMVwp74RbTX8WQI1dy5qx12xI1Fpa1K5AreeEHCCN8q/QXul+LrSC3s4nh93jltkVPDIYt5KJkcIKStCReo4rVQ/CZI6dyEzToCCJu7hAtry/1QH/qXncQB400KJwqPxZHxEyona0xS/E3rt1m9Ld1rZl+uhaxecRtP3EjtgddCyimtXyj9H/Ii3eId7uOGTkyk/wOEbQ9h)

  :::


### ref解包

就是自动添加 .value 把被包含的值取出来.

1. 在 reactive 对象中自动解包

作为响应式对象的属性被访问或修改时自动解包,就像一个普通的属性.
只有当嵌套在一个深层响应式对象内时，才会发生 ref 解包。当其作为浅层响应式对象shallowReactive的属性被访问时不会解包

```js
const count = ref(0)
const state = reactive({
  count
})

// 自动解包
console.log(state.count) // 0
state.count = 1
console.log(count.value) // 1

// 将新 ref 赋值给已有 ref 的属性，那么它会替换掉旧的 ref
const otherCount = ref(2)
state.count = otherCount
console.log(state.count) // 2
// 原始 ref 现在已经和 state.count 失去联系
console.log(count.value) // 1

```


2. 在reactive的数组或集合中不会解包


```js
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```

3. 在模板中顶级的 ref 属性自动解包

```js
// 顶级属性 count object id
const count = ref(0)
const object = { id: ref(1) }
const { id } = object

// 模板中
{{ object.id + 1 }} // 不会解包,object.id不是顶级属性且为表达式,会解析失败报错[object Object]1
{{ count + 1 }} // 自动解包,输出2
{{ id + 1 }} // 自动解包,输出2
{{ object.id }} // 自动解包,object.id 是文本插值的最终计算值不是表达式, 输出 1

```

### reactive & shallowReactive

声明响应式状态使对象本身具有响应性, 只接受对象类型. reactive() 返回的是一个原始对象的 Proxy，它和原始对象是不相等的.
使用shallowReactive() API 可以选择退出深层响应性.

```js
import { reactive, shallowReactive } from 'vue'

const state = reactive({ count: 0 })
console.log(state) // {count: 0}

// 浅监听
const shallowState = shallowReactive({count: 1})

// 原始对象与reactive响应对象
const raw = {}
const proxy = reactive(raw)
// 代理对象和原始对象不是全等的
console.log(proxy === raw) // false
// 在同一个对象上调用 reactive() 会返回相同的代理
console.log(reactive(raw) === proxy) // true
// 在一个代理上调用 reactive() 会返回它自己
console.log(reactive(proxy) === proxy) // true
// 响应式对象内的嵌套对象依然是代理
proxy.nested = raw
console.log(proxy.nested === raw) // false

```

**局限性：**

1. 有限的值类型：只能用于对象类型 (对象、数组和如 `Map`、`Set` 这样的[集合类型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects#keyed_collections))。它不能持有如 `string`、`number` 或 `boolean` 这样的[原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)。

2. 不能替换整个对象：响应式跟踪通过属性访问实现，必须始终保持对响应式对象的相同引用。替换整个对象会让第一个引用的响应性连接丢失：

   ```js
   let state = reactive({ count: 0 })

   // 上面的 ({ count: 0 }) 引用将不再被追踪
   // (响应性连接已丢失！)
   state = reactive({ count: 1 })
   ```

3. 对解构操作不友好：解构丢失响应性：

   ```js
   const state = reactive({ count: 0 })

   // 当解构时，count 已经与 state.count 断开连接
   let { count } = state
   // 不会影响原始的 state
   count++

   // 该函数接收到的是一个普通的数字
   // 并且无法追踪 state.count 的变化
   // 我们必须传入整个对象以保持响应性
   callSomeFunction(state.count)
   ```

### readonly()
接受一个对象 (不论是响应式还是普通的) 或是一个 ref，返回一个原值的只读代理.
只读代理是深层的：对任何嵌套属性的访问都将是只读的。它的 ref 解包行为与 reactive() 相同，但解包得到的值是只读的.

```js
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // 用来做响应性追踪
  console.log(copy.count)
})

// 更改源属性会触发其依赖的侦听器
original.count++

// 更改该只读副本将会失败，并会得到一个警告
copy.count++ // warning!
```

### shallowReadonly()

`readonly()` 的浅层作用形式, 不会自动解包ref属性

- **类型**

  ```ts
  function shallowReadonly<T extends object>(target: T): Readonly<T>
  ```

- **示例**

  ```js
  const state = shallowReadonly({
    foo: 1,
    nested: {
      bar: 2
    }
  })

  // 更改状态自身的属性会失败
  state.foo++

  // ...但可以更改下层嵌套对象
  isReadonly(state.nested) // false

  // 这是可以通过的
  state.nested.bar++
  ```

### computed 计算属性

1. 计算属性方法根据响应值是否变化而触发.
2. 已计算值形成缓存,当最终缓存值相同值,不会触发计算属性方法.
3. 不建议在计算属性方法内放副作用操作及修改响应式值,不能直接修改计算属性返回值,要修改就显示声明可修改的get/set.

```vue
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// 1. 计算属性返回是一个 ref, 在模板中也会自动解包
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})


// 2. 可修改的计算属性
const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
   // getter
   get() {
      return firstName.value + ' ' + lastName.value
   },
   // setter
   set(newValue) {
      // 注意：我们这里使用的是解构赋值语法
      [firstName.value, lastName.value] = newValue.split(' ')
   }
})
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>

```

### watch & watchEffect & watchPostEffect & watchSyncEffect & onWatcherCleanup

```vue
<script setup>
import { ref, watch, watchEffect, watchPostEffect, watchSyncEffect,onWatcherCleanup } from 'vue'

// 1. watch 仅在数据源确实改变时才会触发回调
// 监听方式:
const y = ref(0)
const x = ref(0)
const obj = reactive({ count: 0 })
// 单个 ref
watch(x, (newX) => {
   console.log(`x is ${newX}`)
})

// getter 函数
watch(
        () => x.value + y.value,
        (sum) => {
           console.log(`sum of x + y is: ${sum}`)
        }
)
watch(
        () => obj.count,
        (count) => {
           console.log(`Count is: ${count}`)
        }
)
// 错误，因为 watch() 得到的参数是一个 number
// watch(obj.count, (count) => {
//    console.log(`Count is: ${count}`)
// })


// 多个来源组成的数组
watch([x, () => y.value], ([newX, newY]) => {
   console.log(`x is ${newX} and y is ${newY}`)
})
   
// 2. watch 深度监听 deep ,在3.5+版本中值可为数字,代表监听到第几层
// 不以getter 函数形式监听的对象,默认深度监听
watch(
        obj,
        (newValue, oldValue) => {
        }
)
// 也可明文声明deep
watch(
        obj,
        (newValue, oldValue) => {
        },
        { deep: true }
)

// 3. watch 即时触发
watch(
        obj,
        (newValue, oldValue) => {
           // 立即执行，且当 `obj` 改变时再次执行
        },
        { immediate: true }
)

// 4. watch 一次性监听
watch(
        obj,
        (newValue, oldValue) => {
           // 当 `obj` 变化时，仅触发一次
        },
        { once: true }
)

// 5. onWatcherCleanup 3.5+版本,侦听器失效并准备重新运行时会被调用
// 必须在 watchEffect 或 watch 函数中同步执行期间调用,不能在异步函数的 await 语句之后调用它
watch(x, (newx) => {
   onWatcherCleanup(() => {
      // 终止过期请求等逻辑
   })
})

// 6. onCleanup 没有以上同步语法限制,在异步逻辑中也可以使用
watch(x, (newId, oldId, onCleanup) => {
   // ...
   onCleanup(() => {
      // 清理逻辑
   })
})

watchEffect((onCleanup) => {
   // ...
   onCleanup(() => {
      // 清理逻辑
   })
})

// 7. 访问更新后的DOM flush & watchPostEffect
// 设置flush为post可以在回调中访问到更新后的DOM
watch(x, callback, {
   flush: 'post'
})
watchEffect(callback, {
   flush: 'post'
})
// 在 Vue 更新后执行
watchPostEffect(() => {
    // ...
})

// 8. 同步侦听 flush & watchSyncEffect
// 同步侦听器不会进行批处理，每当检测到响应式数据发生变化时就会触发. 注意性能
watch(source, callback, {
   flush: 'sync'
})

watchEffect(callback, {
   flush: 'sync'
})

watchSyncEffect(() => {
   /* 在响应式数据变化时同步执行 */
})

// 9. watchEffect 无需写明监听对象,自动监听同步代码中被使用的响应式值
// 执行机制:
// 回调会立即执行，不需要指定 immediate: true
// 仅会在其同步执行期间，才追踪依赖。在使用异步回调时，只有在第一个 await 正常工作前访问到的属性才会被追踪
watchEffect(async () => {
   const response = await fetch(
           `https://xxxx/${y.value}`
   )
   x.value = await response.json()
})

// 10. 停止监听器
// 组件卸载,会自动停止
watchEffect(() => {})
// 组件卸载,异步监听器不会自动卸载,不与组件绑定
let unwatch = null;
setTimeout(() => {
   unwatch = watchEffect(() => {})
}, 100)
// 需手动卸载
unwatch()
   
</script>
```

### toRaw()

根据一个 Vue 创建的代理返回其原始对象。

- **类型**

  ```ts
  function toRaw<T>(proxy: T): T
  ```

- **详细信息**

  `toRaw()` 可以返回由 `reactive()`、`readonly()`、`shallowReactive()` 或者 `shallowReadonly()` 创建的代理对应的原始对象。

  这是一个可以用于临时读取而不引起代理访问/跟踪开销，或是写入而不触发更改的特殊方法。不建议保存对原始对象的持久引用，请谨慎使用。

- **示例**

  ```js
  const foo = {}
  const reactiveFoo = reactive(foo)

  console.log(toRaw(reactiveFoo) === foo) // true
  ```

### markRaw()

将一个对象标记为不可被转为代理。返回该对象本身。用于第三方实例不需要响应式时使用,节省开销.

- **类型**

  ```ts
  function markRaw<T extends object>(value: T): T
  ```

- **示例**

  ```js
  const foo = markRaw({})
  console.log(isReactive(reactive(foo))) // false

  // 也适用于嵌套在其他响应性对象
  const bar = reactive({ foo })
  console.log(isReactive(bar.foo)) // false
  ```
  使用注意:

  ```js
  const foo = markRaw({
    nested: {}
  })

  const bar = reactive({
    // 尽管 `foo` 被标记为了原始对象，但 foo.nested 却没有
    nested: foo.nested
  })

  console.log(foo.nested === bar.nested) // false
  ```


### effectScope()

创建 effect 作用域，捕获其中响应式副作用 (即计算属性和侦听器)，统一创建与销毁。

- **示例**

  ```js
  // 创建副作用作用域
  const scope = effectScope()

  scope.run(() => {
    const doubled = computed(() => counter.value * 2)

    watch(doubled, () => console.log(doubled.value))

    watchEffect(() => console.log('Count: ', doubled.value))
  })

  // 统一销毁作用域内的所有 effect
  scope.stop()
  ```

### getCurrentScope()

获取当前effectScope创建的作用域对象。

- **类型**

  ```ts
  function getCurrentScope(): EffectScope | undefined
  ```

### onScopeDispose()

在当前effectScope创建的所有没被停止的作用域对象中注册回调函数,当作用域对象停止时会调用这个回调函数。

如果在没有活跃的 effect 作用域的情况下调用此函数，将会抛出警告。在 3.5+ 版本中，可以通过将第二个参数设为 `true` 来消除此警告。

- **示例**

  ```js
    onScopeDispose(()=>{// 销毁处理},true/*可选参数 在已停止作用域中使用,防止抛出警告*/)
  ```

## 响应式 API 工具函数

### isRef()

检查某个值是否为 ref。

- **类型**

  ```ts
  function isRef<T>(r: Ref<T> | unknown): r is Ref<T>
  ```

  请注意，返回值是一个类型判定，这意味着 `isRef` 可以被用作类型守卫：

  ```ts
  let foo: unknown
  if (isRef(foo)) {
    // foo 的类型被收窄为了 Ref<unknown>
    foo.value
  }
  ```

### unref()

如果参数是 ref，则返回内部值，否则返回参数本身。这是 `val = isRef(val) ? val.value : val` 计算的一个语法糖。

- **类型**

  ```ts
  function unref<T>(ref: T | Ref<T>): T
  ```

- **示例**

  ```ts
  function useFoo(x: number | Ref<number>) {
    const unwrapped = unref(x)
    // unwrapped 现在保证为 number 类型
  }
  ```

### toRef()

可以将 值 / refs / getters 规范化为 refs (3.3+)。
保持值的响应性,主要作为props等数据传递给组合式函数使用依然保持响应式连接.
可为不存在属性使用toRef(),依然返回ref对象.

- **示例**

  1. 规范化签名 (3.3+)：

  ```js
  // 按原样返回现有的 ref
  toRef(existingRef)

  // 创建一个只读的 ref，当访问 .value 时会调用此 getter 函数
  toRef(() => props.foo)

  // 从非函数的值中创建普通的 ref
  // 等同于 ref(1)
  toRef(1)
  ```

  2. 对象属性连接：

  ```js
  const state = reactive({
    foo: 1,
    bar: 2
  })

  // 双向 ref，会与源属性同步
  const fooRef = toRef(state, 'foo')

  // 更改该 ref 会更新源属性
  fooRef.value++
  console.log(state.foo) // 2
  // 这里只是将数值转为ref, 不会形成数据连接
  // const fooRef = ref(state.foo)

  // 更改源属性也会更新该 ref
  state.foo++
  console.log(fooRef.value) // 3
  ```


3. 传递 prop 参数给组合式函数,保持数据响应式连接：

  ```vue
  <script setup>
  import { toRef } from 'vue'

  const props = defineProps(/* ... */)

  // 将 `props.foo` 转换为 ref，然后传入
  // 不能修改对应数据，修改props是不允许的,要修改就使用带有 `get` 和 `set` 的 `computed`替代。
  useSomeFeature(toRef(props, 'foo'))

  // getter 语法——推荐在 3.3+ 版本使用
  useSomeFeature(toRef(() => props.foo))
  </script>
  ```

### toRefs()

为响应式对象每个可枚举的属性创建数据连接.
主要用于组合式函数的参数传递.
不能为不存在属性创建响应式连接.

- **示例**

  ```js
  const state = reactive({
    foo: 1,
    bar: 2
  })
  
  // 1. 每个属性都形成响应式连接
  const stateAsRefs = toRefs(state)
  /*
  stateAsRefs 的类型：{
    foo: Ref<number>,
    bar: Ref<number>
  }
  */
  // 这个 ref 和源属性已经“链接上了”
  state.foo++
  console.log(stateAsRefs.foo.value) // 2
  stateAsRefs.foo.value++
  console.log(state.foo) // 3
  
  // 2. 解构也不会失去响应式
  const { foo, bar } = stateAsRefs;
  ```

### toValue() 3.3+

将值、refs 或 getters 转化为值。
可在组合式函数中使用，传递规范化的值、ref 或 getter 的参数。

- **示例**

  ```js
  toValue(1) //       --> 1 普通值
  toValue(ref(1)) //  --> 1 自动解包
  toValue(() => 1) // --> 1 getter
  ```

  在组合式函数中规范化参数：

  ```ts
  import type { MaybeRefOrGetter } from 'vue'

  function useFeature(id: MaybeRefOrGetter<number>) {
    watch(() => toValue(id), id => {
      // 处理 id 变更
    })
  }

  // 这个组合式函数支持以下的任意形式, 传递的值都是 1 ：
  useFeature(1)
  useFeature(ref(1))
  useFeature(() => 1)
  ```

### isProxy()

检查一个对象是否是由 `reactive()`、`readonly()`、`shallowReactive()` 或 `shallowReadonly()` 创建的代理。

- **类型**

  ```ts
  function isProxy(value: any): boolean
  ```

### isReactive()

检查一个对象是否是由 `reactive()` 或 `shallowReactive()` 创建的代理。

- **类型**

  ```ts
  function isReactive(value: unknown): boolean
  ```

### isReadonly()

检查传入的值是否为只读对象。只读对象的属性可以更改，但他们不能通过传入的对象直接赋值。

通过 `readonly()` 和 `shallowReadonly()` 创建的代理都是只读的，因为他们是没有 `set` 函数的 `computed()` ref。

- **类型**

  ```ts
  function isReadonly(value: unknown): boolean
  ```


## 组合式 API

### useAttrs()

用于 `<script setup>` 中返回 `attrs` 对象,对应$attrs

### useSlots()

用于 `<script setup>` 中返回 `slots` 对象,对应$slots


### useModel() 3.4+

是驱动 `defineModel()`的底层辅助函数.
选项式组件使用`useModel()`, 组合式组件应使用 `defineModel()`。

- **示例**

```vue
// child.vue
  <script setup>
    export default {
      props: ['count'],
      emits: ['update:count'],
      setup(props) {
        // 第一个参数是 `props` 对象，第二个参数是 model 名称,第三个参数可以用于为生成的 model ref 声明自定义的 getter 和 setter.
        const msg = useModel(props, 'count')
        msg.value = 1
      }
    }
  </script>
  <template>
    <input type="text" v-model="msg" />
  </template>
```

```vue
// parent.vue
<template>
  <Child v-model:count="test" />
</template>
```

### useId() 3.5+

主要让服务端渲染的组件ID与浏览器产生的ID一致。

- **类型**

  ```ts
  function useId(): string
  ```

- **示例**

  ```vue
  <script setup>
  import { useId } from 'vue'

  const id = useId()
  </script>

  <template>
    <form>
      <label :for="id">Name:</label>
      <input :id="id" type="text" />
    </form>
  </template>
  ```

- **详细信息**

  `useId()` 生成的每个 ID 在每个应用内都是唯一的。它可以用于为表单元素和无障碍属性生成 ID。在同一个组件中多次调用会生成不同的 ID；同一个组件的多个实例调用 `useId()` 也会生成不同的 ID。

  `useId()` 生成的 ID 在服务器端和客户端渲染之间是稳定的，因此可以安全地在 SSR 应用中使用，不会导致激活不匹配。

  如果同一页面上有多个 Vue 应用实例，可以通过 [`app.config.idPrefix`](/api/application#app-config-idprefix) 为每个应用提供一个 ID 前缀，以避免 ID 冲突。


## 生命周期

**流程图:**

![](https://cn.vuejs.org/assets/lifecycle_zh-CN.W0MNXI0C.png)

```vue
<script setup>
import { 
   onBeforeMount,
   onMounted,
   onBeforeUpdate,
   onUpdated,
   onBeforeUnmount,
   onUnmounted,
   onActivated,
   onDeactivated,
   onErrorCaptured,
   onRenderTracked,
   onRenderTriggered,
   onServerPrefetch,
   onServerPrefetch 
} from 'vue'

// 在组件被挂载之前被调用, DOM节点还未创建
// 在服务器端渲染期间不会被调用
onBeforeMount(()=>{
    // ...
})

// 在组件挂载完成后执行,完成以下两步
// 1. 所有同步子组件都已经被挂载 (不包含异步组件或 <Suspense> 树内的组件)
// 2. 自身的 DOM 树已经创建完成并插入了父容器中
// 在服务器端渲染期间不会被调用
onMounted(() => {
    // 通常用于访问/操作DOM 
})

// 在组件响应式状态变更而更新其 DOM 树之前调用
// 在服务器端渲染期间不会被调用
onBeforeUpdate(()=>{
    // 用来在 Vue 更新 DOM 之前访问 DOM 状态
})

// 在组件的响应式状态变更而更新 DOM 树之后调用
// 在服务器端渲染期间不会被调用
onUpdated(() => {
   // 访问更新后的 DOM
})

// 在组件实例被卸载之前调用,组件实例依然还保有全部的功能
// 在服务器端渲染期间不会被调用
onBeforeUnmount(()=>{
    // ...
})

// 组件实例被卸载之后调用
// 在服务器端渲染期间不会被调用
onUnmounted(() => {
    // 清除定时器/订阅等
})

// 组件实例是 <KeepAlive> 缓存树的一部分，当组件被插入到 DOM 中时调用
// 服务器端渲染期间不会被调用
onActivated(()=>{
})

// 组件实例是 <KeepAlive> 缓存树的一部分，当组件从 DOM 中被移除时调用
// 服务器端渲染期间不会被调用
onDeactivated(()=>{
})

// 在捕获了后代组件传递的错误时调用
// 捕获以下错误:
// 组件渲染
// 事件处理器
// 生命周期钩子
// setup() 函数
// 侦听器
// 自定义指令钩子
// 过渡钩子
onErrorCaptured((err, instance, info)=>{
   // err 错误对象
   // instance 触发该错误组件实例
   // info 错误信息
    
   // 错误传递方式:
   // 1. 正常错误将逐层传递onErrorCaptured,最终到达全局错误处理app.config.errorHandler
   // 2. 返回false将不再向上传递错误
   // 3. 当onErrorCaptured出现错误,将直接传递给全局错误处理app.config.errorHandler
   
   // 返回false,就停止向上传递错误
    return false;
})

// 当组件渲染过程中追踪到响应式依赖时调用
// 仅在开发模式下可用，且在服务器端渲染期间不会被调用
onRenderTracked(()=>{
})

// 当响应式依赖的变更触发了组件渲染时调用
// 仅在开发模式下可用，且在服务器端渲染期间不会被调用
onRenderTriggered(()=>{
})

// 在组件实例在服务器上被渲染之前调用
// 钩子如果返回了一个 Promise，服务端渲染会在渲染该组件前等待该 Promise 完成
onServerPrefetch(async()=>{
   // 主要用于渲染前,数据获取等
   data.value = await fetchOnServer(/* ... */)
})
   
</script>
```

## 指令

### v-show

场景: 显示隐藏经常切换
原理: 修改元素display值,开销低

### v-if
场景: 显示隐藏不经常切换
原理: 根据值决定是否渲染元素,开销高

注意: v-if与v-for不能一起使用, vue3版本 v-if优先于 v-for执行.

```vue
// 可作用于template, v-show不可作用于template
<template v-if="true">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

### v-lese
前面需要有v-if块,否则不能使用

### v-if-else
前面需要有v-if或v-if-else块,否则不能使用

### v-for

- **期望的绑定值类型：**`Array | Object | number | string | Iterable`

```vue
<template>
   // 1. 数组值
   <div v-for="item in items" :key="item.id">
      {{ item.message }}
   </div>
   // 解构
   <div v-for="{ message } in items" :key="item.id">
      {{ message }}
   </div>
   // 解构
   <div v-for="({ message }, index) in items" :key="item.id">
      {{ message }} {{ index }}
   </div>
   // 父级作用域
   <div v-for="item in items" :key="item.id">
     <span v-for="childItem in item.children">
       {{ item.message }} {{ childItem }}
     </span>
   </div>
   
   // 2. of语法
   <div v-for="item of items" :key="item.id"></div>
   
   // 3. 对象值
   <div v-for="(value, key, index) in myObject" :key="value.id">
      {{ index }}. {{ key }}: {{ value }}
   </div>
   
   // 4. 数值
   <div v-for="item in 10" :key="item"></div>
</template>
```

### v-on

```vue
<script setup>
   import { ref } from 'vue'
   const name = ref('Vue.js')

   function greet(event) {
      // `event` 是 DOM 原生事件
      alert(`Hello ${name.value}!`)
      if (event) {
         alert(event.target.tagName)
      }
   }
   
   function say(message) {
      alert(message)
   }

   function warn(message, event) {
      // 这里可以访问原生事件
      if (event) {
         event.preventDefault()
      }
      alert(message)
   }
</script>

<template>
   <div>
      <button @click="greet">Greet</button>
      
      <button @click="say('hello')">Say hello</button>
      
      <!-- 使用特殊的 $event 变量 -->
      <button @click="warn('Form cannot be submitted yet.', $event)">
         Submit
      </button>

      <!-- 使用内联箭头函数 -->
      <button @click="(event) => warn('Form cannot be submitted yet.', event)">
         Submit
      </button>

      <!-- 停止事件冒泡 -->
      <a @click.stop="doThis"></a>

      <!-- 阻止默认行为 -->
      <form @submit.prevent="onSubmit"></form>

      <!-- 停止冒泡和默认行为 -->
      <a @click.stop.prevent="doThat"></a>

      <!-- 也可以只有修饰符 -->
      <form @submit.prevent></form>

      <!-- 事件来自元素本身 -->
      <div @click.self="doThat">...</div>

      <!-- 添加事件监听器时，使用 `capture` 捕获模式 -->
      <!-- 内部元素触发的事件，在被内部元素处理前，先被外部处理 -->
      <div @click.capture="doThis">...</div>

      <!-- 点击事件最多被触发一次 -->
      <a @click.once="doThis"></a>

      <!-- 滚动事件的默认行为 (scrolling) 将立即发生而非等待 `onScroll` 完成 -->
      <!-- 立即触发,不会等待事件结束 -->
      <div @scroll.passive="onScroll">...</div>

      <!-- 键盘修饰符  KeyboardEvent.key中的值作为修饰符,需要转为 kebab-case 形式 -->
      <input @keyup.enter="submit" />
      <input @keyup.page-down="onPageDown" />
      <!-- Alt + Enter -->
      <input @keyup.alt.enter="clear" />
      <!-- Ctrl + 点击 -->
      <div @click.ctrl="doSomething">Do something</div>

      <!-- exact 精确按键修饰符 -->
      <!-- 仅当按下 Ctrl 且未按任何其他键时才会触发 -->
      <button @click.ctrl.exact="onCtrlClick">A</button>
      <!-- 仅当没有按下任何系统按键时触发 -->
      <button @click.exact="onClick">A</button>
   </div>
</template>
```
### v-model

双向绑定:
1. 文本类型的 `<input>` 和 `<textarea>` 元素会绑定 `value` property 并侦听 `input 事件`
2. `<input type="checkbox">` 和 `<input type="radio">` 会绑定 `checked` property 并侦听 `change 事件`
3. `<select>` 会绑定 `value` property 并侦听 `change 事件`。


```vue
<script setup>
import { ref } from 'vue'
</script>

<template>
   <div>
      // 1. 普通使用
      <input v-model="text">
      // 本质是以下语法的语法糖
      <input :value="text" @input="event => text = event.target.value">

      // 2. radio 使用与值绑定
      <input type="radio" id="one" value="One" v-model="picked" />
      // 被选中时pick值为first
      <input type="radio" v-model="pick" :value="first" />

      // 3. checkbox 使用与值绑定
      <input type="checkbox" id="checkbox" v-model="checked" />
      // true-value 和 false-value 和 v-model 配套使用。 toggle 值会在选中时被设为 'yes'，取消选择时设为 'no'
      <input
              type="checkbox"
              v-model="toggle"
              true-value="yes"
              false-value="no" />

      <textarea v-model="text"></textarea>

      // 4. select 使用与值绑定
      <select v-model="selected">
         <option disabled value="">Please select one</option>
         <option>A</option>
         <option>B</option>
         <option>C</option>
      </select>
      // 被选中时selected值为{ number: 123 }
      <select v-model="selected">
         <!-- 内联对象字面量 -->
         <option :value="{ number: 123 }">123</option>
      </select>

      // 5. .lazy 修饰符来改为在每次 change 事件后更新数据
      <input v-model.lazy="msg" />
      
      // 6. .number 用户输入自动转换为数字
      <input v-model.number="age" />
      
      // 7. .trim 自动去除用户输入内容中两端的空格
      <input v-model.trim="msg" />
   </div>
</template>

```
### 组件v-model & defineModel 3.4+

+ defineModel() 返回一个ref,可以直接使用在子组件v-model中与父组件的值相互绑定
+ defineModel() 返回值的 .value 和父组件的 v-model 的值同步,模板中使用会自动解包

```vue
// parent.vue

<script setup>
import { ref } from 'vue'
let countModel = ref();
</script>

<template>
   // 1. 默认使用
   <Child v-model="countModel" />
   
   // 2. 底层使用
   <Child :modelValue="countModel" @update:modelValue="$event => (countModel = $event)"/>
   
   // 3. 必填与默认值
   <Child v-model="" />
   
   // 4. v-model参数使用
   <Child v-model:title="countModel" />
   
   // 5. 多个v-model参数使用
   <Child v-model:first-name="first" v-model:last-name="last"/>
   
   // 6. 修饰符使用
   <Child v-model.test="countModel" v-model:last-name.uppercase="last" />
</template>

```

```vue
// child.vue
<!-- Child.vue -->
<script setup>
   import { defineModel, defineProps, defineEmits } from 'vue'
   // 1. 默认使用
   const model = defineModel()
   function update() {
      model.value++
   }
   
   // 2.底层使用
   const props = defineProps(['modelValue'])
   const emit = defineEmits(['update:modelValue'])
   
   // 3. 必填与默认值
   // 使 v-model 必填/默认值
   // 注意: 设置默认值后,父组件如不传参,子组件会自动使用默认值,导致父子组件数据不一致
   const model1 = defineModel({ required: true, default: 0 })

   // 4. v-model参数使用
   const title = defineModel('title')
   const title = defineModel('title', { required: true })
   // defineModel 不能使用的版本写法
   defineProps({ title: { required: true }})
   defineEmits(['update:title'])
   
   // 5. 多个v-model参数使用
   const firstName = defineModel('firstName')
   const lastName = defineModel('lastName')
   
   // 6. 修饰符使用
   const [model, modifiers] = defineModel()
   console.log(modifiers) // { test: true }
   const [lastName, lastNameModifiers] = defineModel('lastName')
   console.log(lastNameModifiers) // { uppercase: true }
   
   // 不适用defineModel的版本使用
   const props = defineProps({
      modelValue: String,
      modifiers: { default: ()=>{} },
      lastName: String,
      lastNameModifiers: { default: () => ({}) }
   })
   defineEmits(['update:modelValue', 'update:lastName'])
   
   // 7. defineModel get/set使用
   // 用于对model值的存储与显示逻辑深度加工/控制
   const [model, modifiers] = defineModel({
      set(value) { // 修改存储值
          // value是用户输入的值
         
         // 函数内可以访问modifiers对象
         if (modifiers.capitalize) {
            return value.charAt(0).toUpperCase() + value.slice(1)
         }
         
         // 返回值为需要存储和传递给get的值
         return value
      },
      get(value) { // 获取显示值
          // value 是通过set函数返回的值
         
         // 返回值作为显示的值
          return value
      }
   })


</script>

<template>
   // 1. 默认使用
   <div>自动解包model值: {{ model }}</div>
   <button @click="update">Increment</button>
   <input type="text" v-model="model">
   
   // 2. 底层使用
   <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)"/>
   
   // 3. 必填与默认值
   <input type="text" v-model="model1">
   
   // 4. v-model参数使用
   <input type="text" v-model="title" />
   <input type="text" :value="title" @input="$emit('update:title', $event.target.value)"/>
   
   // 5. 多个v-model的使用
   <input type="text" v-model="firstName" />
   <input type="text" v-model="lastName" />
   
   // 6. 修饰符使用
   <input type="text" v-model="model">
   <input type="text" v-model="lastName">
</template>

```
### v-slot & 插槽 <slot> 

```vue
// child.vue
<script setup>
   import { ref } from 'vue'
   let count = ref(0)
   let arg = ref({a:1})
</script>

<template>
   <div class="container">
      <header>
         <slot name="header"></slot>
      </header>
      <main>
         <slot></slot>
      </main>
      <footer>
         <slot name="footer"></slot>
      </footer>
      
      // 作用域插槽
      <slot name="data" :arg="arg" :count="count"></slot>
   </div>
</template>

```

```vue
// parent.vue
<template>
    <child>
       // 1. 使用插槽
       <template v-slot:header>
          <!-- header 插槽的内容放这里 -->
       </template>
       <template v-slot:default> // 默认插槽 也可不写v-slot:default
          <!-- header 插槽的内容放这里 -->
       </template>
       <template #footer> // v-slot简写
          <!-- footer 插槽的内容放这里 -->
       </template>
       
       // 2. 条件插槽 $slots 包含所有插槽对象
       <div v-if="$slots.header" class="card-header">
          <slot name="header" />
       </div>
       <div v-if="$slots.default" class="card-content">
          <slot />
       </div>
       <div v-if="$slots.footer" class="card-footer">
          <slot name="footer" />
       </div>
       
       // 3. 动态插槽
       <template v-slot:[dynamicSlotName]>
          ...
       </template>
       <!-- 缩写为 -->
       <template #[dynamicSlotName]>
          ...
       </template>
       
       // 4. 作用域插槽 获取子组件传递过来的数据
       <template #data="slotData">
          <div>{{slotData.arg.a}}</div> // 1
          <div>{{slotData.count}}</div> // 0
       </template>
    </child>
</template>

```

### v-text

更新元素的文本内容,通过设置元素的 textContent属性来工作，因此它将覆盖元素中所有现有的内容

- **期望的绑定值类型：**`string`

- **示例**

  ```vue-html
  <span v-text="msg"></span>
  <!-- 等同于 -->
  <span>{{msg}}</span>
  ```

### v-html

更新元素的 innerHTML.
注意:
1. 在你的站点上动态渲染任意的 HTML 是非常危险的，因为它很容易导致 [XSS 攻击](https://en.wikipedia.org/wiki/Cross-site_scripting)。请只对可信内容使用 HTML 插值，**绝不要**将用户提供的内容作为插值
2. 在单文件组件css 的 `scoped` 样式将不会作用于 `v-html` 里的内容，因为 HTML 内容不会被 Vue 的模板编译器解析。
3. 如果你想让 `v-html` 的内容也支持 scoped CSS，可使用 CSS modules 或使用一个额外的全局 `<style>` 元素，手动设置类似 BEM 的作用域策略。

- **期望的绑定值类型：**`string`

- **示例**

  ```vue-html
  <div v-html="html"></div>
  ```

### v-bind

动态的绑定一个或多个 attribute，也可以是组件的 prop。
属性命名冲突在处理绑定时，Vue 默认会利用 `in` 操作符来检查该元素上是否有同名的 DOM property,如果同名它会作为 DOM property 赋值，而不是作为 attribute 设置。

- **缩写：**
  - `:` 或者 `.` (当使用 `.prop` 修饰符)
  - 值可以省略 (当 attribute 和绑定的值同名时，需要 3.4+ 版本)

- **期望：**`any (带参数) | Object (不带参数)`

- **参数：**`attrOrProp (可选的)`

- **修饰符**

  - `.camel` - 将短横线命名的 attribute 转变为驼峰式命名。
  - `.prop` - 强制绑定为 DOM property (3.2+)。
  - `.attr` - 强制绑定为 DOM attribute (3.2+)。


- **示例**

  ```vue-html
  <!-- 绑定 attribute -->
  <img v-bind:src="imageSrc" />

  <!-- 动态 attribute 名 -->
  <button v-bind:[key]="value"></button>

  <!-- 缩写 -->
  <img :src="imageSrc" />

  <!-- 缩写形式的动态 attribute 名 (3.4+)，扩展为 :src="src" -->
  <img :src />

  <!-- 动态 attribute 名的缩写 -->
  <button :[key]="value"></button>

  <!-- 内联字符串拼接 -->
  <img :src="'/path/to/images/' + fileName" />

  <!-- class 绑定 -->
  <div :class="{ red: isRed }"></div>
  <div :class="[classA, classB]"></div>
  <div :class="[classA, { classB: isB, classC: isC }]"></div>

  <!-- style 绑定 -->
  <div :style="{ fontSize: size + 'px' }"></div>
  <div :style="[styleObjectA, styleObjectB]"></div>

  <!-- 绑定对象形式的 attribute -->
  <div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

  <!-- prop 绑定。“prop” 必须在子组件中已声明。 -->
  <MyComponent :prop="someThing" />

  <!-- 传递子父组件共有的 prop -->
  <MyComponent v-bind="$props" />

  <!-- XLink -->
  <svg><a :xlink:special="foo"></a></svg>
  ```


  ```vue-html
  // .prop 修饰符也有专门的缩写，`.`：
  // 强制绑定为DOM property
  <div :someProperty.prop="someObject"></div>
  <!-- 等同于 -->
  <div .someProperty="someObject"></div>
  
  // 强制绑定为DOM attribute
  <div :someProperty.attr="someObject"></div>
  
  // 当在 DOM 内模板使用 `.camel` 修饰符，可以驼峰化 `v-bind` attribute 的名称，例如 SVG `viewBox` attribute：
  <svg :view-box.camel="viewBox"></svg>
  ```


### v-pre

跳过该元素及其所有子元素的编译。

- **无需传入**

- **详细信息**

  元素内具有 `v-pre`，所有 Vue 模板语法都会被保留并按原样渲染。最常见的用例就是显示原始双大括号标签及内容。

- **示例**

  ```vue-html
  <span v-pre>{{ this will not be compiled }}</span>
  ```

### v-once

仅渲染元素和组件一次，并跳过之后的更新。

- **无需传入**

- **详细信息**

  在随后的重新渲染，元素/组件及其所有子项将被当作静态内容并跳过渲染。这可以用来优化更新时的性能。

  ```vue-html
  <!-- 单个元素 -->
  <span v-once>This will never change: {{msg}}</span>
  <!-- 带有子元素的元素 -->
  <div v-once>
    <h1>Comment</h1>
    <p>{{msg}}</p>
  </div>
  <!-- 组件 -->
  <MyComponent v-once :comment="msg" />
  <!-- `v-for` 指令 -->
  <ul>
    <li v-for="i in list" v-once>{{i}}</li>
  </ul>
  ```

  从 3.2 起，你也可以搭配 [`v-memo`](#v-memo) 的无效条件来缓存部分模板。

### v-memo

- 仅在 3.2+ 中支持

- **期望的绑定值类型：**`any[]`

- **详细信息**

  缓存一个模板的子树,传入依赖数组,数组值不变则缓存内容不更新.

  ```vue-html
  // 当组件重新渲染，如果 `valueA` 和 `valueB` 都保持不变，这个 `<div>` 及其子项的所有更新都将被跳过。实际上，甚至虚拟 DOM 的 vnode 创建也将被跳过，因为缓存的子树副本可以被重新使用。
  <div v-memo="[valueA, valueB]">
    ...
  </div>
  ```

  **与 `v-for` 一起使用**
  当搭配 `v-for` 使用 `v-memo`，确保两者都绑定在同一个元素上。**`v-memo` 不能用在 `v-for` 内部。

  ```vue-html
  // `v-memo` 用在这里本质上是在说“只有当该项的被选中状态改变时才需要更新”。
  <div v-for="item in list" :key="item.id" v-memo="[item.id === selected]">
    <p>ID: {{ item.id }} - selected: {{ item.id === selected }}</p>
    <p>...more child nodes</p>
  </div>
  ```


### v-cloak

用于隐藏尚未完成编译的 DOM 模板。

- **无需传入**

- **详细信息**

  **该指令只在没有构建步骤的环境下需要使用。**

  当使用直接在 DOM 中书写的模板时，可能会出现一种叫做“未编译模板闪现”的情况：用户可能先看到的是还没编译完成的双大括号标签，直到挂载的组件将它们替换为实际渲染的内容。

  `v-cloak` 会保留在所绑定的元素上，直到相关组件实例被挂载后才移除。配合像 `[v-cloak] { display: none }` 这样的 CSS 规则，它可以在组件编译完毕前隐藏原始模板。

- **示例**

  ```css
  [v-cloak] {
    display: none;
  }
  ```

  ```vue-html
  <div v-cloak>
    {{ message }}
  </div>
  ```

  直到编译完成前，`<div>` 将不可见。


### 自定义指令

+ 注意:
1. 不推荐在组件上使用自定义指令,因为可能是多个根节点
2. 指令回调中除了 el 外，其他参数都是只读的

```vue
// 组合式写法
<script setup>
import { ref } from 'vue'
// 完整自定义指令
const vDirective = {
   // 在绑定元素的 attribute 前
   // 或事件监听器应用前调用
   created(el, binding, vnode) {
   },
   // 在元素被插入到 DOM 前调用
   beforeMount(el, binding, vnode) {},
   // 在绑定元素的父组件
   // 及他自己的所有子节点都挂载完成后调用
   mounted(el, binding, vnode) {},
   // 绑定元素的父组件更新前调用
   beforeUpdate(el, binding, vnode, prevVnode) {},
   // 在绑定元素的父组件
   // 及他自己的所有子节点都更新后调用
   updated(el, binding, vnode, prevVnode) {
      // el: 元素DOM
      // binding: { arg: 'foo', modifiers: { bar: true },value: 'baz 的值',oldValue: '上一次更新时 baz 的值', instance: '使用该指令的组件实例', dir: '指令的定义对象'} 
      // vnode: 代表绑定元素的底层 VNode
      // prevVnode：代表之前的渲染中指令所绑定元素的 VNode
   },
   // 绑定元素的父组件卸载前调用
   beforeUnmount(el, binding, vnode) {},
   // 绑定元素的父组件卸载后调用
   unmounted(el, binding, vnode) {}
}
</script>

<template>
   <div v-directive:foo.bar="baz"> </div>
</template>
```

```vue
// 选项式写法 directives
export default {
   setup() {
        /*...*/
   },
   directives: {
      // 在模板中启用 v-focus
      focus: {
      /* ... */
      }
   }
}
```

```js
// 全局注册指令
// 简写 回调会在 `mounted` 和 `updated` 时都调用
app.directive('color', (el, binding) => {
  el.style.color = binding.value
})

// 使用 
// <div v-color="color"></div>
```

## 组件内容
### 组件注册
1. 全局注册 app.component('MyComponent', MyComponent)
2. 局部注册
```vue
// 组合式
<script setup>
// 引入即注册
import ComponentA from './ComponentA.vue'
</script>

// 选项式
export default {
components: {
    ComponentA
},
}
```

### 组件props defineProps
特性: props只读不可修改.

```vue
// child.vue
<script setup>
// 1. 值使用
const props = defineProps(['foo'])
console.log(props.foo)

// 2. 响应式解构 3.5+版本有效, 以往版本解构会变常量失去响应式
const { foo } = defineProps(['foo'])
// 错误用法 watch(foo, /* ... */)
watch(() => foo, /* ... */)
watchEffect(() => {
   // 在 3.5 之前只运行一次
   // 在 3.5+ 中在 "foo" prop 变化时重新执行
   console.log(foo)
})


class Person {
   constructor(firstName, lastName) {
      this.firstName = firstName
      this.lastName = lastName
   }
}

// 3. 类型限制
// 声明 default 值，无论 prop 是未被传递还是显式指明的 undefined，都会改为 default 值
defineProps({
   // 基础类型检查
   // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
   propA: Number,
   // Boolean 类型的未传递 prop 将被转换为 false。设置 default 修改默认行为
   propBoolean: Boolean,
   // 多种可能的类型
   // 允许多种类型时，Boolean 的转换规则也将被应用
   propB: [String, Number,Boolean,Array,Object,Date,Function,Symbol,Error],
   // 必传，且为 String 类型
   propC: {
      type: String,
      required: true
   },
   // 必传但可为 null 的字符串
   propD: {
      type: [String, null],
      required: true
   },
   // Number 类型的默认值
   propE: {
      type: Number,
      default: 100
   },
   // 对象类型的默认值
   propF: {
      type: Object,
      // 对象或数组的默认值
      // 必须从一个工厂函数返回。
      // 该函数接收组件所接收到的原始 prop 作为参数。
      default(rawProps) {
         return { message: 'hello' }
      }
   },
   // 自定义类型校验函数
   // 在 3.4+ 中完整的 props 作为第二个参数传入
   propG: {
      validator(value, props) {
         // The value must match one of these strings
         return ['success', 'warning', 'danger'].includes(value)
      }
   },
   // 函数类型的默认值
   propH: {
      type: Function,
      // 不像对象或数组的默认，这不是一个
      // 工厂函数。这会是一个用来作为默认值的函数
      default() {
         return 'Default function'
      }
   },
   // 类检测, 会通过 instanceof Person 来校验 author prop 的值是否是 Person 类的一个实例
   author: Person
})

</script>

```

```vue
// parent.vue
<child :foo="123"></child>
```

### 组件事件 defineEmits

```vue
// child.vue
<!--选项式api-->
<!--<script>-->
<!--   export default {-->
<!--      emits: ['inFocus', 'submit'],-->
<!--      setup(props, ctx) {-->
<!--         ctx.emit('submit')-->
<!--      }-->
<!--   }-->
<!--</script>-->

<script setup>
import { ref, defineEmits } from 'vue'
// 1. 数组使用
const emit = defineEmits(['inFocus', 'submit'])
function buttonClick() {
   emit('submit')
}

// 2. 对象使用 & 事件校验
const emit1 = defineEmits({
   // 没有校验
   click: null,

   // 校验 submit 事件
   submit: ({ email, password }) => {
      if (email && password) {
         return true // 触发事件
      } else {
         console.warn('Invalid submit event payload!')
         return false // 阻止触发事件
      }
   }
})
function submitForm(email, password) {
   emit1('submit', { email, password })
}
</script>

<template>
   <div>
      <!--   模板中通过$emit 触发事件   -->
      <button @click="$emit('someEvent', '参数')">Click Me</button>
   </div>
</template>

```

```vue
// parent.vue
<template>
   <child @someEvent="hanldeFun" v-on:submit="()=>{}"></child>
</template>
```

### 组件属性透传 $attrs & defineOptions & inheritAttrs
透传的 attribute 会自动被添加到组件根元素上.
+ 透传 class 和 style 的合并到根元素
+ v-on 监听器继承,事件会被添加到根元素
+ 根元素是组件也依然继续向下透传给子组件. 如果props或事件被本组件声明过,就不会透传给子组件.
+ 多个根节点的组件不会自动透传属性,通过$attrs进行自定义设置.
```vue
<header>...</header>
<main v-bind="$attrs">...</main>
<footer>...</footer>
```

+ `<script setup>` 中使用 useAttrs() API 来访问一个组件的所有透传 attribute
```vue
<script setup>
   import { useAttrs } from 'vue'
   const attrs = useAttrs()
</script>
```
+ 禁止属性透传 inheritAttrs & defineOptions 3.3+
```vue
// 选项式
<script >
   export default {
      inheritAttrs: false
   }
</script>

// 组合式
<script setup>
defineOptions({
  inheritAttrs: false
})
// ...setup 逻辑
</script>

<template>
   // $attrs 对象包含了除组件所声明的 props 和 emits 之外的所有其他 attribute
   <span>Fallthrough attribute: {{ $attrs }}</span>
</template>
```

### 依赖注入 provide & inject & readonly & hasInjectionContext
后代组件跨层级通信


```vue
// 祖先组件
<script setup>
import { ref, provide, readonly } from 'vue'

let value = ref(0)
let value1 = ref(0)
provide(/* 注入名 */ 'keyName', /* 值 */ value)
provide(/* 注入名 */ 'key', /* 值 */ {data: readonly(value)})

</script>

```

```vue
// 后代组件
import { inject, hasInjectionContext } from 'vue'

// 正常使用
const message = inject('keyName')
const { data } = inject('keyName')

// 默认值设置
const value = inject('keyName', '默认值')

// 函数/类默认值写法 避免多余调用
const value = inject('keyName', () => new ExpensiveClass(), true)

// 在setup之外的地方调用inject产生错误但是不发出警告  hasInjectionContext() 则返回true
const isInjectError = hasInjectionContext()
```

```js
// 全局注入 使用app实例
import { createApp } from 'vue'
const app = createApp({})
app.provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
```




### 异步组件 defineAsyncComponent
在组件被使用时才会加载,节约开销.

使用() => import('....')利于打包工具的代码分割.

```vue
<script setup>
import { ref, defineAsyncComponent } from 'vue'
import LoadingComponent from 'LoadingComponent.vue'
import ErrorComponent from 'ErrorComponent.vue'

// 简写异步组件
const AsyncComp = defineAsyncComponent(() => import('./AsyncComp.vue'))

// 完整版
const AsyncComp = defineAsyncComponent({
   // 加载函数
   loader: () => import('./AsyncComp.vue'),

   // 加载异步组件时使用的组件
   loadingComponent: LoadingComponent,
   // 展示加载组件前的延迟时间，默认为 200ms
   delay: 200,

   // 加载失败后展示的组件
   errorComponent: ErrorComponent,
   // 如果提供了一个 timeout 时间限制，并超时了
   // 也会显示这里配置的报错组件，默认值是：Infinity
   timeout: 3000
})
   
</script>
```

## 组合式函数 & toValue
+ 外部js中封装vue实例函数/属性,要使用 useName 作为命名. 类似react Hooks的自定义钩子函数 

+ 组合式函数使用限制:
1. DOM操作等副作用,服务端渲染要在onMounted()中操作.需在onUnmounted() 时清理副作用避免内存泄露.
2. 组合式函数只能在 `<script setup>` 或 setup() 钩子中被调用. 才能将生命周期钩子/计算属性/监听器注册到组件,否则会内存泄漏.

+ 替代mixin,mixin缺点:
1. 不清晰的数据来源
2. 命名空间冲突
3. 隐式的跨 mixin 交流,多个 mixin 依赖相同属性会相互影响.

```js
// useFetch.js
import { ref, watchEffect, toValue } from 'vue'

export function useFetch(url) {
   const data = ref(null)
   const error = ref(null)

   const fetchData = () => {
      data.value = null
      error.value = null
      
      // toValue 3.3+版本能使用, 类似一个map,如果参数是 ref，它会返回 ref 的值；参数是函数，会调用函数并返回其返回值。否则，它会原样返回参数。
      fetch(toValue(url))
              .then((res) => res.json())
              .then((json) => (data.value = json))
              .catch((err) => (error.value = err))
   }

   watchEffect(() => {
      // 会立即运行，并且会跟踪 toValue(url), 其值为响应式则监听,否则只运行一次. 
      fetchData()
   })

   return { data, error }
}
```


```js
const url = ref('/initial-url')
// 传递响应式参数
const { data, error } = useFetch(url)
const { data, error } = useFetch(() => `/posts/${props.id}`)

// 这将会重新触发 fetch
url.value = '/new-url'
```

## 内置属性

### 属性ref & useTemplateRef & defineExpose

+ 获取DOM/组件实例
+ 组件挂载后才能访问引用
+ useTemplateRef 3.5+版本才可使用
+ `this.$refs` 也是非响应式的，因此你不应该尝试在模板中使用它来进行数据绑定。

```vue
<script setup>
import { useTemplateRef, onMounted, ref } from 'vue'

// 1. 普通使用
// 第一个参数必须与模板中的 ref 值匹配
const input = useTemplateRef('my-input')
const input1 = ref(null)
onMounted(() => {
  input.value.focus()
   setTimeout(()=>{
      input1.value.focus()
   }, 500)
})

// 2. v-for中使用
const list = ref([
   /* ... */
])
const itemRefs = useTemplateRef('items')
const itemRefs1 = ref([])
   
// 3. 回调获取DOM
function getRef (el) {
    // el 为获取的对应元素的DOM
}

// 4. 组件实例获取
// 获取选项式组件ref可获得完全控制权
// 获取<script setup> 的组件,只能获取到其组件通过 defineExpose 所暴露的内容
const childRef = useTemplateRef('child')
const childRef1 = ref(null)
// 得到的实例类型为 { a: number, b: number } (ref 都会自动解包，和一般的实例一样)
   
</script>

<template>
   <div>
      <input ref="my-input" />
      <input :ref="input1" />
      
      <ul>
         <li v-for="item in list" ref="items">
            {{ item }}
         </li>
      </ul>
      <ul>
         <li v-for="item in list" :ref="itemRefs1">
            {{ item }}
         </li>
      </ul>

      <input :ref="getRef">

      <Child ref="child" />
      <Child :ref="childRef1" />
   </div>
</template>
```

```vue
// Child.vue
// ref属性得到的实例类型为 { a: number, b: number } (ref 都会自动解包，和一般的实例一样)
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

### 属性key

`key` 这个特殊的 attribute 主要作为 Vue 的虚拟 DOM 算法提示，在比较新旧节点列表时用于识别 vnode。

- **预期**：`number | string | symbol`

- **详细信息**

  在没有 key 的情况下，Vue 将使用一种最小化元素移动的算法，并尽可能地就地更新/复用相同类型的元素。如果传了 key，则将根据 key 的变化顺序来重新排列元素，并且将始终移除/销毁 key 已经不存在的元素。

  同一个父元素下的子元素必须具有**唯一的 key**。重复的 key 将会导致渲染异常。

  最常见的用例是与 `v-for` 结合：

  ```vue-html
  <ul>
    <li v-for="item in items" :key="item.id">...</li>
  </ul>
  ```

  也可以用于强制替换一个元素/组件而不是复用它。当你想这么做时它可能会很有用：

  - 在适当的时候触发组件的生命周期钩子
  - 触发过渡

  举例来说：

  ```vue-html
  <transition>
    <span :key="text">{{ text }}</span>
  </transition>
  ```

  当 `text` 变化时，`<span>` 总是会被替换而不是更新，因此 transition 将会被触发。


### 属性is

用于绑定动态组件。

- **预期**：`string | Component`

- **用于原生元素**

  - 仅在 3.1+ 中支持

  当 `is` attribute 用于原生 HTML 元素时，它将被当作 自定义元素，其为原生 web 平台的特性。

  Vue 用其组件来替换原生元素，在 `is` attribute 的值中加上 `vue:` 前缀，就会把该元素渲染为 Vue 组件：

  ```vue-html
  <table>
    <tr is="vue:my-row-component"></tr>
  </table>
  
  <component :is="Math.random() > 0.5 ? FooCmp : BarCmp" />
  ```

## 内置组件
可在模板中直接使用的组件

### Transition

+ 触发机制:
1. 由 v-if 所触发的切换
2. 由 v-show 所触发的切换
3. 由特殊元素 <component> 切换的动态组件
4. 改变特殊的 key 属性

+ 作用机制:
1. `<Transition>` 仅支持单个元素或组件作为其插槽内容
2. 在以下状态进行样式类的添加替换: (<Transition name="fade"></Transition> 设置了name时, v会替换成name,如:fade-enter-from)
![](https://cn.vuejs.org/assets/transition-classes.DYG5-69l.png)

v-enter-from：进入动画的起始状态。在元素插入之前添加，在元素插入完成后的下一帧移除。

v-enter-active：进入动画的生效状态。应用于整个进入动画阶段。在元素被插入之前添加，在过渡或动画完成之后移除。这个 class 可以被用来定义进入动画的持续时间、延迟与速度曲线类型。

v-enter-to：进入动画的结束状态。在元素插入完成后的下一帧被添加 (也就是 v-enter-from 被移除的同时)，在过渡或动画完成之后移除。

v-leave-from：离开动画的起始状态。在离开过渡效果被触发时立即添加，在一帧后被移除。

v-leave-active：离开动画的生效状态。应用于整个离开动画阶段。在离开过渡效果被触发时立即添加，在过渡或动画完成之后移除。这个 class 可以被用来定义离开动画的持续时间、延迟与速度曲线类型。

v-leave-to：离开动画的结束状态。在一个离开动画被触发后的下一帧被添加 (也就是 v-leave-from 被移除的同时)，在过渡或动画完成之后移除

```vue
<script setup>
import { ref } from 'vue'
let show = ref(false)

// 在元素被插入到 DOM 之前被调用
// 用这个来设置元素的 "enter-from" 状态
function onBeforeEnter(el) {}

// 在元素被插入到 DOM 之后的下一帧被调用
// 用这个来开始进入动画
function onEnter(el, done) {
   // 调用回调函数 done 表示过渡结束
   // 如果与 CSS 结合使用，则这个回调是可选参数
   done()
}

// 当进入过渡完成时调用。
function onAfterEnter(el) {}

// 当进入过渡在完成之前被取消时调用
function onEnterCancelled(el) {}

// 在 leave 钩子之前调用
// 大多数时候，你应该只会用到 leave 钩子
function onBeforeLeave(el) {}

// 在离开过渡开始时调用
// 用这个来开始离开动画
function onLeave(el, done) {
   // 调用回调函数 done 表示过渡结束
   // 如果与 CSS 结合使用，则这个回调是可选参数
   done()
}

// 在离开过渡完成、
// 且元素已从 DOM 中移除时调用
function onAfterLeave(el) {}

// 仅在 v-show 过渡中可用
function onLeaveCancelled(el) {}
</script>

<template>
   <button @click="show = !show">Toggle</button>
   // 1. 默认使用
   <Transition>
      <p v-if="show">hello</p>
   </Transition>
   
   // 2. 具名使用
   <Transition name="fade">
      ...
   </Transition>
   <Transition :name="transitionName">
      <!-- ... -->
   </Transition>

   // 3. 自定类名使用
   <Transition
           name="custom-classes"
           enter-from-class="xxx"
           enter-active-class="animate__animated animate__tada"
           enter-to-class="xxx"
           leave-from-class="xxx"
           leave-active-class="animate__animated animate__bounceOutRight"
           leave-to-class="xxx"
   >
      <p v-if="show">hello</p>
   </Transition>
   
   // 4. 钩子
   <Transition
           @before-enter="onBeforeEnter"
           @enter="onEnter"
           @after-enter="onAfterEnter"
           @enter-cancelled="onEnterCancelled"
           @before-leave="onBeforeLeave"
           @leave="onLeave"
           @after-leave="onAfterLeave"
           @leave-cancelled="onLeaveCancelled"
           :css="false"
   >
      <!--  :css="false" // 跳过css自动探测提升性能 -->
   </Transition>
   
   // 5. 首次渲染过渡
   <Transition appear>
      ...
   </Transition>
   
   // 6. 过渡模式 内置的一个过渡样式
   <Transition mode="out-in">
      ...
   </Transition>
   <Transition mode="in-out">
      ...
   </Transition>
</template>

<style scoped>
   .v-enter-active,
   .v-leave-active {
      transition: opacity 0.5s ease;
   }

   .v-enter-from,
   .v-leave-to {
      opacity: 0;
   }
</style>
```


### TransitionGroup
与Transition区别:
1. 默认情况下，它不会渲染一个容器元素.可以通过传入 tag prop 来指定一个元素作为容器元素来渲染.
2. CSS 过渡 class 会被应用在列表内的元素上，而不是容器元素上.
3. 列表中的每个元素都必须有一个独一无二的 key attribute.
4. 过渡模式(mode属性)在这里不可用，因为我们不再是在互斥的元素之间进行切换.

```vue

<script setup>
import { ref } from 'vue'

function onEnter(el, done) {
   gsap.to(el, {
      opacity: 1,
      height: '1.6em',
      delay: el.dataset.index * 0.15,
      onComplete: done
   })
}
</script>

<template>
   // 1. 使用
   <TransitionGroup name="list" tag="ul">
      <li v-for="item in items" :key="item">
         {{ item }}
      </li>
   </TransitionGroup>
   
   // 2. 钩子
   <TransitionGroup
           tag="ul"
           :css="false"
           @before-enter="onBeforeEnter"
           @enter="onEnter"
           @leave="onLeave"
   >
      <li
              v-for="(item, index) in computedList"
              :key="item.msg"
              :data-index="index"
      >
         {{ item.msg }}
      </li>
   </TransitionGroup>
</template>

<style scoped>
   .list-enter-active,
   .list-leave-active {
      transition: all 0.5s ease;
   }
   .list-enter-from,
   .list-leave-to {
      opacity: 0;
      transform: translateX(30px);
   }
</style>


```

### KeepAlive
多个组件间动态切换时缓存被移除的组件实例.

```vue
<script setup>
   import { onActivated, onDeactivated } from 'vue'

   // onActivated 在组件挂载时也会调用，并且 onDeactivated 在组件卸载时也会调用
   onActivated(() => {
      // 调用时机为首次挂载
      // 以及每次从缓存中被重新插入时
   })

   onDeactivated(() => {
      // 在从 DOM 上移除、进入缓存
      // 以及组件卸载时调用
   })
</script>

<template>
<!-- 1. 使用  -->
   <!-- 非活跃的组件将会被缓存！ -->
   <KeepAlive>
      <component :is="activeComponent" />
   </KeepAlive>

<!-- 2. 仅缓存include使用 匹配组件的name,使用 <script setup> 的单文件组件会自动根据文件名生成对应的 name 选项，无需再手动声明 -->
   <!-- 以英文逗号分隔的字符串 -->
   <KeepAlive include="a,b">
      <component :is="view" />
   </KeepAlive>
   <!-- 正则表达式 (需使用 `v-bind`) -->
   <KeepAlive :include="/a|b/">
      <component :is="view" />
   </KeepAlive>
   <!-- 数组 (需使用 `v-bind`) -->
   <KeepAlive :include="['a', 'b']">
      <component :is="view" />
   </KeepAlive>
   
<!--3. 最大缓存组件数量-->
   <KeepAlive :max="10">
      <component :is="activeComponent" />
   </KeepAlive>
</template>

<style scoped>

</style>


```

### Teleport
将元素或组件传递到任意DOM节点内

```vue
<template>
<!-- 1.  传送的 to 目标必须已经存在于 DOM 中,否则报错-->
   <Teleport to="body">
      <div v-if="open" class="modal">
         <p>Hello from the modal!</p>
         <button @click="open = false">Close</button>
      </div>
   </Teleport>

<!-- 2. 禁用 渲染在原组件位置  -->
   <Teleport :disabled="isMobile">
      ...
   </Teleport>
<!--3. 延迟加载 defer 3.5+版本可以使用-->
   <Teleport defer to="#late-div">...</Teleport>
</template>

```

### Suspense
触发依赖:
1. 带有异步 setup() 钩子的组件。这也包含了使用 `<script setup>` 时有顶层 await 表达式的组件。
2. 异步组件。

```vue
<script setup>
import { ref } from 'vue'
const res = await fetch('...')
</script>

<template>
<!--   1. 使用-->
<!--   pending 事件是在进入挂起状态时触发。-->
<!--   resolve 事件是在 default 插槽完成获取新内容时触发。-->
<!--   fallback 事件则是在 fallback 插槽的内容显示时触发-->
   <Suspense @pending @resolve @fallback>
      <!-- 具有深层异步依赖的组件 -->
      <Dashboard />

      <!-- 在 #fallback 插槽中显示 “正在加载中” -->
      <template #fallback>
         Loading...
      </template>
   </Suspense>

<!--   2. 嵌套使用 suspensible 让父子组件同步使用父组件的Suspense,不会产生空白过渡问题-->
   <Suspense>
      <component :is="DynamicAsyncOuter">
         <Suspense suspensible> <!-- 像这样 -->
            <component :is="DynamicAsyncInner" />
         </Suspense>
      </component>
   </Suspense>
</template>

```

+ 有序组合嵌套

```vue
<template>
   <RouterView v-slot="{ Component }">
      <template v-if="Component">
         <Transition mode="out-in">
            <KeepAlive>
               <Suspense>
                  <!-- 主要内容 -->
                  <component :is="Component"></component>

                  <!-- 加载中状态 -->
                  <template #fallback>
                     正在加载...
                  </template>
               </Suspense>
            </KeepAlive>
         </Transition>
      </template>
   </RouterView>
</template>

```

### component
渲染动态组件或元素

```vue
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Math.random() > 0.5 ? Foo : Bar" />
  <component :is="Math.random() > 0.5 ? 'a' : 'span'"></component>
</template>
```

### template
用于包裹模板的空标签,不会被渲染,不能使用ref/is等属性.
带有 v-for 的 <template> 也可以有一个 key 属性,所有其他的属性和指令都将被丢弃.

```vue
<template>
    <!--   可以使用以下指令-->
   <template v-if></template>
   <template v-for></template>
   <template v-slot></template>
</template>
```

## 编译打包选项

### `__VUE_OPTIONS_API__`

- **默认值：**`true`

  启用/禁用选项式 API 支持。禁用此功能将减小打包结果的体积，但如果第三方库依赖选项式 API，则可能影响兼容性。

### `__VUE_PROD_DEVTOOLS__`

- **默认值：**`false`

  在生产环境中启用/禁用开发者工具支持。启用会在打包结果中包含更多代码，因此建议仅在调试时启用此功能。

### `__VUE_PROD_HYDRATION_MISMATCH_DETAILS__`

- **默认值：**`false`

  启用/禁用生产环境构建下激活 (hydration) 不匹配的详细警告。启用会在打包结果中包含更多代码，因此建议仅在调试时启用此功能。

- 仅在 3.4+ 中可用

### Vite 配置指南

```js
// vite.config.js
import { defineConfig } from 'vite'

export default defineConfig({
  define: {
    // 启用生产环境构建下激活不匹配的详细警告
    __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'true'
  }
})
```

### vue-cli 配置指南

```js
// vue.config.js
module.exports = {
  chainWebpack: (config) => {
    config.plugin('define').tap((definitions) => {
      Object.assign(definitions[0], {
        __VUE_OPTIONS_API__: 'true',
        __VUE_PROD_DEVTOOLS__: 'false',
        __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'false'
      })
      return definitions
    })
  }
}
```

### webpack 配置指南

```js
// webpack.config.js
module.exports = {
  // ...
  plugins: [
    new webpack.DefinePlugin({
      __VUE_OPTIONS_API__: 'true',
      __VUE_PROD_DEVTOOLS__: 'false',
      __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'false'
    })
  ]
}
```

## 使用安全
核心: 不使用用户输入内容作为模板.

1. v-html/innerHTML/渲染函数 注入任何元素/链接,做任何操作
2. 元素 url 注入劫持,乱跳转
3. 元素 style 注入劫持,乱跳转
4. 渲染script标签注入, 做任何事

## 组合式 对比 选项式 优势

1. 粒度小,更好的逻辑复用
2. 更灵活
3. 更好的ts支持
4. 更小打包体积

## 响应式 & 渲染机制

响应式机制:
+ Vue 3 中则使用了 Proxy 来创建响应式对象，仅将 getter / setter 用于 ref.
+ 在访问时跟踪依赖、在变更时触发副作用的值容器.
+ 运行时响应式,追踪和触发都是在浏览器中运行时进行的,较少边界情况.

渲染机制: 
编译 (模板编译为渲染函数) => 挂载 (调用渲染函数,返回虚拟DOM树,并创建实际DOM节点) => 更新 (运行副作用,新旧DOM树对比,更新真实DOM)
![](https://cn.vuejs.org/assets/render-pipeline.CwxnH_lZ.png)

模板与渲染函数比较: 模板静态分析性能好, 渲染函数灵活.

VUE编译时创建/对比虚拟DOM:
+ 对比react等运行时虚拟DOM框架,性能更好. 因为vue可以通过模板做静态分析及更多优化,而其他框架永远需要接受一个未知的内容.
+ 静态提升 对于静态元素除了首次渲染外,更新对比时自动忽略及缓存使用
```html
<div>
  <div>foo</div> <!-- 静态提升 -->
  <div>bar</div> <!-- 静态提升 -->
  <div>{{ dynamic }}</div>
</div>
```

+ 更新类型标记 根据每种使用vue语法的元素,自动推动其更新的类型,减少对比提升性能
```html
<!-- 仅含 class 绑定 -->
<div :class="{ active }"></div>

<!-- 仅含 id 和 value 绑定 -->
<input :id="id" :value="value">

<!-- 仅含文本子节点 -->
<div>{{ dynamic }}</div>
```

```js
createElementVNode("div", {
  class: _normalizeClass({ active: _ctx.active })
}, null, 2 /* 仅更新CLASS的类型 */)
```

+ 树结构打平 内部编译模板时,会将静态元素剥离,形成只存在有vue操作的元素的树结构,减少对比提升性能
```html
<div> <!-- root block -->
  <div>...</div>         <!-- 不会追踪 -->
  <div :id="id"></div>   <!-- 要追踪 -->
  <div>                  <!-- 不会追踪 -->
    <div>{{ bar }}</div> <!-- 要追踪 -->
  </div>
</div>
```

```text
// 上面模板内容转化为以下树结构
div (block root)
- div 带有 :id 绑定
- div 带有 {{ bar }} 绑定
```

## 渲染函数 h()
[更灵活,但各种指令及修饰符需要自己写逻辑](https://cn.vuejs.org/guide/extras/render-function.html)

```vue
<script>
   import { ref, h } from 'vue'

   export default {
      props: {
         /* ... */
      },
      setup(props) {
         const count = ref(1)
         // 返回渲染函数
         return () => h('div', props.msg + count.value)
      }
   }
</script>
```
