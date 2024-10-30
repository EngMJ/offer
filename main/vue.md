# Vue2 API & 面试题

* * *

## [1. Vue 2.x API参考](vue2.md)

***

## 2. Vue实例挂载的过程中发生了什么?

### vue2 挂载过程
1. 调用 `new Vue(options)` 创建一个 Vue 实例，并传入选项对象（例如 `data`、`methods`、`computed` 等）,在内部初始化一些实例属性($set\$mount等)。

2. 调用 `beforeCreate` 生命周期钩子

3. 初始化 `data`、`props`、`computed`、`methods` 等选项,通过 `Object.defineProperty` 为每个 `data` 属性设置 getter(依赖收集) 和 setter，以实现响应式系统.

4. 调用 `created` 生命周期钩子

5. 检查是否有 `el` 或 `$mount` 挂载点,将实例挂载指定的DOM元素上.

6. 模板编译,将 `template` 内容转换为渲染函数（`render` 函数）,没有 `template`会将 `el` 元素的 HTML 作为模板转化为渲染函数,最终渲染到DOM上。

7. 调用 `beforeMount` 生命周期钩子

8. 创建 Watcher 来追踪渲染函数中数据的变化, 触发 Watcher 会执行渲染函数并生成虚拟 DOM 树,并首次渲染页面。

9. 调用 `mounted` 生命周期钩子

10. 当 `data` 中的属性变化时，会触发响应式系统,触发 Watcher 重新执行渲染函数，生成新的虚拟 DOM 并将其与旧的虚拟 DOM 进行对比（diff 算法），将变化的部分渲染到页面。

### vue3 挂载过程

1. 使用 `createApp` 方法创建应用实例，返回一个包含根组件和全局配置的应用对象。Vue 3 中应用实例是独立的，允许创建多个 Vue 应用，每个应用有独立的配置。

2. 调用应用实例的 `.mount()` 方法，将根组件渲染到 DOM 上，并触发挂载过程。

3. 挂载时创建根组件实例（`App` 组件）,调用 `setup` 函数。

4. 调用 `beforeCreate` 和 `created` 生命周期钩子

5. 执行 `setup` 函数,使用Proxy创建响应式数据和副作用,并收集依赖.

6. 将 `template` 编译为 `render` 函数，创建虚拟DOM,渲染器将虚拟 DOM 映射为真实 DOM 并更新页面。

7. 调用 `beforeMount` 生命周期钩子

8. 创建一个渲染 `effect`（类似于 Vue 2 中的渲染 Watcher），追踪渲染过程中的响应式数据依赖变化(proxy数据写操作/活跃的 Effect 函数),会生产新的虚拟DOM。

9. 调用 `mounted` 生命周期钩子

10. 数据变化后会触发响应式更新，仅更新变化部分的虚拟 DOM。

***

## 2. 简述 Vue 的生命周期以及每个阶段做的事

分为8个阶段：**创建前后, 载入前后, 更新前后, 销毁前后**，特殊场景的生命周期。

* * *

| 生命周期v2        | 生命周期v3            | 描述                              | 
|---------------|-------------------|---------------------------------|
| beforeCreate  | beforeCreate      | 组件实例被创建之初                       |
| created       | created           | 组件实例已经完全创建,各种数据可以使用，常用于异步数据获取   |
| beforeMount   | beforeMount       | 组件挂载之前                          |
| mounted       | mounted           | 组件挂载到实例后,dom已创建，可用于获取访问数据和dom元素 |
| beforeUpdate  | beforeUpdate      | 组件数据发生变化更新之前,可用于获取更新前各种状态       |
| updated       | updated           | 数据数据更新之后                        |
| beforeDestroy | **beforeUnmount** | 组件实例销毁之前,可用于一些定时器或订阅的取消         |
| destroyed     | **unmounted**     | 组件实例销毁之后,可用于一些定时器或订阅的取消         |

* * *

| 生命周期v2        | 生命周期v3              | 描述                       |
|---------------|---------------------|--------------------------|
| activated     | activated           | keep-alive 缓存的组件激活时      |
| deactivated   | deactivated         | keep-alive 缓存的组件停用时调用    |
| errorCaptured | errorCaptured       | 捕获一个来自子孙组件的错误时被调用        |
| \-            | **renderTracked**   | 调试钩子，响应式依赖被收集时调用         |
| \-            | **renderTriggered** | 调试钩子，响应式依赖被触发时调用         |
| \-            | **serverPrefetch**  | ssr only，组件实例在服务器上被渲染前调用 |


![vue3 生命周期图](https://cn.vuejs.org/assets/lifecycle_zh-CN.W0MNXI0C.png)

* * *

+ 生命周期中的数据请求

    vue2中适合在created中请求,更早调用. 在mounted中请求可能会造成页面闪动.

    vue3中在setup中直接请求,因为这个调用在所有生命周期之前.

+ setup中为什么没有beforeCreate和created？

   setup 函数最先执行,本身已经承担了初始化阶段的职责，因此 beforeCreate 和 created 钩子不再单独存在。

* * *

## 3. 说一下 Vue 子组件和父组件创建和挂载顺序

创建先父后子，挂载先子后父

### Vue 2 中，父子组件的生命周期顺序：
    父组件的 beforeCreate
    父组件的 created
    父组件的 beforeMount
    子组件的 beforeCreate
    子组件的 created
    子组件的 beforeMount
    子组件的 mounted
    父组件的 mounted

###  Vue 3 中，父子组件的生命周期顺序：

    父组件的 beforeCreate
    父组件的 created
    子组件的 beforeCreate
    子组件的 created
    子组件的 beforeMount
    子组件的 mounted
    父组件的 beforeMount
    父组件的 mounted


* * *

## 4. 说说从 template 到 render 处理过程

+ 作用: Vue的编译器模块“compiler”，将template编译为render函数。编译器流程是先对template进行parse，获得抽象语法树AST，然后进行transform，转换出render函数.
+ 用途: 让开发者可以使用模板编码降低开发成本,渲染函数编码成本高。

* * *

## 5. Vue组件为什么只能有一个根元素?

+   `vue2`组件只能有一个根元素，但`vue3`可以多根元素
+   组件模板会转化为`vdom`,而虚拟DOM是一颗单根树形结构，且`patch`方法在遍历的时候从根节点开始遍历。
+   `vue3`可写多个根节点，编译时自动使用`Fragment`虚拟节点进行包裹，把多个根节点作为它的children,进行patch时直接遍历children创建或更新。

* * *

## 6. Vue组件之间通信方式有哪些

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf775050e1f948bfa52f3c79b3a3e538~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

###  组件通信常用方式： (vue3去除了删除线的api)

+   props
+   $emit / ~~$on~~
+   $parent / ~~$children~~
+   $attrs / ~~$listeners~~
+   $root
+   ref
+   Provide / Inject
+   vuex
+   eventbus


### 组件关系通信

+   父子组件

    +   `props`/`$emit`/ ~~$on~~ /`$parent`/ ~~$children~~ / `ref` / `$attrs` / ~~$listeners~~
+   兄弟组件

    +   `$parent` / `$root`
+   任意关系

    +   `eventbus` / `vuex` / `provide`+`inject`

* * *

## 7.子组件可以直接改变父组件的数据么，说明原因

+ 原因: vue组件化开发遵循**单项数据流原则**，如果能互相修改将状态就难以控制与朔源,修改props也会报错.
+ 正规修改方法: 事件传递/~~$parent~~

* * *

## 8.Vue中如何扩展一个组件

1.  按照逻辑扩展和内容扩展来列举，

    +   逻辑扩展有：mixins、extends、composition api；

    +   内容扩展有slots；

2.  分别说出他们使用方法、场景差异和问题。

3.  作为扩展，还可以说说vue3中新引入的composition api带来的变化


* * *

### 回答范例：

1.  常见的组件扩展方法有：mixins，slots，extends等

2.  混入mixins是分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。

    ```js
    // 复用代码：它是一个配置对象，选项和组件里面一样
    const mymixin = {
       methods: {
          dosomething(){}
       }
    }
    // 全局混入：将混入对象传入
    Vue.mixin(mymixin)
    
    // 局部混入：做数组项设置到mixins选项，仅作用于当前组件
    const Comp = {
       mixins: [mymixin]
    }
    ```


* * *

3.  插槽主要用于vue组件中的内容分发，也可以用于组件扩展。

    子组件Child

    ```html
    <div>
      <slot>这个内容会被父组件传递的内容替换</slot>
    </div>
    ```

    父组件Parent

    ```html
    <div>
       <Child>来自老爹的内容</Child>
    </div>
    ```

    如果要精确分发到不同位置可以使用具名插槽，如果要使用子组件中的数据可以使用作用域插槽。


* * *

4.  组件选项中还有一个不太常用的选项extends，也可以起到扩展组件的目的

    ```js
    // 扩展对象
    const myextends = {
       methods: {
          dosomething(){}
       }
    }
    // 组件扩展：做数组项设置到extends选项，仅作用于当前组件
    // 跟混入的不同是它只能扩展单个对象
    // 另外如果和混入发生冲突，该选项优先级较高，优先起作用
    const Comp = {
       extends: myextends
    }
    ```


* * *

5.  混入的数据和方法**不能明确判断来源**且可能和当前组件内变量**产生命名冲突**，vue3中引入的composition api，可以很好解决这些问题，利用独立出来的响应式模块可以很方便的编写独立逻辑并提供响应式的数据，然后在setup选项中组合使用，增强代码的可读性和维护性。例如：

    ```js
    // 复用逻辑1
    function useXX() {}
    // 复用逻辑2
    function useYY() {}
    // 逻辑组合
    const Comp = {
       setup() {
          const {xx} = useXX()
          const {yy} = useYY()
          return {xx, yy}
       }
    }
    ```


* * *

### 可能的追问

Vue.extend方法你用过吗？它能用来做组件扩展吗？

* * *

### 知其所以然

mixins原理：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiCreateApp.ts#L232-L233 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiCreateApp.ts#L232-L233")

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/componentOptions.ts#L545 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/componentOptions.ts#L545")

slots原理：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/componentSlots.ts#L129-L130 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/componentSlots.ts#L129-L130")

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1373-L1374 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1373-L1374")

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/helpers/renderSlot.ts#L23-L24 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/helpers/renderSlot.ts#L23-L24")

* * *

## 15-怎么缓存当前的组件？缓存后怎么更新？

缓存组件使用keep-alive组件，这是一个非常常见且有用的优化手段，vue3中keep-alive有比较大的更新，能说的点比较多。

### 思路

1.  缓存用keep-alive，它的作用与用法
2.  使用细节，例如缓存指定/排除、结合router和transition
3.  组件缓存后更新可以利用activated或者beforeRouteEnter
4.  原理阐述

* * *

### 回答范例

1.  开发中缓存组件使用keep-alive组件，keep-alive是vue内置组件，keep-alive包裹动态组件component时，会缓存不活动的组件实例，而不是销毁它们，这样在组件切换过程中将状态保留在内存中，防止重复渲染DOM。

    ```vue
    <keep-alive>
      <component :is="view"></component>
    </keep-alive>
    ```

2.  结合属性include和exclude可以明确指定缓存哪些组件或排除缓存指定组件。vue3中结合vue-router时变化较大，之前是`keep-alive`包裹`router-view`，现在需要反过来用`router-view`包裹`keep-alive`：

    ```vue
    <router-view v-slot="{ Component }">
      <keep-alive>
        <component :is="Component"></component>
      </keep-alive>
    </router-view>
    ```


* * *

3.  缓存后如果要获取数据，解决方案可以有以下两种：

    +   beforeRouteEnter：在有vue-router的项目，每次进入路由的时候，都会执行`beforeRouteEnter`

        ```js
        beforeRouteEnter(to, from, next){
          next(vm=>{
            console.log(vm)
            // 每次进入路由执行
            vm.getData()  // 获取数据
          })
        },
        ```

    +   actived：在`keep-alive`缓存的组件被激活的时候，都会执行`actived`钩子

        ```js
        activated(){
        	  this.getData() // 获取数据
        },
        ```


* * *

4.  keep-alive是一个通用组件，它内部定义了一个map，缓存创建过的组件实例，它返回的渲染函数内部会查找内嵌的component组件对应组件的vnode，如果该组件在map中存在就直接返回它。由于component的is属性是个响应式数据，因此只要它变化，keep-alive的render函数就会重新执行。


* * *

### 知其所以然

KeepAlive定义

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L73-L74 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L73-L74")

缓存定义

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L102-L103 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L102-L103")

缓存组件

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L215-L216 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L215-L216")

获取缓存组件

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L241-L242 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/components/KeepAlive.ts#L241-L242")

测试缓存特性，test-v3.html

* * *

## 29-什么是递归组件？举个例子说明下？

### 分析

递归组件我们用的比较少，但是在Tree、Menu这类组件中会被用到。

* * *

### 体验

组件通过组件名称引用它自己，这种情况就是递归组件。

> An SFC can implicitly refer to itself via its filename.

```xml
<template>
  <li>
    <div> {{ model.name }}</div>
    <ul v-show="isOpen" v-if="isFolder">
      <!-- 注意这里：组件递归渲染了它自己 -->
      <TreeItem
        class="item"
        v-for="model in model.children"
        :model="model">
      </TreeItem>
    </ul>
  </li>
<script>
export default {
  name: 'TreeItem',
  // ...
}
</script>
```

* * *

### 思路

+   下定义
+   使用场景
+   使用细节
+   原理阐述

* * *

### 回答范例

0.  如果某个组件通过组件名称引用它自己，这种情况就是递归组件。
1.  实际开发中类似Tree、Menu这类组件，它们的节点往往包含子节点，子节点结构和父节点往往是相同的。这类组件的数据往往也是树形结构，这种都是使用递归组件的典型场景。
2.  使用递归组件时，由于我们并未也不能在组件内部导入它自己，所以设置组件`name`属性，用来查找组件定义，如果使用SFC，则可以通过SFC文件名推断。组件内部通常也要有递归结束条件，比如model.children这样的判断。
3.  查看生成渲染函数可知，递归组件查找时会传递一个布尔值给`resolveComponent`，这样实际获取的组件就是当前组件本身。

* * *

### 知其所以然

递归组件编译结果中，获取组件时会传递一个标识符 `_resolveComponent("Comp", true)`

```ini
const _component_Comp = _resolveComponent("Comp", true)
```

就是在传递`maybeSelfReference`

```typescript
export function resolveComponent(
  name: string,
  maybeSelfReference?: boolean
): ConcreteComponent | string {
  return resolveAsset(COMPONENTS, name, true, maybeSelfReference) || name
}
```

resolveAsset中最终返回的是组件自身：

```kotlin
if (!res && maybeSelfReference) {
    // fallback to implicit self-reference
    return Component
}
```

* * *

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/helpers/resolveAssets.ts#L22-L23 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/helpers/resolveAssets.ts#L22-L23")

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/helpers/resolveAssets.ts#L110-L111 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/helpers/resolveAssets.ts#L110-L111")

[sfc.vuejs.org/#eyJBcHAudn…](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBjb21wIGZyb20gJy4vQ29tcC52dWUnXG5jb25zdCBtc2cgPSByZWYoJ+mAkuW9kue7hOS7ticpXG5jb25zdCBtb2RlbCA9IHtcbiAgbGFiZWw6ICdub2RlLTEnLFxuICBjaGlsZHJlbjogW1xuICAgIHtsYWJlbDogJ25vZGUtMS0xJ30sXG4gICAge2xhYmVsOiAnbm9kZS0xLTInfVxuICBdXG59XG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aDE+e3sgbXNnIH19PC9oMT5cbiAgPGNvbXAgOm1vZGVsPVwibW9kZWxcIj48L2NvbXA+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJDb21wLnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGRpdj5cbiAgICB7e21vZGVsLmxhYmVsfX1cbiAgPC9kaXY+XG4gIDxDb21wIHYtZm9yPVwiaXRlbSBpbiBtb2RlbC5jaGlsZHJlblwiIDptb2RlbD1cIml0ZW1cIj48L0NvbXA+XG4gIDxjb21wMj48L2NvbXAyPlxuPC90ZW1wbGF0ZT5cbjxzY3JpcHQ+XG5cdGV4cG9ydCBkZWZhdWx0IHtcbiAgICBuYW1lOiAnQ29tcCcsXG4gICAgcHJvcHM6IHtcbiAgICAgIG1vZGVsOiBPYmplY3RcbiAgICB9LFxuICAgIGNvbXBvbmVudHM6IHtcbiAgICAgIGNvbXAyOiB7XG4gICAgICAgIHJlbmRlcigpe31cbiAgICAgIH1cbiAgICB9XG4gIH1cbjwvc2NyaXB0PiJ9 "https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBjb21wIGZyb20gJy4vQ29tcC52dWUnXG5jb25zdCBtc2cgPSByZWYoJ+mAkuW9kue7hOS7ticpXG5jb25zdCBtb2RlbCA9IHtcbiAgbGFiZWw6ICdub2RlLTEnLFxuICBjaGlsZHJlbjogW1xuICAgIHtsYWJlbDogJ25vZGUtMS0xJ30sXG4gICAge2xhYmVsOiAnbm9kZS0xLTInfVxuICBdXG59XG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aDE+e3sgbXNnIH19PC9oMT5cbiAgPGNvbXAgOm1vZGVsPVwibW9kZWxcIj48L2NvbXA+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJDb21wLnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGRpdj5cbiAgICB7e21vZGVsLmxhYmVsfX1cbiAgPC9kaXY+XG4gIDxDb21wIHYtZm9yPVwiaXRlbSBpbiBtb2RlbC5jaGlsZHJlblwiIDptb2RlbD1cIml0ZW1cIj48L0NvbXA+XG4gIDxjb21wMj48L2NvbXAyPlxuPC90ZW1wbGF0ZT5cbjxzY3JpcHQ+XG5cdGV4cG9ydCBkZWZhdWx0IHtcbiAgICBuYW1lOiAnQ29tcCcsXG4gICAgcHJvcHM6IHtcbiAgICAgIG1vZGVsOiBPYmplY3RcbiAgICB9LFxuICAgIGNvbXBvbmVudHM6IHtcbiAgICAgIGNvbXAyOiB7XG4gICAgICAgIHJlbmRlcigpe31cbiAgICAgIH1cbiAgICB9XG4gIH1cbjwvc2NyaXB0PiJ9")

* * *

## 30-异步组件是什么？使用场景有哪些？

### 分析

因为异步路由的存在，我们使用异步组件的次数比较少，因此还是有必要两者的不同。

### 体验

大型应用中，我们需要分割应用为更小的块，并且在需要组件时再加载它们。

> In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it's needed.

```javascript
import { defineAsyncComponent } from 'vue'
// defineAsyncComponent定义异步组件
const AsyncComp = defineAsyncComponent(() => {
  // 加载函数返回Promise
  return new Promise((resolve, reject) => {
    // ...可以从服务器加载组件
    resolve(/* loaded component */)
  })
})
// 借助打包工具实现ES模块动态导入
const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)
```

* * *

### 思路

0.  异步组件作用
1.  何时使用异步组件
2.  使用细节
3.  和路由懒加载的不同

* * *

### 范例

0.  在大型应用中，我们需要分割应用为更小的块，并且在需要组件时再加载它们。
1.  我们不仅可以在路由切换时懒加载组件，还可以在页面组件中继续使用异步组件，从而实现更细的分割粒度。
2.  使用异步组件最简单的方式是直接给defineAsyncComponent指定一个loader函数，结合ES模块动态导入函数import可以快速实现。我们甚至可以指定loadingComponent和errorComponent选项从而给用户一个很好的加载反馈。另外Vue3中还可以结合Suspense组件使用异步组件。
3.  异步组件容易和路由懒加载混淆，实际上不是一个东西。异步组件不能被用于定义懒加载路由上，处理它的是vue框架，处理路由组件加载的是vue-router。但是可以在懒加载的路由组件中使用异步组件。

* * *

### 知其所以然

defineAsyncComponent定义了一个高阶组件，返回一个包装组件。包装组件根据加载器的状态决定渲染什么内容。

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiAsyncComponent.ts#L43-L44 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiAsyncComponent.ts#L43-L44")

* * *

## 组件与插件的区别


## 09 - 说说你对虚拟 DOM 的理解？

### 分析

现有框架几乎都引入了虚拟 DOM 来对真实 DOM 进行抽象，也就是现在大家所熟知的 VNode 和 VDOM，那么为什么需要引入虚拟 DOM 呢？围绕这个疑问来解答即可！

### 思路

1.  vdom是什么
2.  引入vdom的好处
3.  vdom如何生成，又如何成为dom
4.  在后续的diff中的作用

* * *

### 回答范例

1.  虚拟dom顾名思义就是虚拟的dom对象，它本身就是一个 `JavaScript` 对象，只不过它是通过不同的属性去描述一个视图结构。

2.  通过引入vdom我们可以获得如下好处：

    **将真实元素节点抽象成 VNode，有效减少直接操作 dom 次数，从而提高程序性能**

    +   直接操作 dom 是有限制的，比如：diff、clone 等操作，一个真实元素上有许多的内容，如果直接对其进行 diff 操作，会去额外 diff 一些没有必要的内容；同样的，如果需要进行 clone 那么需要将其全部内容进行复制，这也是没必要的。但是，如果将这些操作转移到 JavaScript 对象上，那么就会变得简单了。
    +   操作 dom 是比较昂贵的操作，频繁的dom操作容易引起页面的重绘和回流，但是通过抽象 VNode 进行中间处理，可以有效减少直接操作dom的次数，从而减少页面重绘和回流。

    **方便实现跨平台**

    +   同一 VNode 节点可以渲染成不同平台上的对应的内容，比如：渲染在浏览器是 dom 元素节点，渲染在 Native( iOS、Android) 变为对应的控件、可以实现 SSR 、渲染到 WebGL 中等等
    +   Vue3 中允许开发者基于 VNode 实现自定义渲染器（renderer），以便于针对不同平台进行渲染。

* * *

3.  vdom如何生成？在vue中我们常常会为组件编写模板 - template， 这个模板会被编译器 - compiler编译为渲染函数，在接下来的挂载（mount）过程中会调用render函数，返回的对象就是虚拟dom。但它们还不是真正的dom，所以会在后续的patch过程中进一步转化为dom。

    ![image-20220209153820845](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80b653050433436da876459a26ab5a65~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

4.  挂载过程结束后，vue程序进入更新流程。如果某些响应式数据发生变化，将会引起组件重新render，此时就会生成新的vdom，和上一次的渲染结果diff就能得到变化的地方，从而转换为最小量的dom操作，高效更新视图。


* * *

### 知其所以然

vnode定义：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L127-L128 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L127-L128")

观察渲染函数：21-vdom/test-render-v3.html

创建vnode：

+   createElementBlock:

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L291-L292 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L291-L292")

+   createVnode:

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L486-L487 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L486-L487")

+   首次调用时刻：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiCreateApp.ts#L283-L284 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiCreateApp.ts#L283-L284")

* * *

mount:

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1171-L1172 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1171-L1172")

调试mount过程：mountComponent

21-vdom/test-render-v3.html

* * *

## 10 - 你了解diff算法吗？

### 分析

必问题目，涉及vue更新原理，比较考查理解深度。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bffb8ffca9f0468c8a31576cebe6e692~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

### 思路

1.  diff算法是干什么的
2.  它的必要性
3.  它何时执行
4.  具体执行方式
5.  拔高：说一下vue3中的优化

* * *

### 回答范例

1.Vue中的diff算法称为patching算法，它由Snabbdom修改而来，虚拟DOM要想转化为真实DOM就需要通过patch方法转换。

2.最初Vue1.x视图中每个依赖均有更新函数对应，可以做到精准更新，因此并不需要虚拟DOM和patching算法支持，但是这样粒度过细导致Vue1.x无法承载较大应用；Vue 2.x中为了降低Watcher粒度，每个组件只有一个Watcher与之对应，此时就需要引入patching算法才能精确找到发生变化的地方并高效更新。

3.vue中diff执行的时刻是组件内响应式数据变更触发实例执行其更新函数时，更新函数会再次执行render函数获得最新的虚拟DOM，然后执行patch函数，并传入新旧两次虚拟DOM，通过比对两者找到变化的地方，最后将其转化为对应的DOM操作。

* * *

4.patch过程是一个递归过程，遵循深度优先、同层比较的策略；以vue3的patch为例：

+   首先判断两个节点是否为相同同类节点，不同则删除重新创建
+   如果双方都是文本则更新文本内容
+   如果双方都是元素节点则递归更新子元素，同时更新元素属性
+   更新子节点时又分了几种情况：
    +   新的子节点是文本，老的子节点是数组则清空，并设置文本；
    +   新的子节点是文本，老的子节点是文本则直接更新文本；
    +   新的子节点是数组，老的子节点是文本则清空文本，并创建新子节点数组中的子元素；
    +   新的子节点是数组，老的子节点也是数组，那么比较两组子节点，更新细节blabla

5.  vue3中引入的更新策略：编译期优化patchFlags、block等

* * *

### 知其所以然

patch关键代码

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L354-L355 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L354-L355")

调试 [test-v3.html](https://juejin.cn/post/text-v3.html "text-v3.html")

* * *

## 4. 能说一说双向绑定使用和原理吗？

### **1. Vue2 双向绑定**

**使用方式:**

```html
// 默认使用
<input v-model="message">

// 自定义使用
Vue.component('my-checkbox', {
    model: {
        prop: 'checked',  // 将 ':value' 改为 ':checked'
        event: 'change'   // 将 '@input' 改为 '@change'
    },
    props: {
        checked: Boolean
    },
    template: `
        <input type="checkbox" :checked="checked" @change="$emit('change', $event.target.checked)">
    `
});
    
    // 使用
 <my-checkbox v-model="isChecked"></my-checkbox>
    
 new Vue({
     el: '#app',
     data: {
         isChecked: false
     }
});

```
- `v-model`是语法糖，默认情况下相当于`:value`和`@input`。

**底层原理**

Vue2 中的双向绑定是基于 **Object.defineProperty** 实现的。Vue2 通过`递归遍历data对象`的每一个属性，利用 `Object.defineProperty` 把每个属性转化为 getter 和 setter，从而实现数据的响应式。

- **getter**：用于收集依赖（即订阅者，通常是依赖该数据的组件或视图）。
- **setter**：在数据变更时触发更新，通知所有订阅者进行视图更新。

Vue2 响应式的局限：
- 由于 `Object.defineProperty` 只能劫持对象的属性，因此对于新增的属性或数组的变化（如新增元素）无法自动检测到，必须通过 `Vue.set()` 或 `this.$set()` 手动添加响应式。


**双向绑定更新流程:**
1. new Vue()首先执行初始化，对data执行响应化处理，这个过程发生Observe中
2. 同时对模板执行编译，找到其中动态绑定的数据，从data中获取并初始化视图，这个过程发生在Compile中
3. 同时定义⼀个更新函数和Watcher，将来对应数据变化时Watcher会调用更新函数
4. 由于data的某个key在⼀个视图中可能出现多次，所以每个key都需要⼀个管家Dep来管理多个Watcher
5. 将来data中数据⼀旦发生变化，会首先找到对应的Dep，通知所有Watcher执行更新函数

![](https://static.vue-js.com/e5369850-3ac9-11eb-85f6-6fac77c0c9b3.png)

### 2. **Vue3 双向绑定**

**使用方式:**

```html
// 默认使用
<input v-model="message">

// 自定义使用
<template>
    <input type="checkbox" :checked="checked" @change="$emit('update:checked', $event.target.checked)" />
</template>

<script>
    export default {
        props: {
            checked: Boolean  // v-model:checked的绑定值
        },
        emits: ['update:checked'] // v-model:checked的绑定值的自定义事件
    }
</script>

// 使用方法
// Vue3 中可以直接使用 v-model:propName 语法，比如 v-model:checked，这样绑定的就是 checked 属性，并触发 update:checked 事件。
<my-checkbox v-model:checked="isChecked"></my-checkbox>

<script>
    import MyCheckbox from './MyCheckbox.vue'

    export default {
        components: { MyCheckbox },
        data() {
            return {
                isChecked: false
            }
        }
    }
</script>

// 使用defineModel
<!-- Child.vue -->
<script setup>
    // model代表父组件传入属性countModel
    const model = defineModel()
    // model.value可以直接操作父组件属性countModel
    function update() {
        model.value++
    }
</script>

<template>
    <div>Parent bound v-model is: {{ model }}</div>
    <button @click="update">Increment</button>
</template>

<!-- Parent.vue -->
<Child v-model="countModel" />

```
- `v-model`是语法糖，默认情况下相当于`:modelValue`和`@update:modelValue`。

**底层原理**

Vue3 的响应式系统是基于 **Proxy** 实现的。与 Vue2 不同，Vue3 不再需要递归地遍历对象的每个属性，而是`通过 Proxy 对整个data对象进行拦截`，这样可以实现对对象新增属性和数组变化的自动响应式支持。

- **Proxy** 拦截对对象的访问（如读取、修改、删除等），并相应地处理依赖收集和更新。
- 相比于 Vue2 的 `Object.defineProperty`，`Proxy` 提供了更强大的功能，可以直接监听对象的结构变化（如新增属性或删除属性），从而更高效地实现双向绑定和响应式系统。


### **`v-model`和`sync`修饰符有什么区别:**
1. 绑定的属性
- `v-model` 默认绑定的是 `value` 属性（Vue2）或 `modelValue` 属性（Vue3），并监听 `input` 事件或 `update:modelValue` 事件。
- `sync` 修饰符可以绑定任何属性，`prop` 名可以是任意的，不限于 `value` 或 `modelValue`。

```js
    // vue2 sync修饰符的使用方法:
    // 不使用修饰符的原版
    <text-document
        v-bind:title="doc.title"
        v-on:update:title="doc.title = $event"
    ></text-document>
    // 使用sync修饰符
    <text-document v-bind:title.sync="doc.title"></text-document>
```

2. 绑定的事件
- `v-model` 默认监听 `input` 事件（Vue2）或 `update:modelValue` 事件（Vue3）。
- `sync` 修饰符会监听 `update:propName` 事件，`propName` 是自定义的属性名。

3. 使用场景
- `v-model` 适用于单一的双向绑定，通常用于输入框等需要绑定 `value` 的组件。
- `sync` 适用于对多个 `prop` 进行双向绑定，特别是需要同时绑定多个属性时。

4. Vue3 中 `sync` 的变化
- 在 Vue3 中，`sync` 修饰符已被移除，但可以通过手动绑定 `update:propName` 的方式来实现类似的功能。

* * *

## 08 - 说一说你对vue响应式理解？

### 分析

这是一道必问题目，但能回答到位的比较少。如果只是看看一些网文，通常没什么底气，经不住面试官推敲，但像我们这样即看过源码还造过轮子的，回答这个问题就会比较有底气啦。

### 答题思路：

1.  啥是响应式？
2.  为什么vue需要响应式？
3.  它能给我们带来什么好处？
4.  vue的响应式是怎么实现的？有哪些优缺点？
5.  vue3中的响应式的新变化

* * *

### 回答范例：

1.  所谓数据响应式就是**能够使数据变化可以被检测并对这种变化做出响应的机制**。
2.  MVVM框架中要解决的一个核心问题是连接数据层和视图层，通过**数据驱动**应用，数据变化，视图更新，要做到这点的就需要对数据做响应式处理，这样一旦数据发生变化就可以立即做出更新处理。
3.  以vue为例说明，通过数据响应式加上虚拟DOM和patch算法，开发人员只需要操作数据，关心业务，完全不用接触繁琐的DOM操作，从而大大提升开发效率，降低开发难度。
4.  vue2中的数据响应式会根据数据类型来做不同处理，如果是**对象则采用Object.defineProperty()**的方式定义数据拦截，当数据被访问或发生变化时，我们感知并作出响应；如果是**数组则通过覆盖数组对象原型的7个变更方法**，使这些方法可以额外的做更新通知，从而作出响应。这种机制很好的解决了数据响应化的问题，但在实际使用中也存在一些缺点：比如初始化时的递归遍历会造成性能损失；新增或删除属性时需要用户使用Vue.set/delete这样特殊的api才能生效；对于es6中新产生的Map、Set这些数据结构不支持等问题。
5.  为了解决这些问题，vue3重新编写了这一部分的实现：利用ES6的Proxy代理要响应化的数据，它有很多好处，编程体验是一致的，不需要使用特殊api，初始化性能和内存消耗都得到了大幅改善；另外由于响应化的实现代码抽取为独立的reactivity包，使得我们可以更灵活的使用它，第三方的扩展开发起来更加灵活了。

* * *

### 知其所以然

vue2响应式：

[github1s.com/vuejs/vue/b…](https://github1s.com/vuejs/vue/blob/HEAD/src/core/observer/index.js#L135-L136 "https://github1s.com/vuejs/vue/blob/HEAD/src/core/observer/index.js#L135-L136")

vue3响应式：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L89-L90 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L89-L90")

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/ref.ts#L67-L68 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/ref.ts#L67-L68")

* * *

## 面试官：为什么data属性是一个函数而不是一个对象？

## 面试官：动态给vue的data添加一个新的属性时会发生什么？怎样解决？

## 面试官：Vue.observable你有了解过吗？说说看


## v-show 与 v-if 的区别

## 2. v-if和v-for哪个优先级更高？

两个指令一起使用,会造成性能浪费,因vue2版本中v-for优先于v-if执行.

* * *

1.  文档中明确指出**永远不要把 `v-if` 和 `v-for` 同时用在同一个元素上**

2.  在**vue2中**，**v-for的优先级是高于v-if**，先执行循环再判断条件，造成浪费；在**vue3中则完全相反，v-if的优先级高于v-for**，所以v-if执行时，v-for产生的变量还不存在，就会导致报错.

3.  处理方法：

    +   使用computed或js提前过滤列表数据

    +   外层包裹template或div执行v-if判断

4.  问题原因: vue源码判断循序造成的，vue2 判断中el.for快于el.if,vue3正好相反.

```js
// vue 2x
// \vue-dev\src\compiler\codegen\index.js
export function genElement (el: ASTElement, state: CodegenState): string {
    if (el.parent) {
        el.pre = el.pre || el.parent.pre
    }
    if (el.staticRoot && !el.staticProcessed) {
        return genStatic(el, state)
    } else if (el.once && !el.onceProcessed) {
        return genOnce(el, state)
    } else if (el.for && !el.forProcessed) {
        return genFor(el, state)
    } else if (el.if && !el.ifProcessed) {
        return genIf(el, state)
    } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
        return genChildren(el, state) || 'void 0'
    } else if (el.tag === 'slot') {
        return genSlot(el, state)
    } else {
        // component or element
    }
}
```


* * *

## 28-v-once的使用场景有哪些？

### 分析

`v-once`是Vue中内置指令，很有用的API，在优化方面经常会用到，不过小伙伴们平时可能容易忽略它。

* * *

### 体验

仅渲染元素和组件一次，并且跳过未来更新

> Render the element and component once only, and skip future updates.

```xml
<!-- single element -->
<span v-once>This will never change: {{msg}}</span>
<!-- the element have children -->
<div v-once>
  <h1>comment</h1>
  <p>{{msg}}</p>
</div>
<!-- component -->
<my-component v-once :comment="msg"></my-component>
<!-- `v-for` directive -->
<ul>
  <li v-for="i in list" v-once>{{i}}</li>
</ul>
```

* * *

### 思路

0.  `v-once`是什么
1.  什么时候使用
2.  如何使用
3.  扩展`v-memo`
4.  探索原理

* * *

### 回答范例

0.  `v-once`是vue的内置指令，作用是仅渲染指定组件或元素一次，并跳过未来对其更新。
1.  如果我们有一些元素或者组件在初始化渲染之后不再需要变化，这种情况下适合使用`v-once`，这样哪怕这些数据变化，vue也会跳过更新，是一种代码优化手段。
2.  我们只需要作用的组件或元素上加上v-once即可。
3.  vue3.2之后，又增加了`v-memo`指令，可以有条件缓存部分模板并控制它们的更新，可以说控制力更强了。
4.  编译器发现元素上面有v-once时，会将首次计算结果存入缓存对象，组件再次渲染时就会从缓存获取，从而避免再次计算。

* * *

### 知其所以然

下面例子使用了v-once：

```xml
<script setup>
import { ref } from 'vue'

const msg = ref('Hello World!')
</script>

<template>
  <h1 v-once>{{ msg }}</h1>
  <input v-model="msg">
</template>
```

我们发现v-once出现后，编译器会缓存作用元素或组件，从而避免以后更新时重新计算这一部分：

```scss
// ...
return (_ctx, _cache) => {
  return (_openBlock(), _createElementBlock(_Fragment, null, [
    // 从缓存获取vnode
    _cache[0] || (
      _setBlockTracking(-1),
      _cache[0] = _createElementVNode("h1", null, [
        _createTextVNode(_toDisplayString(msg.value), 1 /* TEXT */)
      ]),
      _setBlockTracking(1),
      _cache[0]
    ),
// ...
```

* * *

## 26-你写过自定义指令吗？使用场景有哪些？

### 分析

这是一道API题，我们可能写的自定义指令少，但是我们用的多呀，多举几个例子就行。

* * *

### 体验

定义一个包含类似组件生命周期钩子的对象，钩子函数会接收指令挂钩的dom元素：

```javascript
const focus = {
  mounted: (el) => el.focus()
}

export default {
  directives: {
    // enables v-focus in template
    focus
  }
}
<input v-focus />
```

```css
<input v-focus />
```

* * *

### 思路

0.  定义
1.  何时用
2.  如何用
3.  常用指令
4.  vue3变化

* * *

### 回答范例

0.  Vue有一组默认指令，比如`v-mode`l或`v-for`，同时Vue也允许用户注册自定义指令来扩展Vue能力

1.  自定义指令主要完成一些可复用低层级DOM操作

2.  使用自定义指令分为定义、注册和使用三步：

    +   定义自定义指令有两种方式：对象和函数形式，前者类似组件定义，有各种生命周期；后者只会在mounted和updated时执行
    +   注册自定义指令类似组件，可以使用app.directive()全局注册，使用{directives:{xxx}}局部注册
    +   使用时在注册名称前加上v-即可，比如v-focus
3.  我在项目中常用到一些自定义指令，例如：

    +   复制粘贴 v-copy
    +   长按 v-longpress
    +   防抖 v-debounce
    +   图片懒加载 v-lazy
    +   按钮权限 v-premission
    +   页面水印 v-waterMarker
    +   拖拽指令 v-draggable
4.  vue3中指令定义发生了比较大的变化，主要是钩子的名称保持和组件一致，这样开发人员容易记忆，不易犯错。另外在v3.2之后，可以在setup中以一个小写v开头方便的定义自定义指令，更简单了！


* * *

### 知其所以然

编译后的自定义指令会被withDirective函数装饰，进一步处理生成的vnode，添加到特定属性中。

[sfc.vuejs.org/#eyJBcHAudn…](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcblxuY29uc3QgbXNnID0gcmVmKCdIZWxsbyBXb3JsZCEnKVxuXG5jb25zdCB2Rm9jdXMgPSB7XG4gIG1vdW50ZWQoZWwpIHtcbiAgICAvLyDojrflj5ZpbnB1dO+8jOW5tuiwg+eUqOWFtmZvY3VzKCnmlrnms5VcbiAgICBlbC5mb2N1cygpXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxoMT57eyBtc2cgfX08L2gxPlxuICA8aW5wdXQgdi1tb2RlbD1cIm1zZ1wiIHYtZm9jdXM+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0ifQ== "https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcblxuY29uc3QgbXNnID0gcmVmKCdIZWxsbyBXb3JsZCEnKVxuXG5jb25zdCB2Rm9jdXMgPSB7XG4gIG1vdW50ZWQoZWwpIHtcbiAgICAvLyDojrflj5ZpbnB1dO+8jOW5tuiwg+eUqOWFtmZvY3VzKCnmlrnms5VcbiAgICBlbC5mb2N1cygpXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxoMT57eyBtc2cgfX08L2gxPlxuICA8aW5wdXQgdi1tb2RlbD1cIm1zZ1wiIHYtZm9jdXM+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0ifQ==")

* * *


## 27-说下$attrs和$listeners的使用场景

### 分析

API考察，但$attrs和$listeners是比较少用的边界知识，而且vue3有变化，$listeners已经移除，还是有细节可说的。

* * *

### 思路

0.  这两个api的作用
1.  使用场景分析
2.  使用方式和细节
3.  vue3变化

* * *

### 体验

一个包含组件透传属性的对象。

> An object that contains the component's fallthrough attributes.

```xml
<template>
    <child-component v-bind="$attrs">
        将非属性特性透传给内部的子组件
    </child-component>
</template>
```

* * *

### 范例

0.  我们可能会有一些属性和事件没有在props中定义，这类称为非属性特性，结合v-bind指令可以直接透传给内部的子组件。
1.  这类“属性透传”常常用于包装高阶组件时往内部传递属性，常用于爷孙组件之间传参。比如我在扩展A组件时创建了组件B组件，然后在C组件中使用B，此时传递给C的属性中只有props里面声明的属性是给B使用的，其他的都是A需要的，此时就可以利用v-bind="$attrs"透传下去。
2.  最常见用法是结合v-bind做展开；$attrs本身不是响应式的，除非访问的属性本身是响应式对象。
3.  vue2中使用listeners获取事件，vue3中已移除，均合并到listeners获取事件，vue3中已移除，均合并到attrs中，使用起来更简单了。

* * *

### 原理

查看透传属性foo和普通属性bar，发现vnode结构完全相同，这说明vue3中将分辨两者工作由框架完成而非用户指定：

```xml
<template>
  <h1>{{ msg }}</h1>
  <comp foo="foo" bar="bar" />
</template>
```

```xml
<template>
  <div>
    {{$attrs.foo}} {{bar}}
  </div>
</template>
<script setup>
defineProps({
  bar: String
})
</script>
```

```php
_createVNode(Comp, {
    foo: "foo",
    bar: "bar"
})
```

* * *

[sfc.vuejs.org/#eyJBcHAudn…](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBDb21wIGZyb20gJy4vQ29tcC52dWUnXG5jb25zdCBtc2cgPSByZWYoJ0hlbGxvIFdvcmxkIScpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aDE+e3sgbXNnIH19PC9oMT5cbiAgPGNvbXAgZm9vPVwiZm9vXCIgYmFyPVwiYmFyXCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkNvbXAudnVlIjoiPHRlbXBsYXRlPlxuXHQ8ZGl2PlxuICAgIHt7JGF0dHJzLmZvb319IHt7YmFyfX1cbiAgPC9kaXY+XG48L3RlbXBsYXRlPlxuPHNjcmlwdCBzZXR1cD5cbmRlZmluZVByb3BzKHtcbiAgYmFyOiBTdHJpbmdcbn0pXG48L3NjcmlwdD4ifQ== "https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBDb21wIGZyb20gJy4vQ29tcC52dWUnXG5jb25zdCBtc2cgPSByZWYoJ0hlbGxvIFdvcmxkIScpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aDE+e3sgbXNnIH19PC9oMT5cbiAgPGNvbXAgZm9vPVwiZm9vXCIgYmFyPVwiYmFyXCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkNvbXAudnVlIjoiPHRlbXBsYXRlPlxuXHQ8ZGl2PlxuICAgIHt7JGF0dHJzLmZvb319IHt7YmFyfX1cbiAgPC9kaXY+XG48L3RlbXBsYXRlPlxuPHNjcmlwdCBzZXR1cD5cbmRlZmluZVByb3BzKHtcbiAgYmFyOiBTdHJpbmdcbn0pXG48L3NjcmlwdD4ifQ==")

* * *

## 13-watch和computed的区别以及选择?

两个重要API，反应应聘者熟练程度。

### 思路分析

1.  先看[computed](https://vuejs.org/api/reactivity-core.html#computed "https://vuejs.org/api/reactivity-core.html#computed"), [watch](https://vuejs.org/api/reactivity-core.html#watch "https://vuejs.org/api/reactivity-core.html#watch")两者定义，列举使用上的差异
2.  列举使用场景上的差异，如何选择
3.  使用细节、注意事项
4.  vue3变化

* * *

computed特点：具有响应式的返回值

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)
```

watch特点：侦测变化，执行回调

```js
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)
```

* * *

### 回答范例

1.  计算属性可以**从组件数据派生出新数据**，最常见的使用方式是设置一个函数，返回计算之后的结果，computed和methods的差异是它具备缓存性，如果依赖项不变时不会重新计算。侦听器**可以侦测某个响应式数据的变化并执行副作用**，常见用法是传递一个函数，执行副作用，watch没有返回值，但可以执行异步操作等复杂逻辑。
2.  计算属性常用场景是简化行内模板中的复杂表达式，模板中出现太多逻辑会是模板变得臃肿不易维护。侦听器常用场景是状态变化之后做一些额外的DOM操作或者异步操作。选择采用何用方案时首先看是否需要派生出新值，基本能用计算属性实现的方式首选计算属性。
3.  使用过程中有一些细节，比如计算属性也是可以传递对象，成为既可读又可写的计算属性。watch可以传递对象，设置deep、immediate等选项。
4.  vue3中watch选项发生了一些变化，例如不再能侦测一个点操作符之外的字符串形式的表达式； reactivity API中新出现了watch、watchEffect可以完全替代目前的watch选项，且功能更加强大。

* * *

### 回答范例

1.  计算属性可以**从组件数据派生出新数据**，最常见的使用方式是设置一个函数，返回计算之后的结果，computed和methods的差异是它具备缓存性，如果依赖项不变时不会重新计算。侦听器**可以侦测某个响应式数据的变化并执行副作用**，常见用法是传递一个函数，执行副作用，watch没有返回值，但可以执行异步操作等复杂逻辑。
2.  计算属性常用场景是简化行内模板中的复杂表达式，模板中出现太多逻辑会是模板变得臃肿不易维护。侦听器常用场景是状态变化之后做一些额外的DOM操作或者异步操作。选择采用何用方案时首先看是否需要派生出新值，基本能用计算属性实现的方式首选计算属性。
3.  使用过程中有一些细节，比如计算属性也是可以传递对象，成为既可读又可写的计算属性。watch可以传递对象，设置deep、immediate等选项。
4.  vue3中watch选项发生了一些变化，例如不再能侦测一个点操作符之外的字符串形式的表达式； reactivity API中新出现了watch、watchEffect可以完全替代目前的watch选项，且功能更加强大。

* * *

### 可能追问

1.  watch会不会立即执行？
2.  watch 和 watchEffect有什么差异

* * *

### 知其所以然

computed的实现

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L79-L80 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L79-L80")

ComputedRefImpl

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L26-L27 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L26-L27")

缓存性

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L59-L60 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L59-L60")

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L45-L46 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/computed.ts#L45-L46")

watch的实现

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiWatch.ts#L158-L159 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiWatch.ts#L158-L159")

* * *

## 12-说说nextTick的使用和原理？

### 分析

这道题及考察使用，有考察原理，nextTick在开发过程中应用的也较少，原理上和vue异步更新有密切关系，对于面试者考查很有区分度，如果能够很好回答此题，对面试效果有极大帮助。

### 答题思路

1.  nextTick是做什么的？
2.  为什么需要它呢？
3.  开发时何时使用它？抓抓头，想想你在平时开发中使用它的地方
4.  下面介绍一下如何使用nextTick
5.  原理解读，结合异步更新和nextTick生效方式，会显得你格外优秀

* * *

### 回答范例：

1.  [nextTick](https://staging-cn.vuejs.org/api/general.html#nexttick "https://staging-cn.vuejs.org/api/general.html#nexttick")是等待下一次 DOM 更新刷新的工具方法。

2.  Vue有个异步更新策略，意思是如果数据变化，Vue不会立刻更新DOM，而是开启一个队列，把组件更新函数保存在队列中，在同一事件循环中发生的所有数据变更会异步的批量更新。这一策略导致我们对数据的修改不会立刻体现在DOM上，此时如果想要获取更新后的DOM状态，就需要使用nextTick。

3.  开发时，有两个场景我们会用到nextTick：


+   created中想要获取DOM时；
+   响应式数据变化后获取DOM更新后的状态，比如希望获取列表更新后的高度。

4.  nextTick签名如下：`function nextTick(callback?: () => void): Promise<void>`

    所以我们只需要在传入的回调函数中访问最新DOM状态即可，或者我们可以await nextTick()方法返回的Promise之后做这件事。

5.  在Vue内部，nextTick之所以能够让我们看到DOM更新后的结果，是因为我们传入的callback会被添加到队列刷新函数(flushSchedulerQueue)的后面，这样等队列内部的更新函数都执行完毕，所有DOM操作也就结束了，callback自然能够获取到最新的DOM值。


* * *

### 知其所以然：

1.  源码解读:

组件更新函数入队：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1547-L1548 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1547-L1548")

入队函数：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/scheduler.ts#L84-L85 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/scheduler.ts#L84-L85")

nextTick定义：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/scheduler.ts#L58-L59 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/scheduler.ts#L58-L59")

2.  测试案例，test-v3.html

* * *

## 11-能说说key的作用吗？

### 分析：

这是一道特别常见的问题，主要考查大家对虚拟DOM和patch细节的掌握程度，能够反映面试者理解层次。

* * *

### 思路分析：

1.  给出结论，key的作用是用于优化patch性能
2.  key的必要性
3.  实际使用方式
4.  总结：可从源码层面描述一下vue如何判断两个节点是否相同

* * *

### 回答范例：

1.  key的作用主要是为了更高效的更新虚拟DOM。
2.  vue在patch过程中**判断两个节点是否是相同节点是key是一个必要条件**，渲染一组列表时，key往往是唯一标识，所以如果不定义key的话，vue只能认为比较的两个节点是同一个，哪怕它们实际上不是，这导致了频繁更新元素，使得整个patch过程比较低效，影响性能。
3.  实际使用中在渲染一组列表时key必须设置，而且必须是唯一标识，应该避免使用数组索引作为key，这可能导致一些隐蔽的bug；vue中在使用相同标签元素过渡切换时，也会使用key属性，其目的也是为了让vue可以区分它们，否则vue只会替换其内部属性而不会触发过渡效果。
4.  从源码中可以知道，vue判断两个节点是否相同时主要判断两者的key和元素类型等，因此如果不设置key，它的值就是undefined，则可能永远认为这是两个相同节点，只能去做更新操作，这造成了大量的dom更新操作，明显是不可取的。

* * *

### 知其所以然

测试代码，[test-v3.html](https://juejin.cn/post/test-v3.html "./test-v3.html")

上面案例重现的是以下过程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d6750831c024116bcac3ece9497b597~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

不使用key

![image-20220214110059028](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d393bc0c4a8946a9a20973b64633852f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

如果使用key

```js
// 首次循环patch A
A B C D E
A B F C D E

// 第2次循环patch B
B C D E
B F C D E

// 第3次循环patch E
C D E
F C D E

// 第4次循环patch D
C D
F C D

// 第5次循环patch C
C 
F C

// oldCh全部处理结束，newCh中剩下的F，创建F并插入到C前面
```

* * *

源码中找答案：

判断是否为相同节点

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L342-L343 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L342-L343")

更新时的处理

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1752-L1753 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1752-L1753")

* * *

不使用key

![image-20220214110059028](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39eb54d8c667473ca344382782f7042a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如果使用key

```js
// 首次循环patch A
A B C D E
A B F C D E

// 第2次循环patch B
B C D E
B F C D E

// 第3次循环patch E
C D E
F C D E

// 第4次循环patch D
C D
F C D

// 第5次循环patch C
C 
F C

// oldCh全部处理结束，newCh中剩下的F，创建F并插入到C前面
```

源码中找答案：

判断是否为相同节点

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L342-L343 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/vnode.ts#L342-L343")

更新时的处理

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1752-L1753 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/renderer.ts#L1752-L1753")

* * *

## 31-你是怎么处理vue项目中的错误的？

### 分析

这是一个综合应用题目，在项目中我们常常需要将App的异常上报，此时错误处理就很重要了。

这里要区分错误的类型，针对性做收集。

然后是将收集的的错误信息上报服务器。

* * *

### 思路

0.  首先区分错误类型
1.  根据错误不同类型做相应收集
2.  收集的错误是如何上报服务器的

* * *

### 回答范例

0.  应用中的错误类型分为"`接口异常`"和“`代码逻辑异常`”
1.  我们需要根据不同错误类型做相应处理：`接口异常`是我们请求后端接口过程中发生的异常，可能是请求失败，也可能是请求获得了服务器响应，但是返回的是错误状态。以Axios为例，这类异常我们可以通过封装Axios，在拦截器中统一处理整个应用中请求的错误。`代码逻辑异常`是我们编写的前端代码中存在逻辑上的错误造成的异常，vue应用中最常见的方式是使用全局错误处理函数`app.config.errorHandler`收集错误。
2.  收集到错误之后，需要统一处理这些异常：分析错误，获取需要错误信息和数据。这里应该有效区分错误类型，如果是请求错误，需要上报接口信息，参数，状态码等；对于前端逻辑异常，获取错误名称和详情即可。另外还可以收集应用名称、环境、版本、用户信息，所在页面等。这些信息可以通过vuex存储的全局状态和路由信息获取。

* * *

### 实践

axios拦截器中处理捕获异常：

```vbscript
// 响应拦截器
instance.interceptors.response.use(
  (response) => {
    return response.data;
  },
  (error) => {
    // 存在response说明服务器有响应
    if (error.response) {
      let response = error.response;
      if (response.status >= 400) {
        handleError(response);
      }
    } else {
      handleError(null);
    }
    return Promise.reject(error);
  },
);
```

* * *

vue中全局捕获异常：

```javascript
import { createApp } from 'vue'

const app = createApp(...)

app.config.errorHandler = (err, instance, info) => {
  // report error to tracking services
}
```

* * *

处理接口请求错误：

```lua
function handleError(error, type) {
  if(type == 1) {
    // 接口错误，从config字段中获取请求信息
    let { url, method, params, data } = error.config
    let err_data = {
       url, method,
       params: { query: params, body: data },
       error: error.data?.message || JSON.stringify(error.data),
    })
  }
}
```

* * *

处理前端逻辑错误：

```go
function handleError(error, type) {
  if(type == 2) {
    let errData = null
    // 逻辑错误
    if(error instanceof Error) {
      let { name, message } = error
      errData = {
        type: name,
        error: message
      }
    } else {
      errData = {
        type: 'other',
        error: JSON.strigify(error)
      }
    }
  }
}
```

* * *

## 07-Vue要做权限管理该怎么做？控制到按钮级别的权限怎么做？

### 分析

综合实践题目，实际开发中经常需要面临权限管理的需求，考查实际应用能力。

权限管理一般需求是两个：页面权限和按钮权限，从这两个方面论述即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/631e5a9510f349e488227498ec6212e9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

### 思路

1.  权限管理需求分析：页面和按钮权限
2.  权限管理的实现方案：分后端方案和前端方案阐述
3.  说说各自的优缺点

* * *

### 回答范例

1.  权限管理一般需求是**页面权限**和**按钮权限**的管理

2.  具体实现的时候分后端和前端两种方案：

    前端方案会**把所有路由信息在前端配置**，通过路由守卫要求用户登录，用户**登录后根据角色过滤出路由表**。比如我会配置一个`asyncRoutes`数组，需要认证的页面在其路由的`meta`中添加一个`roles`字段，等获取用户角色之后取两者的交集，若结果不为空则说明可以访问。此过滤过程结束，剩下的路由就是该用户能访问的页面，**最后通过`router.addRoutes(accessRoutes)`方式动态添加路由**即可。

    后端方案会**把所有页面路由信息存在数据库**中，用户登录的时候根据其角色**查询得到其能访问的所有页面路由信息**返回给前端，前端**再通过`addRoutes`动态添加路由**信息

    按钮权限的控制通常会**实现一个指令**，例如`v-permission`，**将按钮要求角色通过值传给v-permission指令**，在指令的`moutned`钩子中可以**判断当前用户角色和按钮是否存在交集**，有则保留按钮，无则移除按钮。

3.  纯前端方案的优点是实现简单，不需要额外权限管理页面，但是维护起来问题比较大，有新的页面和角色需求就要修改前端代码重新打包部署；服务端方案就不存在这个问题，通过专门的角色和权限管理页面，配置页面和按钮权限信息到数据库，应用每次登陆时获取的都是最新的路由信息，可谓一劳永逸！


* * *

### 知其所以然

路由守卫

[github1s.com/PanJiaChen/…](https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/permission.js#L13-L14 "https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/permission.js#L13-L14")

路由生成

[github1s.com/PanJiaChen/…](https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/store/modules/permission.js#L50-L51 "https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/store/modules/permission.js#L50-L51")

动态追加路由

[github1s.com/PanJiaChen/…](https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/permission.js#L43-L44 "https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/permission.js#L43-L44")

* * *

### 可能的追问

1.  类似`Tabs`这类组件能不能使用`v-permission`指令实现按钮权限控制？

    ```html
    <el-tabs> 
      <el-tab-pane label="⽤户管理" name="first">⽤户管理</el-tab-pane> 
    	<el-tab-pane label="⻆⾊管理" name="third">⻆⾊管理</el-tab-pane>
    </el-tabs>
    ```


* * *

2.  服务端返回的路由信息如何添加到路由器中？

    ```js
    // 前端组件名和组件映射表
    const map = {
      //xx: require('@/views/xx.vue').default // 同步的⽅式
      xx: () => import('@/views/xx.vue') // 异步的⽅式
    }
    // 服务端返回的asyncRoutes
    const asyncRoutes = [
      { path: '/xx', component: 'xx',... }
    ]
    // 遍历asyncRoutes，将component替换为map[component]
    function mapComponent(asyncRoutes) {
      asyncRoutes.forEach(route => {
        route.component = map[route.component];
        if(route.children) {
          route.children.map(child => mapComponent(child))
        }
    	})
    }
    mapComponent(asyncRoutes)
    ```


* * *

## 24-SPA、SSR的区别是什么

我们现在编写的Vue、React和Angular应用大多数情况下都会在一个页面中，点击链接跳转页面通常是内容切换而非页面跳转，由于良好的用户体验逐渐成为主流的开发模式。但同时也会有首屏加载时间长，SEO不友好的问题，因此有了SSR，这也是为什么面试中会问到两者的区别。

### 思路分析

0.  两者概念
1.  两者优缺点分析
2.  使用场景差异
3.  其他选择

* * *

### 回答范例

0.  SPA（Single Page Application）即**单页面应用**。一般也称为 **客户端渲染**（Client Side Render）， 简称 CSR。SSR（Server Side Render）即 **服务端渲染**。一般也称为 **多页面应用**（Mulpile Page Application），简称 MPA。
1.  SPA应用只会首次请求html文件，后续只需要请求JSON数据即可，因此用户体验更好，节约流量，服务端压力也较小。但是首屏加载的时间会变长，而且SEO不友好。为了解决以上缺点，就有了SSR方案，由于HTML内容在服务器一次性生成出来，首屏加载快，搜索引擎也可以很方便的抓取页面信息。但同时SSR方案也会有性能，开发受限等问题。
2.  在选择上，如果我们的应用存在首屏加载优化需求，SEO需求时，就可以考虑SSR。
3.  但并不是只有这一种替代方案，比如对一些不常变化的静态网站，SSR反而浪费资源，我们可以考虑[预渲染](https://github.com/chrisvfritz/prerender-spa-plugin "https://github.com/chrisvfritz/prerender-spa-plugin")（prerender）方案。另外nuxt.js/next.js中给我们提供了SSG（Static Site Generate）静态网站生成方案也是很好的静态站点解决方案，结合一些CI手段，可以起到很好的优化效果，且能节约服务器资源。

* * *

### 知其所以然

内容生成上的区别：

SSR

![ss](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f486234794794c8baf4f44496d8e824f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

SPA

![sp](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5171c9f5a94447fc8f12d644ab31e078~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

部署上的区别

![部署上区别](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3af3e8cc8da34394bf7d4c3d75bf9ec8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

## 20-你了解哪些Vue性能优化方法？

### 分析

这是一道综合实践题目，写过一定数量的代码之后小伙伴们自然会开始关注一些优化方法，答得越多肯定实践经验也越丰富，是很好的题目。

### 答题思路：

根据题目描述，这里主要探讨Vue代码层面的优化

* * *

### 回答范例

+   我这里主要从Vue代码编写层面说一些优化手段，例如：代码分割、服务端渲染、组件缓存、长列表优化等

+   最常见的路由懒加载：有效拆分App尺寸，访问时才异步加载

    ```css
    const router = createRouter({
      routes: [
        // 借助webpack的import()实现异步组件
        { path: '/foo', component: () => import('./Foo.vue') }
      ]
    })
    ```


* * *

+   `keep-alive`缓存页面：避免重复创建组件实例，且能保留缓存组件状态

    ```ruby
    <router-view v-slot="{ Component }">
        <keep-alive>
        <component :is="Component"></component>
      </keep-alive>
    </router-view>
    ```


* * *

+   使用`v-show`复用DOM：避免重复创建组件

    ```xml
    <template>
      <div class="cell">
        <!-- 这种情况用v-show复用DOM，比v-if效果好 -->
        <div v-show="value" class="on">
          <Heavy :n="10000"/>
        </div>
        <section v-show="!value" class="off">
          <Heavy :n="10000"/>
        </section>
      </div>
    </template>
    ```


* * *

+   `v-for` 遍历避免同时使用 `v-if`：实际上在Vue3中已经是个错误写法

    ```xml
    <template>
        <ul>
          <li
            v-for="user in activeUsers"
            <!-- 避免同时使用，vue3中会报错 -->
            <!-- v-if="user.isActive" -->
            :key="user.id">
            {{ user.name }}
          </li>
        </ul>
    </template>
    <script>
      export default {
        computed: {
          activeUsers: function () {
            return this.users.filter(user => user.isActive)
          }
        }
      }
    </script>
    ```


* * *

+   v-once和v-memo：不再变化的数据使用`v-once`

    ```xml
    <!-- single element -->
    <span v-once>This will never change: {{msg}}</span>
    <!-- the element have children -->
    <div v-once>
      <h1>comment</h1>
      <p>{{msg}}</p>
    </div>
    <!-- component -->
    <my-component v-once :comment="msg"></my-component>
    <!-- `v-for` directive -->
    <ul>
      <li v-for="i in list" v-once>{{i}}</li>
    </ul>
    ```

    按条件跳过更新时使用`v-memo`：下面这个列表只会更新选中状态变化项

    ```css
    <div v-for="item in list" :key="item.id" v-memo="[item.id === selected]">
      <p>ID: {{ item.id }} - selected: {{ item.id === selected }}</p>
      <p>...more child nodes</p>
    </div>
    ```

    > [vuejs.org/api/built-i…](https://vuejs.org/api/built-in-directives.html#v-memo "https://vuejs.org/api/built-in-directives.html#v-memo")


* * *

+   长列表性能优化：如果是大数据长列表，可采用虚拟滚动，只渲染少部分区域的内容

    ```ini
    <recycle-scroller
      class="items"
      :items="items"
      :item-size="24"
    >
      <template v-slot="{ item }">
        <FetchItemView
          :item="item"
          @vote="voteItem(item)"
        />
      </template>
    </recycle-scroller>
    ```

    > 一些开源库：
    >
    > +   [vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller "https://github.com/Akryum/vue-virtual-scroller")
    > +   [vue-virtual-scroll-grid](https://github.com/rocwang/vue-virtual-scroll-grid "https://github.com/rocwang/vue-virtual-scroll-grid")


* * *

+   事件的销毁：Vue 组件销毁时，会自动解绑它的全部指令及事件监听器，但是仅限于组件本身的事件。

    ```javascript
    export default {
      created() {
        this.timer = setInterval(this.refresh, 2000)
      },
      beforeUnmount() {
        clearInterval(this.timer)
      }
    }
    ```


* * *

+   图片懒加载

    对于图片过多的页面，为了加速页面加载速度，所以很多时候我们需要将页面内未出现在可视区域内的图片先不做加载， 等到滚动到可视区域后再去加载。

    ```ini
    <img v-lazy="/static/img/1.png">
    ```

    > 参考项目：[vue-lazyload](https://github.com/hilongjw/vue-lazyload "https://github.com/hilongjw/vue-lazyload")


* * *

+   第三方插件按需引入

    像`element-plus`这样的第三方组件库可以按需引入避免体积太大。

    ```javascript
    import { createApp } from 'vue';
    import { Button, Select } from 'element-plus';
    
    const app = createApp()
    app.use(Button)
    app.use(Select)
    ```


* * *

+   子组件分割策略：较重的状态组件适合拆分

    ```xml
    <template>
      <div>
        <ChildComp/>
      </div>
    </template>
    
    <script>
    export default {
      components: {
        ChildComp: {
          methods: {
            heavy () { /* 耗时任务 */ }
          },
          render (h) {
            return h('div', this.heavy())
          }
        }
      }
    }
    </script>
    ```

    但同时也不宜过度拆分组件，尤其是为了所谓组件抽象将一些不需要渲染的组件特意抽出来，组件实例消耗远大于纯dom节点。参考：[vuejs.org/guide/best-…](https://vuejs.org/guide/best-practices/performance.html#avoid-unnecessary-component-abstractions "https://vuejs.org/guide/best-practices/performance.html#avoid-unnecessary-component-abstractions")


* * *

+   服务端渲染/静态网站生成：SSR/SSG

    如果SPA应用有首屏渲染慢的问题，可以考虑SSR、SSG方案优化。参考[SSR Guide](https://vuejs.org/guide/scaling-up/ssr.html "https://vuejs.org/guide/scaling-up/ssr.html")


* * *

## 面试官：vue项目本地开发完成后部署到服务器后报404是什么原因呢？

## 25-vue-loader是什么？它有什么作用？

### 分析

这是一道工具类的原理题目，相当有深度，具有不错的人才区分度。

* * *

### 体验

使用官方提供的SFC playground可以很好的体验`vue-loader`。

[sfc.vuejs.org](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcblxuY29uc3QgbXNnID0gcmVmKCdIZWxsbyBXb3JsZCEnKVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGgxPnt7IG1zZyB9fTwvaDE+XG4gIDxpbnB1dCB2LW1vZGVsPVwibXNnXCI+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0ifQ== "https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcblxuY29uc3QgbXNnID0gcmVmKCdIZWxsbyBXb3JsZCEnKVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGgxPnt7IG1zZyB9fTwvaDE+XG4gIDxpbnB1dCB2LW1vZGVsPVwibXNnXCI+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0ifQ==")

* * *

有了`vue-loader`加持，我们才可以以SFC的方式快速编写代码。

```xml
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data() {
    return {
      msg: 'Hello world!',
    }
  },
}
</script>

<style>
.example {
  color: red;
}
</style>
```

* * *

### 思路

+   `vue-loader`是什么东东
+   `vue-loader`是做什么用的
+   `vue-loader`何时生效
+   `vue-loader`如何工作

* * *

### 回答范例

0.  `vue-loader`是用于处理单文件组件（SFC，Single-File Component）的webpack loader
1.  因为有了`vue-loader`，我们就可以在项目中编写SFC格式的Vue组件，我们可以把代码分割为<template>、<script>和<style>，代码会异常清晰。结合其他loader我们还可以用Pug编写<template>，用SASS编写<style>，用TS编写<script>。我们的<style>还可以单独作用当前组件。
2.  webpack打包时，会以loader的方式调用`vue-loader`
3.  `vue-loader`被执行时，它会对SFC中的每个语言块用单独的loader链处理。最后将这些单独的块装配成最终的组件模块。

* * *

### 知其所以然

0.  `vue-loader`会调用`@vue/compiler-sfc`模块解析SFC源码为一个描述符（Descriptor），然后为每个语言块生成import代码，返回的代码类似下面：

```javascript
// source.vue被vue-loader处理之后返回的代码

// import the <template> block
import render from 'source.vue?vue&type=template'
// import the <script> block
import script from 'source.vue?vue&type=script'
export * from 'source.vue?vue&type=script'
// import <style> blocks
import 'source.vue?vue&type=style&index=1'

script.render = render
export default script
```

* * *

2.  我们想要script块中的内容被作为js处理（当然如果是`<script lang="ts">`被作为ts处理），这样我们想要webpack把配置中跟.js匹配的规则都应用到形如`source.vue?vue&type=script`的这个请求上。例如我们对所有\*.js配置了babel-loader，这个规则将被克隆并应用到所在Vue SFC的

```javascript
import script from 'source.vue?vue&type=script'
```

将被展开为：

```javascript
import script from 'babel-loader!vue-loader!source.vue?vue&type=script'
```

类似的，如果我们对`.sass`文件配置了`style-loader` + `css-loader` + `sass-loader`，对下面的代码：

```ini
<style scoped lang="scss">
```

`vue-loader`将会返回给我们下面结果：

```arduino
import 'source.vue?vue&type=style&index=1&scoped&lang=scss'
```

* * *

然后webpack会展开如下：

```arduino
import 'style-loader!css-loader!sass-loader!vue-loader!source.vue?vue&type=style&index=1&scoped&lang=scss'
```

0.  当处理展开请求时，`vue-loader`将被再次调用。这次，loader将会关注那些有查询串的请求，且仅针对特定块，它会选中特定块内部的内容并传递给后面匹配的loader。
1.  对于`<script>`块，处理到这就可以了，但是`<template>` 和 `<style>`还有一些额外任务要做，比如：

+   需要用Vue 模板编译器编译template，从而得到render函数
+   需要对`<style scoped>`中的CSS做后处理（post-process），该操作在css-loader之后但在style-loader之前

实现上这些附加的loader需要被注入到已经展开的loader链上，最终的请求会像下面这样：

```arduino
// <template lang="pug">
import 'vue-loader/template-loader!pug-loader!source.vue?vue&type=template'

// <style scoped lang="scss">
import 'style-loader!vue-loader/style-post-loader!css-loader!sass-loader!vue-loader!source.vue?vue&type=style&index=1&scoped&lang=scss'
```

* * *

## 16-从0到1自己构架一个vue项目，说说有哪些步骤、哪些重要插件、目录结构你会怎么组织

综合实践类题目，考查实战能力。没有什么绝对的正确答案，把平时工作的重点有条理的描述一下即可。

### 思路

1.  构建项目，创建项目基本结构
2.  引入必要的插件：
3.  代码规范：prettier，eslint
4.  提交规范：husky，lint-staged
5.  其他常用：svg-loader，vueuse，nprogress
6.  常见目录结构

* * *

### 回答范例

1.  从0创建一个项目我大致会做以下事情：项目构建、引入必要插件、代码规范、提交规范、常用库和组件

2.  目前vue3项目我会用vite或者create-vue创建项目

3.  接下来引入必要插件：路由插件vue-router、状态管理vuex/pinia、ui库我比较喜欢element-plus和antd-vue、http工具我会选axios

4.  其他比较常用的库有vueuse，nprogress，图标可以使用vite-svg-loader

5.  下面是代码规范：结合prettier和eslint即可

6.  最后是提交规范，可以使用husky，lint-staged，commitlint


* * *

7.  目录结构我有如下习惯： `.vscode`：用来放项目中的 vscode 配置

    `plugins`：用来放 vite 插件的 plugin 配置

    `public`：用来放一些诸如 页头icon 之类的公共文件，会被打包到dist根目录下

    `src`：用来放项目代码文件

    `api`：用来放http的一些接口配置

    `assets`：用来放一些 CSS 之类的静态资源

    `components`：用来放项目通用组件

    `layout`：用来放项目的布局

    `router`：用来放项目的路由配置

    `store`：用来放状态管理Pinia的配置

    `utils`：用来放项目中的工具方法类

    `views`：用来放项目的页面文件


* * *

## 17-实际工作中，你总结的vue最佳实践有哪些？

看到这样的题目，可以用以下图片来回答：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/893abf278d56465e81ac83492b150684~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

### 思路

查看vue官方文档：

风格指南：[vuejs.org/style-guide…](https://vuejs.org/style-guide/ "https://vuejs.org/style-guide/")

性能：[vuejs.org/guide/best-…](https://vuejs.org/guide/best-practices/performance.html#overview "https://vuejs.org/guide/best-practices/performance.html#overview")

安全：[vuejs.org/guide/best-…](https://vuejs.org/guide/best-practices/security.html "https://vuejs.org/guide/best-practices/security.html")

访问性：[vuejs.org/guide/best-…](https://vuejs.org/guide/best-practices/accessibility.html "https://vuejs.org/guide/best-practices/accessibility.html")

发布：[vuejs.org/guide/best-…](https://vuejs.org/guide/best-practices/production-deployment.html "https://vuejs.org/guide/best-practices/production-deployment.html")

* * *

### 回答范例

我从编码风格、性能、安全等方面说几条：

1.  编码风格方面：
    +   命名组件时使用“多词”风格避免和HTML元素冲突
    +   使用“细节化”方式定义属性而不是只有一个属性名
    +   属性名声明时使用“驼峰命名”，模板或jsx中使用“肉串命名”
    +   使用v-for时务必加上key，且不要跟v-if写在一起
2.  性能方面：
    +   路由懒加载减少应用尺寸
    +   利用SSR减少首屏加载时间
    +   利用v-once渲染那些不需要更新的内容
    +   一些长列表可以利用虚拟滚动技术避免内存过度占用
    +   对于深层嵌套对象的大数组可以使用shallowRef或shallowReactive降低开销
    +   避免不必要的组件抽象

* * *

3.  安全：
    +   不使用不可信模板，例如使用用户输入拼接模板：`template: <div> + userProvidedString + </div>`
    +   小心使用v-html，:url，:style等，避免html、url、样式等注入
4.  等等......

* * *



# Vue3 API & 面试题

## API参考: [vue 3.x](vue3.md)

## 33-Composition API 与 Options API 有什么不同

### 分析

Vue3最重要更新之一就是Composition API，它具有一些列优点，其中不少是针对Options API暴露的一些问题量身打造。是Vue3推荐的写法，因此掌握好Composition API应用对掌握好Vue3至关重要。

![image-20220629182639250](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b7b4bfafa5c4507be726d273161c3c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

[vuejs.org/guide/extra…](https://vuejs.org/guide/extras/composition-api-faq.html "https://vuejs.org/guide/extras/composition-api-faq.html")

* * *

### 体验

Composition API能更好的组织代码，下面这个代码用options api实现

![image-20220629183203082](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a280d15533ad4481a6121064940eae1b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如果用composition api可以提取为useCount()，用于组合、复用

![image-20220629184919471](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1aa01aeeff224815bef1356b773fae2d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

### 思路

+   总述不同点
+   composition api动机
+   两者选择

* * *

### 回答范例

+   `Composition API`是一组API，包括：Reactivity API、生命周期钩子、依赖注入，使用户可以通过导入函数方式编写vue组件。而`Options API`则通过声明组件选项的对象形式编写组件。
+   `Composition API`最主要作用是能够简洁、高效复用逻辑。解决了过去`Options API`中`mixins`的各种缺点；另外`Composition API`具有更加敏捷的代码组织能力，很多用户喜欢`Options API`，认为所有东西都有固定位置的选项放置代码，但是单个组件增长过大之后这反而成为限制，一个逻辑关注点分散在组件各处，形成代码碎片，维护时需要反复横跳，`Composition API`则可以将它们有效组织在一起。最后`Composition API`拥有更好的类型推断，对ts支持更友好，`Options API`在设计之初并未考虑类型推断因素，虽然官方为此做了很多复杂的类型体操，确保用户可以在使用`Options API`时获得类型推断，然而还是没办法用在mixins和provide/inject上。
+   Vue3首推`Composition API`，但是这会让我们在代码组织上多花点心思，因此在选择上，如果我们项目属于中低复杂度的场景，`Options API`仍是一个好选择。对于那些大型，高扩展，强维护的项目上，`Composition API`会获得更大收益。

* * *

### 可能的追问

+   `Composition API`能否和`Options API`一起使用？

* * *

## 22-ref和reactive异同

这是`Vue3`数据响应式中非常重要的两个概念，自然的，跟我们写代码关系也很大。

* * *

### 体验

ref：[vuejs.org/api/reactiv…](https://vuejs.org/api/reactivity-core.html#ref "https://vuejs.org/api/reactivity-core.html#ref")

```scss
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

* * *

reactive：[vuejs.org/api/reactiv…](https://vuejs.org/api/reactivity-core.html#reactive "https://vuejs.org/api/reactivity-core.html#reactive")

```ini
const obj = reactive({ count: 0 })
obj.count++
```

* * *

### 回答思路

0.  两者概念
1.  两者使用场景
2.  两者异同
3.  使用细节
4.  原理

* * *

### 回答范例

0.  `ref`接收内部值（inner value）返回响应式`Ref`对象，`reactive`返回响应式代理对象
1.  从定义上看`ref`通常用于处理单值的响应式，`reactive`用于处理对象类型的数据响应式
2.  两者均是用于构造响应式数据，但是`ref`主要解决原始值的响应式问题
3.  ref返回的响应式数据在JS中使用需要加上`.value`才能访问其值，在视图中使用会自动脱ref，不需要`.value`；ref可以接收对象或数组等非原始值，但内部依然是`reactive`实现响应式；reactive内部如果接收Ref对象会自动脱ref；使用展开运算符(...)展开reactive返回的响应式对象会使其失去响应性，可以结合toRefs()将值转换为Ref对象之后再展开。
4.  reactive内部使用Proxy代理传入对象并拦截该对象各种操作（trap），从而实现响应式。ref内部封装一个RefImpl类，并设置get value/set value，拦截用户对值的访问，从而实现响应式。

* * *

### 知其所以然

reactive实现响应式：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L90-L91 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L90-L91")

ref实现响应式：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/ref.ts#L73-L74 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/ref.ts#L73-L74")

* * *

## 23-watch和watchEffect异同

我们经常性需要侦测响应式数据的变化，vue3中除了watch之外又出现了watchEffect，不少同学会混淆这两个api。

* * *

### 体验

`watchEffect`立即运行一个函数，然后被动地追踪它的依赖，当这些依赖改变时重新执行该函数。

> Runs a function immediately while reactively tracking its dependencies and re-runs it whenever the dependencies are changed.

```scss
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> logs 0

count.value++
// -> logs 1
```

* * *

`watch`侦测一个或多个响应式数据源并在数据源变化时调用一个回调函数。

> Watches one or more reactive data sources and invokes a callback function when the sources change.

```scss
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)
```

* * *

### 思路

0.  给出两者定义
1.  给出场景上的不同
2.  给出使用方式和细节
3.  原理阐述

* * *

### 范例

0.  `watchEffect`立即运行一个函数，然后被动地追踪它的依赖，当这些依赖改变时重新执行该函数。`watch`侦测一个或多个响应式数据源并在数据源变化时调用一个回调函数。
1.  `watchEffect(effect)`是一种特殊`watch`，传入的函数既是依赖收集的数据源，也是回调函数。如果我们不关心响应式数据变化前后的值，只是想拿这些数据做些事情，那么`watchEffect`就是我们需要的。watch更底层，可以接收多种数据源，包括用于依赖收集的getter函数，因此它完全可以实现watchEffect的功能，同时由于可以指定getter函数，依赖可以控制的更精确，还能获取数据变化前后的值，因此如果需要这些时我们会使用watch。
2.  `watchEffect`在使用时，传入的函数会立刻执行一次。`watch`默认情况下并不会执行回调函数，除非我们手动设置`immediate`选项。
3.  从实现上来说，`watchEffect(fn)`相当于`watch(fn,fn,{immediate:true})`

* * *

### 知其所以然

`watchEffect`定义：[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiWatch.ts#L80-L81 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiWatch.ts#L80-L81")

```javascript
export function watchEffect(
  effect: WatchEffect,
  options?: WatchOptionsBase
): WatchStopHandle {
  return doWatch(effect, null, options)
}
```

* * *

`watch`定义如下：[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiWatch.ts#L158-L159 "https://github1s.com/vuejs/core/blob/HEAD/packages/runtime-core/src/apiWatch.ts#L158-L159")

```typescript
export function watch<T = any, Immediate extends Readonly<boolean> = false>(
  source: T | WatchSource<T>,
  cb: any,
  options?: WatchOptions<Immediate>
): WatchStopHandle {
  return doWatch(source as any, cb, options)
}
```

很明显`watchEffect`就是一种特殊的`watch`实现。

* * *

## 1 - 你知道哪些vue3新特性

### 分析

官网列举的最值得注意的新特性：[v3-migration.vuejs.org/](https://v3-migration.vuejs.org/ "https://v3-migration.vuejs.org/")

![image-20220210165307624](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e52235d31934130914925042b96e3a7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

也就是下面这些：

+   Composition API
+   SFC Composition API语法糖
+   Teleport传送门
+   Fragments片段
+   Emits选项
+   自定义渲染器
+   SFC CSS变量
+   Suspense

以上这些是api相关，另外还有很多框架特性也不能落掉。

* * *

### 回答范例

1.  api层面Vue3新特性主要包括：Composition API、SFC Composition API语法糖、Teleport传送门、Fragments 片段、Emits选项、自定义渲染器、SFC CSS变量、Suspense

2.  另外，Vue3.0在框架层面也有很多亮眼的改进：


+   更快
    +   虚拟DOM重写
    +   编译器优化：静态提升、patchFlags、block等
    +   基于Proxy的响应式系统
+   更小：更好的摇树优化
+   更容易维护：TypeScript + 模块化
+   更容易扩展
    +   独立的响应化模块
    +   自定义渲染器

* * *

### 知其所以然

体验编译器优化

[sfc.vuejs.org/](https://sfc.vuejs.org/ "https://sfc.vuejs.org/")

reactive实现

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L90-L91 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L90-L91")

* * *

## 2-Vue 3.0的设计目标是什么？做了哪些优化?

### 分析

还是问新特性，陈述典型新特性，分析其给你带来的变化即可。

* * *

### 思路

从以下几方面分门别类阐述：易用性、性能、扩展性、可维护性、开发体验等

* * *

### 范例

0.  Vue3的最大设计目标是替代Vue2（皮一下），为了实现这一点，Vue3在以下几个方面做了很大改进，如：易用性、框架性能、扩展性、可维护性、开发体验等
1.  易用性方面主要是API简化，比如`v-model`在Vue3中变成了Vue2中`v-model`和`sync`修饰符的结合体，用户不用区分两者不同，也不用选择困难。类似的简化还有用于渲染函数内部生成VNode的`h(type, props, children)`，其中`props`不用考虑区分属性、特性、事件等，框架替我们判断，易用性大增。
2.  开发体验方面，新组件`Teleport`传送门、`Fragments` 、`Suspense`等都会简化特定场景的代码编写，`SFC Composition API`语法糖更是极大提升我们开发体验。
3.  扩展性方面提升如独立的`reactivity`模块，`custom renderer` API等
4.  可维护性方面主要是`Composition API`，更容易编写高复用性的业务逻辑。还有对TypeScript支持的提升。
5.  性能方面的改进也很显著，例如编译期优化、基于`Proxy`的响应式系统
6.  。。。

* * *

### 可能的追问

0.  Vue3做了哪些编译优化？
1.  `Proxy`和`defineProperty`有什么不同？

* * *

## 3-Vue3.0 性能提升体现在哪些方面？

### 分析

vue3在设计时有几个目标：更小、更快、更友好，这些多数适合性能相关，因此可以围绕介绍。

* * *

### 思路

+   总述和性能相关的新特性
+   逐个说细节
+   能说点原理更佳

* * *

### 回答范例

+   我分别从代码、编译、打包三方面介绍vue3性能方面的提升
+   代码层面性能优化主要体现在全新响应式API，基于Proxy实现，初始化时间和内存占用均大幅改进；
+   编译层面做了更多编译优化处理，比如静态提升、动态标记、事件缓存，区块等，可以有效跳过大量diff过程；
+   打包时更好的支持tree-shaking，因此整体体积更小，加载更快

* * *

### 体验

通过playground体验编译优化：[sfc.vuejs.org](https://sfc.vuejs.org/ "https://sfc.vuejs.org")

* * *

### 知其所以然

为什么基于Proxy更快了：初始化时懒处理，用户访问才做拦截处理，初始化更快：

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/baseHandlers.ts#L136-L137 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/baseHandlers.ts#L136-L137")

轻量的依赖关系保存：利用WeakMap、Map和Set保存响应式数据和副作用之间的依赖关系

[github1s.com/vuejs/core/…](https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/effect.ts#L19-L20 "https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/effect.ts#L19-L20")

## 4-Vue3.0里为什么要用 Proxy 替代 defineProperty ？

### 分析

Vue3中最重大的更新之一就是响应式模块`reactivity`的重写。主要的修改就是`Proxy`替换`defineProperty`实现响应式。

此变化主要是从性能方面考量。

### 思路

+   属性拦截的几种方式
+   defineProperty的问题
+   Proxy的优点
+   其他考量

### 回答范例

+   JS中做属性拦截常见的方式有三：: [defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty")，[getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get")/[setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set") 和[Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy").
+   Vue2中使用`defineProperty`的原因是，2013年时只能用这种方式。由于该API存在一些局限性，比如对于数组的拦截有问题，为此vue需要专门为数组响应式做一套实现。另外不能拦截那些新增、删除属性；最后`defineProperty`方案在初始化时需要深度递归遍历待处理的对象才能对它进行完全拦截，明显增加了初始化的时间。
+   以上两点在Proxy出现之后迎刃而解，不仅可以对数组实现拦截，还能对Map、Set实现拦截；另外Proxy的拦截也是懒处理行为，如果用户没有访问嵌套对象，那么也不会实施拦截，这就让初始化的速度和内存占用都改善了。
+   当然Proxy是有兼容性问题的，IE完全不支持，所以如果需要IE兼容就不合适

### 知其所以然

Proxy属性拦截的原理：利用get、set、deleteProperty这三个trap实现拦截

```javascript
function reactive(obj) {
    return new Proxy(obj, {
        get(target, key) {},
        set(target, key, val) {},
        deleteProperty(target, key){}
    })
}
```

Object.defineProperty属性拦截原理：利用get、set这两个trap实现拦截

```vbnet
function defineReactive(obj, key, val) {
    Object.defineReactive(obj, key, {
        get(key) {},
        set(key, val) {}
    })
}
```

很容易看出两者的区别！

* * *

# Vue Router

## API参考: [vue router](vue_router.md)

## 32-History模式和Hash模式有何区别？

### 分析

vue-router有3个模式，其中两个更为常用，那便是history和hash。

两者差别主要在显示形式和部署上。

* * *

### 体验

vue-router4.x中设置模式已经变化：

```bash
const router = createRouter({
  history: createWebHashHistory(), // hash模式
  history: createWebHistory(),     // history模式
})
```

用起来一模一样

```ini
<router-link to="/about">Go to About</router-link>
```

区别只在url形式

```ruby
// hash
// 浏览器里的形态：http://xx.com/#/about
// history
// 浏览器里的形态：http://xx.com/about
```

### 思路

+   区别
+   详细阐述
+   实现

* * *

### 回答范例

+   vue-router有3个模式，其中history和hash更为常用。两者差别主要在显示形式、seo和部署上。
+   hash模式在地址栏显示的时候是已哈希的形式：#/xxx，这种方式使用和部署简单，但是不会被搜索引擎处理，seo有问题；history模式则建议用在大部分web项目上，但是它要求应用在部署时做特殊配置，服务器需要做回退处理，否则会出现刷新页面404的问题。
+   底层实现上其实hash是一种特殊的history实现。

* * *

### 知其所以然

hash是一种特殊的history实现：

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/history/hash.ts#L31-L32 "https://github1s.com/vuejs/router/blob/HEAD/src/history/hash.ts#L31-L32")

* * *

## 1 - 怎么定义动态路由？怎么获取传过来的动态参数？

### 分析

API题目，考查基础能力，不容有失，尽可能说的详细。

### 思路

1.  什么是动态路由
2.  什么时候使用动态路由，怎么定义动态路由
3.  参数如何获取
4.  细节、注意事项

* * *

### 回答范例

1.  很多时候，我们需要**将给定匹配模式的路由映射到同一个组件**，这种情况就需要定义动态路由。
2.  例如，我们可能有一个 `User` 组件，它应该对所有用户进行渲染，但用户 ID 不同。在 Vue Router 中，我们可以在路径中使用一个动态字段来实现，例如：`{ path: '/users/:id', component: User }`，其中`:id`就是路径参数
3.  *路径参数* 用冒号 `:` 表示。当一个路由被匹配时，它的 *params* 的值将在每个组件中以 `this.$route.params` 的形式暴露出来。
4.  参数还可以有多个，例如`/users/:username/posts/:postId`；除了 `$route.params` 之外，`$route` 对象还公开了其他有用的信息，如 `$route.query`、`$route.hash` 等。

* * *

### 可能的追问

1.  如何响应动态路由参数的变化

[router.vuejs.org/zh/guide/es…](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E5%93%8D%E5%BA%94%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0%E7%9A%84%E5%8F%98%E5%8C%96 "https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E5%93%8D%E5%BA%94%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0%E7%9A%84%E5%8F%98%E5%8C%96")

2.  我们如何处理404 Not Found路由

[router.vuejs.org/zh/guide/es…](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E6%8D%95%E8%8E%B7%E6%89%80%E6%9C%89%E8%B7%AF%E7%94%B1%E6%88%96-404-not-found-%E8%B7%AF%E7%94%B1 "https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E6%8D%95%E8%8E%B7%E6%89%80%E6%9C%89%E8%B7%AF%E7%94%B1%E6%88%96-404-not-found-%E8%B7%AF%E7%94%B1")

* * *

## 2-如果让你从零开始写一个vue路由，说说你的思路

### 思路分析：

首先思考vue路由要解决的问题：用户点击跳转链接内容切换，页面不刷新。

+   借助hash或者history api实现url跳转页面不刷新
+   同时监听hashchange事件或者popstate事件处理跳转
+   根据hash值或者state值从routes表中匹配对应component并渲染之

* * *

### 回答范例：

一个SPA应用的路由需要解决的问题是**页面跳转内容改变同时不刷新**，同时路由还需要以插件形式存在，所以：

1.  首先我会定义一个`createRouter`函数，返回路由器实例，实例内部做几件事：
    +   保存用户传入的配置项
    +   监听hash或者popstate事件
    +   回调里根据path匹配对应路由
2.  将router定义成一个Vue插件，即实现install方法，内部做两件事：
    +   实现两个全局组件：router-link和router-view，分别实现页面跳转和内容显示
    +   定义两个全局变量：$route和$router，组件内可以访问当前路由和路由器实例

* * *

### 知其所以然：

+   createRouter如何创建实例

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/router.ts#L355-L356 "https://github1s.com/vuejs/router/blob/HEAD/src/router.ts#L355-L356")

+   事件监听

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/history/html5.ts#L314-L315 "https://github1s.com/vuejs/router/blob/HEAD/src/history/html5.ts#L314-L315") RouterView

+   页面跳转RouterLink

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/RouterLink.ts#L184-L185 "https://github1s.com/vuejs/router/blob/HEAD/src/RouterLink.ts#L184-L185")

+   内容显示RouterView

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/RouterView.ts#L43-L44 "https://github1s.com/vuejs/router/blob/HEAD/src/RouterView.ts#L43-L44")

* * *

## 3-怎么实现路由懒加载呢？

### 分析

这是一道应用题。当打包应用时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问时才加载对应组件，这样就会更加高效。

```javascript
// 将
// import UserDetails from './views/UserDetails'
// 替换为
const UserDetails = () => import('./views/UserDetails')

const router = createRouter({
  // ...
  routes: [{ path: '/users/:id', component: UserDetails }],
})
```

参考[router.vuejs.org/zh/guide/ad…](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html "https://router.vuejs.org/zh/guide/advanced/lazy-loading.html")

* * *

### 思路

0.  必要性
1.  何时用
2.  怎么用
3.  使用细节

* * *

### 回答范例

0.  当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。利用路由懒加载我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样会更加高效，是一种优化手段。

1.  一般来说，对所有的路由**都使用动态导入**是个好主意。

2.  给`component`选项配置一个返回 Promise 组件的函数就可以定义懒加载路由。例如：

    `{ path: '/users/:id', component: () => import('./views/UserDetails') }`

3.  结合注释`() => import(/* webpackChunkName: "group-user" */ './UserDetails.vue')`可以做webpack代码分块

    vite中结合[rollupOptions](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E4%BD%BF%E7%94%A8-vite "https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E4%BD%BF%E7%94%A8-vite")定义分块

4.  路由中不能使用异步组件


* * *

### 知其所以然

`component` (和 `components`) 配置如果接收一个返回 Promise 组件的函数，Vue Router **只会在第一次进入页面时才会获取这个函数**，然后使用缓存数据。

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/navigationGuards.ts#L292-L293 "https://github1s.com/vuejs/router/blob/HEAD/src/navigationGuards.ts#L292-L293")

* * *

## 4-router-link和router-view是如何起作用的？

### 分析

vue-router中两个重要组件`router-link`和`router-view`，分别起到导航作用和内容渲染作用，但是回答如何生效还真有一定难度哪！

### 思路

+   两者作用
+   阐述使用方式
+   原理说明

### 回答范例

+   vue-router中两个重要组件`router-link`和`router-view`，分别起到路由导航作用和组件内容渲染作用
+   使用中router-link默认生成一个a标签，设置to属性定义跳转path。实际上也可以通过custom和插槽自定义最终的展现形式。router-view是要显示组件的占位组件，可以嵌套，对应路由配置的嵌套关系，配合name可以显示具名组件，起到更强的布局作用。
+   router-link组件内部根据custom属性判断如何渲染最终生成节点，内部提供导航方法navigate，用户点击之后实际调用的是该方法，此方法最终会修改响应式的路由变量，然后重新去routes匹配出数组结果，router-view则根据其所处深度deep在匹配数组结果中找到对应的路由并获取组件，最终将其渲染出来。

### 知其所以然

+   RouterLink定义

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/RouterLink.ts#L184-L185 "https://github1s.com/vuejs/router/blob/HEAD/src/RouterLink.ts#L184-L185")

+   RouterView定义

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/RouterView.ts#L43-L44 "https://github1s.com/vuejs/router/blob/HEAD/src/RouterView.ts#L43-L44")

## 5-Vue-router 除了 router-link 怎么实现跳转

### 分析

vue-router导航有两种方式：`声明式导航`和`编程方式导航`

* * *

### 体验

声明式导航

```ini
<router-link to="/about">Go to About</router-link>
```

编程导航

```php
// literal string path
router.push('/users/eduardo')

// object with path
router.push({ path: '/users/eduardo' })

// named route with params to let the router build the url
router.push({ name: 'user', params: { username: 'eduardo' } })
```

* * *

### 思路

+   两种方式
+   分别阐述使用方式
+   区别和选择
+   原理说明

* * *

### 回答范例

+   vue-router导航有两种方式：`声明式导航`和`编程方式导航`
+   声明式导航方式使用`router-link`组件，添加to属性导航；编程方式导航更加灵活，可传递调用router.push()，并传递path字符串或者RouteLocationRaw对象，指定path、name、params等信息
+   如果页面中简单表示跳转链接，使用router-link最快捷，会渲染一个a标签；如果页面是个复杂的内容，比如商品信息，可以添加点击事件，使用编程式导航
+   实际上内部两者调用的导航函数是一样的

* * *

### 知其所以然

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/src/RouterLink.ts#L240-L241 "https://github1s.com/vuejs/router/blob/HEAD/src/RouterLink.ts#L240-L241")

routerlink点击跳转，调用的是navigate方法

![image-20220626173129790](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/015d1efe389c4f4391622876fd880b71~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

navigate内部依然调用的push

## 6-在什么场景下会用到嵌套路由？

### 分析

应用的有些界面是由多层级组件组合而来的，这种情况下，url各部分通常对应某个嵌套的组件，vue-router中就可以使用嵌套路由表示这种关系：[router.vuejs.org/guide/essen…](https://router.vuejs.org/guide/essentials/nested-routes.html "https://router.vuejs.org/guide/essentials/nested-routes.html")

![image-20220628071220515](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3da6683da0204acda653d94fdedfd9c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

### 体验

定义嵌套路由，对应上图嵌套关系：

```sql
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        // UserProfile 会被渲染在 User 组件中的 <router-view> 里
        path: 'profile',
        component: UserProfile,
      },
      {
        // UserPosts 会被渲染在 User 组件中的 <router-view> 里
        path: 'posts',
        component: UserPosts,
      },
    ],
  },
]
```

* * *

### 思路

+   概念和使用场景
+   使用方式
+   实现原理

* * *

### 回答范例

+   平时开发中，应用的有些界面是由多层级组件组合而来的，这种情况下，url各部分通常对应某个嵌套的组件，vue-router中可以使用嵌套路由表示这种关系
+   表现形式是在两个路由间切换时，它们有公用的视图内容。此时通常提取一个父组件，内部放上，从而形成物理上的嵌套，和逻辑上的嵌套对应起来
+   定义嵌套路由时使用`children`属性组织嵌套关系
+   原理上是在router-view组件内部判断当前router-view处于嵌套层级的深度，讲这个深度作为匹配组件数组matched的索引，获取对应渲染组件，渲染之

* * *

### 知其所以然

router-view获取自己所在的深度：默认0，加1之后传给后代，同时根据深度获取匹配路由。

![image-20220628074750827](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edc6b0d7873640d5aa2cb73a9006a971~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

## 7-vue-router中如何保护路由？

### 分析

路由保护在应用开发过程中非常重要，几乎每个应用都要做各种路由权限管理，因此相当考察使用者基本功。

* * *

### 体验

全局守卫：

```javascript
const router = createRouter({ ... })

router.beforeEach((to, from) => {
  // ...
  // 返回 false 以取消导航
  return false
})
```

路由独享守卫：

```javascript
const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: (to, from) => {
      // reject the navigation
      return false
    },
  },
]
```

组件内的守卫：

```javascript
const UserDetails = {
  template: `...`,
  beforeRouteEnter(to, from) {
    // 在渲染该组件的对应路由被验证前调用
  },
  beforeRouteUpdate(to, from) {
    // 在当前路由改变，但是该组件被复用时调用
  },
  beforeRouteLeave(to, from) {
    // 在导航离开渲染该组件的对应路由时调用
  },
}
```

* * *

### 思路

+   路由守卫的概念
+   路由守卫的使用
+   路由守卫的原理

* * *

+   vue-router中保护路由的方法叫做路由守卫，主要用来通过跳转或取消的方式守卫导航。
+   路由守卫有三个级别：全局，路由独享，组件级。影响范围由大到小，例如全局的router.beforeEach()，可以注册一个全局前置守卫，每次路由导航都会经过这个守卫，因此在其内部可以加入控制逻辑决定用户是否可以导航到目标路由；在路由注册的时候可以加入单路由独享的守卫，例如beforeEnter，守卫只在进入路由时触发，因此只会影响这个路由，控制更精确；我们还可以为路由组件添加守卫配置，例如beforeRouteEnter，会在渲染该组件的对应路由被验证前调用，控制的范围更精确了。
+   用户的任何导航行为都会走navigate方法，内部有个guards队列按顺序执行用户注册的守卫钩子函数，如果没有通过验证逻辑则会取消原有的导航。

* * *

### 知其所以然

runGuardQueue(guards)链式的执行用户在各级别注册的守卫钩子函数，通过则继续下一个级别的守卫，不通过进入catch流程取消原本导航。

![image-20220630193341500](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a70d8c83e3254b39800e9d646731039d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

[github1s.com/vuejs/route…](https://github1s.com/vuejs/router/blob/HEAD/packages/router/src/router.ts#L808-L809 "https://github1s.com/vuejs/router/blob/HEAD/packages/router/src/router.ts#L808-L809")




# Vuex

## API参考: [vuex](vuex.md)

## 1 - 简单说一说你对vuex理解？

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb128aee87e5424a83511deee98f1702~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 思路

1.  给定义
2.  必要性阐述
3.  何时使用
4.  拓展：一些个人思考、实践经验等

* * *

### 范例

1.  Vuex 是一个专为 Vue.js 应用开发的**状态管理模式 + 库**。它采用集中式存储，管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
2.  我们期待以一种简单的“单向数据流”的方式管理应用，即状态 -> 视图 -> 操作单向循环的方式。但当我们的应用遇到**多个组件共享状态**时，比如：多个视图依赖于同一状态或者来自不同视图的行为需要变更同一状态。此时单向数据流的简洁性很容易被破坏。因此，我们有必要把组件的共享状态抽取出来，以一个全局单例模式管理。通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。这是vuex存在的必要性，它和react生态中的redux之类是一个概念。
3.  Vuex 解决状态管理的同时引入了不少概念：例如state、mutation、action等，是否需要引入还需要根据应用的实际情况衡量一下：如果不打算开发大型单页应用，使用 Vuex 反而是繁琐冗余的，一个简单的 [store 模式](https://v3.cn.vuejs.org/guide/state-management.html#%E4%BB%8E%E9%9B%B6%E6%89%93%E9%80%A0%E7%AE%80%E5%8D%95%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86 "https://v3.cn.vuejs.org/guide/state-management.html#%E4%BB%8E%E9%9B%B6%E6%89%93%E9%80%A0%E7%AE%80%E5%8D%95%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86")就足够了。但是，如果要构建一个中大型单页应用，Vuex 基本是标配。
4.  我在使用vuex过程中感受到一些blabla

* * *

### 可能的追问

1.  vuex有什么缺点吗？你在开发过程中有遇到什么问题吗？
2.  action和mutation的区别是什么？为什么要区分它们？

* * *

## 2-你有使用过vuex的module吗？

这是基本应用能力考察，稍微上点规模的项目都要拆分vuex模块便于维护。

* * *

### 体验

[vuex.vuejs.org/zh/guide/mo…](https://vuex.vuejs.org/zh/guide/modules.html "https://vuex.vuejs.org/zh/guide/modules.html")

```php
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}
const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}
const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
store.getters.c // -> moduleA里的getters
store.commit('d') // -> 能同时触发子模块中同名mutation
store.dispatch('e') // -> 能同时触发子模块中同名action
```

* * *

### 思路

0.  概念和必要性
1.  怎么拆
2.  使用细节
3.  优缺点

* * *

### 范例

0.  用过module，项目规模变大之后，单独一个store对象会过于庞大臃肿，通过模块方式可以拆分开来便于维护
1.  可以按之前规则单独编写子模块代码，然后在主文件中通过`modules`选项组织起来：`createStore({modules:{...}})`
2.  不过使用时要注意访问子模块状态时需要加上注册时模块名：`store.state.a.xxx`，但同时`getters`、`mutations`和`actions`又在全局空间中，使用方式和之前一样。如果要做到完全拆分，需要在子模块加上`namespace`选项，此时再访问它们就要加上命名空间前缀。
3.  很显然，模块的方式可以拆分代码，但是缺点也很明显，就是使用起来比较繁琐复杂，容易出错。而且类型系统支持很差，不能给我们带来帮助。pinia显然在这方面有了很大改进，是时候切换过去了。

* * *

### 可能的追问

0.  用过pinia吗？都做了哪些改善？

* * *

## 3-如果让你从零开始写一个vuex，说说你的思路

### 思路分析

这个题目很有难度，首先思考`vuex`解决的问题：存储用户全局状态并提供管理状态API。

+   `vuex`需求分析
+   如何实现这些需求

* * *

### 回答范例

0.  官方说`vuex`是一个状态管理模式和库，并确保这些状态以可预期的方式变更。可见要实现一个`vuex`：

    +   要实现一个`Store`存储全局状态
    +   要提供修改状态所需API：`commit(type, payload)`, `dispatch(type, payload)`
1.  实现`Store`时，可以定义Store类，构造函数接收选项options，设置属性state对外暴露状态，提供commit和dispatch修改属性state。这里需要设置state为响应式对象，同时将Store定义为一个Vue插件。

2.  `commit(type, payload)`方法中可以获取用户传入`mutations`并执行它，这样可以按用户提供的方法修改状态。 `dispatch(type, payload)`类似，但需要注意它可能是异步的，需要返回一个Promise给用户以处理异步结果。


* * *

### 实践

Store的实现：

```kotlin
class Store {
    constructor(options) {
        this.state = reactive(options.state)
        this.options = options
    }
    commit(type, payload) {
        this.options.mutations[type].call(this, this.state, payload)
    }
}
```

### 知其所以然

Vuex中Store的实现：

[github1s.com/vuejs/vuex/…](https://github1s.com/vuejs/vuex/blob/HEAD/src/store.js#L19-L20 "https://github1s.com/vuejs/vuex/blob/HEAD/src/store.js#L19-L20")

## 4-vuex中actions和mutations有什么区别？

### 题目分析

`mutations`和`actions`是`vuex`带来的两个独特的概念。新手程序员容易混淆，所以面试官喜欢问。

我们只需记住修改状态只能是`mutations`，`actions`只能通过提交`mutation`修改状态即可。

* * *

### 体验

看下面例子可知，`Action` 类似于 `mutation`，不同在于：

+   `Action` 提交的是 `mutation`，而不是直接变更状态。
+   `Action` 可以包含任意异步操作。

```php
const store = createStore({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

* * *

### 答题思路

0.  给出两者概念说明区别
1.  举例说明应用场景
2.  使用细节不同
3.  简单阐述实现上差异

* * *

### 回答范例

0.  官方文档说：更改 Vuex 的 store 中的状态的唯一方法是提交 `mutation`，`mutation` 非常类似于事件：每个 `mutation` 都有一个字符串的**类型 (type)**和一个** 回调函数 (handler)** 。`Action` 类似于 `mutation`，不同在于：`Action`可以包含任意异步操作，但它不能修改状态， 需要提交`mutation`才能变更状态。
1.  因此，开发时，包含异步操作或者复杂业务组合时使用action；需要直接修改状态则提交mutation。但由于dispatch和commit是两个API，容易引起混淆，实践中也会采用统一使用dispatch action的方式。
2.  调用dispatch和commit两个API时几乎完全一样，但是定义两者时却不甚相同，mutation的回调函数接收参数是state对象。action则是与Store实例具有相同方法和属性的上下文context对象，因此一般会解构它为`{commit, dispatch, state}`，从而方便编码。另外dispatch会返回Promise实例便于处理内部异步结果。
3.  实现上commit(type)方法相当于调用`options.mutations[type](state)`；`dispatch(type)`方法相当于调用`options.actions[type](store)`，这样就很容易理解两者使用上的不同了。

* * *

### 知其所以然

我们可以像下面这样简单实现`commit`和`dispatch`，从而辨别两者不同：

```kotlin
class Store {
    constructor(options) {
        this.state = reactive(options.state)
        this.options = options
    }
    commit(type, payload) {
        // 传入上下文和参数1都是state对象
        this.options.mutations[type].call(this.state, this.state, payload)
    }
    dispatch(type, payload) {
        // 传入上下文和参数1都是store本身
        this.options.actions[type].call(this, this, payload)
    }
}
```

## 5-使用vue渲染大量数据时应该怎么优化？说下你的思路！

### 分析

企业级项目中渲染大量数据的情况比较常见，因此这是一道非常好的综合实践题目。

### 思路

0.  描述大数据量带来的问题
1.  分不同情况做不同处理
2.  总结一下

### 回答

0.  在大型企业级项目中经常需要渲染大量数据，此时很容易出现卡顿的情况。比如大数据量的表格、树。

1.  处理时要根据情况做不通处理：

    +   可以采取分页的方式获取，避免渲染大量数据
    +   [vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller "https://github.com/Akryum/vue-virtual-scroller")等虚拟滚动方案，只渲染视口范围内的数据
    +   如果不需要更新，可以使用`v-once`方式只渲染一次
    +   通过[v-memo](https://vuejs.org/api/built-in-directives.html#v-memo "https://vuejs.org/api/built-in-directives.html#v-memo")可以缓存结果，结合`v-for`使用，避免数据变化时不必要的VNode创建
    +   可以采用懒加载方式，在用户需要的时候再加载数据，比如tree组件子树的懒加载
2.  总之，还是要看具体需求，首先从设计上避免大数据获取和渲染；实在需要这样做可以采用虚表的方式优化渲染；最后优化更新，如果不需要更新可以v-once处理，需要更新可以v-memo进一步优化大数据更新性能。其他可以采用的是交互方式优化，无线滚动、懒加载等方案。


## 6-怎么监听vuex数据的变化？

### 分析

vuex数据状态是响应式的，所以状态变视图跟着变，但是有时还是需要知道数据状态变了从而做一些事情。

既然状态都是响应式的，那自然可以`watch`，另外vuex也提供了订阅的API：`store.subscribe()`。

* * *

### 思路

+   总述知道的方法
+   分别阐述用法
+   选择和场景

* * *

### 回答范例

+   我知道几种方法：

    +   可以通过watch选项或者watch方法监听状态
    +   可以使用vuex提供的API：store.subscribe()
+   watch选项方式，可以以字符串形式监听`$store.state.xx`；subscribe方式，可以调用store.subscribe(cb),回调函数接收mutation对象和state对象，这样可以进一步判断mutation.type是否是期待的那个，从而进一步做后续处理。

+   watch方式简单好用，且能获取变化前后值，首选；subscribe方法会被所有commit行为触发，因此还需要判断mutation.type，用起来略繁琐，一般用于vuex插件中。


### 实践

watch方式

```arduino
const app = createApp({
    watch: {
      '$store.state.counter'() {
        console.log('counter change!');
      }
    }
  })
```

subscribe方式：

```javascript
  store.subscribe((mutation, state) => {
    if (mutation.type === 'add') {
      console.log('counter change in subscribe()!');
    }
  })
```

## 7-页面刷新后vuex的state数据丢失怎么解决？

### 分析

这是一道应用题目，很容易想到使用`localStorage`或数据库存储并还原状态。

但是如何优雅编写代码还是能体现认知水平。

* * *

### 体验

可以从`localStorage`中获取作为状态初始值：

```javascript
const store = createStore({
  state () {
    return {
      count: localStorage.getItem('count')
    }
  }
})
```

业务代码中，提交修改状态同时保存最新值：虽说实现了，但是每次还要手动刷新localStorage不太优雅

```javascript
store.commit('increment')
localStorage.setItem('count', store.state.count)
```

* * *

### 思路

+   问题描述
+   解决方法
+   谈个人理解
+   三方库原理探讨

* * *

### 回答范例

+   vuex只是在内存保存状态，刷新之后就会丢失，如果要持久化就要存起来。
+   localStorage就很合适，提交mutation的时候同时存入localStorage，store中把值取出作为state的初始值即可。
+   这里有两个问题，不是所有状态都需要持久化；如果需要保存的状态很多，编写的代码就不够优雅，每个提交的地方都要单独做保存处理。这里就可以利用vuex提供的subscribe方法做一个统一的处理。甚至可以封装一个vuex插件以便复用。
+   类似的插件有vuex-persist、vuex-persistedstate，内部的实现就是通过订阅mutation变化做统一处理，通过插件的选项控制哪些需要持久化

* * *

### 知其所以然

可以看一下vuex-persist内部确实是利用subscribe实现的

[github.com/championswi…](https://github.com/championswimmer/vuex-persist/blob/master/src/index.ts#L277 "https://github.com/championswimmer/vuex-persist/blob/master/src/index.ts#L277")

* * *

## 8-你觉得vuex有什么缺点？

### 分析

相较于redux，vuex已经相当简便好用了。但模块的使用比较繁琐，对ts支持也不好。

* * *

### 体验

使用模块：用起来比较繁琐，使用模式也不统一，基本上得不到类型系统的任何支持

```php
const store = createStore({
  modules: {
    a: moduleA
  }
})
store.state.a // -> 要带上 moduleA 的key，内嵌模块的话会很长，不得不配合mapState使用
store.getters.c // -> moduleA里的getters，没有namespaced时又变成了全局的
store.getters['a/c'] // -> 有namespaced时要加path，使用模式又和state不一样
store.commit('d') // -> 没有namespaced时变成了全局的，能同时触发多个子模块中同名mutation
store.commit('a/d') // -> 有namespaced时要加path，配合mapMutations使用感觉也没简化
```

* * *

### 思路

+   先夸再贬
+   使用感受
+   解决方案

* * *

### 回答范例

+   vuex利用响应式，使用起来已经相当方便快捷了。但是在使用过程中感觉模块化这一块做的过于复杂，用的时候容易出错，还要经常查看文档
+   比如：访问state时要带上模块key，内嵌模块的话会很长，不得不配合mapState使用，加不加namespaced区别也很大，getters，mutations，actions这些默认是全局，加上之后必须用字符串类型的path来匹配，使用模式不统一，容易出错；对ts的支持也不友好，在使用模块时没有代码提示。
+   之前Vue2项目中用过[vuex-module-decorators](https://github.com/championswimmer/vuex-module-decorators "https://github.com/championswimmer/vuex-module-decorators")的解决方案，虽然类型支持上有所改善，但又要学一套新东西，增加了学习成本。pinia出现之后使用体验好了很多，Vue3 + pinia会是更好的组合。

* * *

### 知其所以然

下面我们来看看vuex中`store.state.x.y`这种嵌套的路径是怎么搞出来的。

首先是子模块安装过程：父模块状态`parentState`上面设置了子模块名称`moduleName`，值为当前模块`state`对象。放在上面的例子中相当于：`store.state['x'] = moduleX.state`。此过程是递归的，那么`store.state.x.y`安装时就是：`store.state['x']['y'] = moduleY.state`。

```scss
if (!isRoot && !hot) {
    // 获取父模块state
    const parentState = getNestedState(rootState, path.slice(0, -1))
    // 获取子模块名称
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
        // 把子模块state设置到父模块上
        parentState[moduleName] = module.state
    })
}
```

这下大家明白了吧！

> 源码地址：[github1s.com/vuejs/vuex/…](https://github1s.com/vuejs/vuex/blob/HEAD/src/store-util.js#L102-L115 "https://github1s.com/vuejs/vuex/blob/HEAD/src/store-util.js#L102-L115")

* * *

# Pinia
参考: [pinia](pinia.md)
