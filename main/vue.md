# Vue 面试题

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
+ 正规修改方法: 事件传递/$parent

* * *

## 8.Vue中如何扩展一个组件
+   逻辑扩展：mixins、extends、composition api
+   内容扩展: slots

    ```js
    // 混入mixins
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

    ```html
    <!--插槽 slots-->
    <!--子组件-->
    <div>
      <slot></slot>
    </div>
    
    <!--父组件-->
    <div>
       <Child>插槽内容</Child>
    </div>
    ```

    ```js
    // extends
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

## 9.怎么缓存当前的组件？缓存后怎么更新？

开发中缓存组件使用keep-alive组件，keep-alive是vue内置组件，keep-alive包裹动态组件component时，会缓存不活动的组件实例，而不是销毁它们，这样在组件切换过程中将状态保留在内存中，防止重复渲染DOM

+ vue3中结合vue-router时变化较大，之前是`keep-alive`包裹`router-view`，现在需要反过来用`router-view`包裹`keep-alive`：
    ```vue
    // 结合属性include和exclude可以明确指定缓存哪些组件或排除缓存指定组件。
    <router-view v-slot="{ Component }">
      <keep-alive>
        <component :is="Component"></component>
      </keep-alive>
    </router-view>
    ```

+ 缓存后获取数据

    ```js
    // beforeRouteEnter 路由进入前触发
    beforeRouteEnter(to, from, next) {
      next(vm=>{
        console.log(vm)
        // 每次进入路由执行
        vm.getData()  // 获取数据
      })
    }
    ```
  
    ```js
    // 在`keep-alive`缓存的组件被激活的时候
    activated(){
          this.getData() // 获取数据
    }
    ```

* * *

## 10. 什么是递归组件？举个例子说明下？

组件通过组件名称引用它自己，这种情况就是递归组件,必须要设置组件 `name`。
实际开发中类似Tree、Menu这类组件，它们的节点往往包含子节点，子节点结构和父节点往往是相同的。这类组件的数据往往也是树形结构，这种都是使用递归组件的典型场景。

```vue
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
</template>
<script>
export default {
  name: 'TreeItem',
  // ...
}
</script>
```

* * *

## 11. 异步组件是什么？使用场景有哪些？

异步组件,可以控制懒加载,利于打包器的代码分割。

```javascript
import { defineAsyncComponent } from 'vue'

// 方式1: defineAsyncComponent定义异步组件
const AsyncComp = defineAsyncComponent(() => {
  // 加载函数返回Promise
  return new Promise((resolve, reject) => {
    // ...可以从服务器加载组件
    resolve(/* loaded component */)
  })
})

// 方式2: import函数
const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)
```

* * *

## 12. 组件与插件的区别
+ 组件：用于构建 UI,独立单元可复用，局部或全局注册，主要用于页面渲染。
+ 插件：用于扩展 Vue 的全局功能，通过 Vue.use() 注册，一般作用于整个应用。

* * *

## 13.说说你对虚拟 DOM 的理解？

在 Vue 中，**虚拟 DOM（Virtual DOM）** 是一种在 JavaScript 中用对象树（虚拟节点树）来模拟真实 DOM 的技术，Vue 使用虚拟 DOM 来模拟页面结构的状态，当数据发生变化时，通过虚拟 DOM 更新真实 DOM，实现高效的页面更新。

+ 什么是虚拟 DOM？

    虚拟 DOM 是一个轻量级的 JavaScript 对象树，用来描述真实 DOM 的结构。

+ 为什么使用虚拟 DOM？

    **直接操作真实 DOM 代价较高**:尤其是页面复杂、DOM 节点多的时候。

    **提升性能**：Vue 通过“最小化 DOM 操作”优化了渲染性能，能有效减少 DOM 重绘和重排的次数。
    
    **提高开发效率**：开发者不必手动操作 DOM，Vue 会在数据变化时自动更新界面。
    
    **跨平台支持**：虚拟 DOM 能独立于浏览器环境，实现更灵活的渲染，如服务端渲染（SSR）和原生应用渲染（如 Vue Native）。

+ 虚拟 DOM 的工作流程:

    **初始渲染**：在第一次渲染时，Vue 将模板编译为渲染函数（`render` 函数），生成一个虚拟 DOM 树。
    
    **创建真实 DOM**：Vue 将虚拟 DOM 转换为真实 DOM 并挂载到页面上。
    
    **数据更新**：当数据变化时，Vue 会触发组件的重新渲染，通过 `render` 函数生成一个新的虚拟 DOM 树（新树）。
    
    **虚拟 DOM 比对（Diff 算法）**：Vue 将新树和旧树进行对比，找出不同点。Vue 使用 **Diff 算法**来高效定位变更部分，生成一个更新补丁（Patch）。
    
    **更新真实 DOM**：Vue 将补丁应用到真实 DOM 中，完成页面更新。

+ Vue 2 与 Vue 3 的虚拟 DOM 差异

    **Vue 2**：基于 `Snabbdom` 库实现的虚拟 DOM，提供了基础的 Diff 算法。
    
    **Vue 3**：Vue 3 的虚拟 DOM 和 Diff 算法经过重新设计，更加高效、灵活。Vue 3 的编译器会在生成虚拟 DOM 时加入 **静态标记**，可以跳过不变的节点，从而减少 Diff 计算，提高渲染性能。

+ 虚拟 DOM 示例

```javascript
// 初始状态的虚拟 DOM
const oldVNode = {
  tag: 'div',
  children: [
    { tag: 'p', text: 'Hello' },
    { tag: 'p', text: 'World' }
  ]
};

// 数据更新后的虚拟 DOM
const newVNode = {
  tag: 'div',
  children: [
    { tag: 'p', text: 'Hello' },
    // 在数据更新时，Vue 会对 `newVNode` 和 `oldVNode` 进行 Diff 比较，发现第二个子节点的文本发生了变化，只会更新这个 `p` 标签，而不会重新渲染整个 `div` 节点。
    { tag: 'p', text: 'Vue' } // 文本从 'World' 变为 'Vue'
  ]
};
```

* * *

## 14. 你了解diff算法吗？

### 执行时机:
`响应式数据变更`触发实例执行其更新函数时，更新函数会再次执行render函数获得最新的虚拟DOM，然后`执行patch函数`，并传入新旧两次虚拟DOM对比变化，最后更新变化的DOM。

* * *

### 1. Vue1.x 无diff算法应用
vue 1.x视图中每个依赖均有更新函数对应，可以做到精准更新，因此并不需要虚拟DOM和patching算法支持.粒度过细导致Vue1.x无法承载较大应用.
Vue 2 降低Watcher粒度，每个组件只有一个Watcher与之对应，使用patching算法精确寻找变化部分并更新。

### 2. **Vue 2 的 diff 算法**
Vue 2 使用的是基于 [Snabbdom](https://github.com/snabbdom/snabbdom) 的虚拟 DOM 实现，diff 算法的工作流程如下：

#### 工作机制：
1. **同层比较,深度优先遍历**：
    - Vue 2 在比较虚拟 DOM 树时采用深度优先遍历的方式。
    - 每次遇到节点变动，都递归对比其子节点。

2. **按类型优化**：
    - 节点类型相同时，会对比属性。
    - 如果节点类型不一致，直接替换整个节点。

3. **静态节点优化有限**：
    - Vue 2 的静态节点只在模板编译阶段进行标记优化，运行时仍然可能重复渲染。

4. **依赖 Watcher**：
    - Vue 2 的响应式系统基于 `Dep` 和 `Watcher`。
    - 每个组件、每个数据都有独立的watcher，会增加性能开销。

---

#### Vue 2 的 Diff 过程

1. **对比节点类型（类型是否相同）**
    - 首先检查新旧虚拟节点（VNode）的类型：
        - 如果类型不同（例如 `div` vs `span`），则直接移除旧节点，插入新节点。
        - 如果类型相同，继续比较属性和子节点。

2. **比较属性（Props）**
    - 遍历节点的属性对象：
        - 如果属性值发生变化，则更新对应的 DOM 属性。
        - 如果新节点没有某个属性但旧节点有，则删除该属性。

3. **比较子节点（Children）**  
   子节点的比较策略有两种：
    - **非 Keyed 列表：**  
      对于无 `key` 的子节点，Vue 2 使用简单的按顺序比较，逐个检查新旧节点：
        - 如果新旧节点都存在且类型相同，则递归比较子节点。
        - 如果不相同，直接替换。
        - 新增或删除的节点直接添加或移除。
    - **Keyed 列表：**  
      使用 `key` 来标记节点后，会尝试找到对应的旧节点进行复用：
        - 通过线性搜索匹配 `key`。
        - 如果找不到匹配的节点，则插入新节点。

4. **递归遍历子节点树**  
   Vue 2 会递归地遍历整个虚拟 DOM 树，直至所有的节点都被处理。

#### 示例：Keyed 列表的对比
旧节点：`[a, b, c]`  
新节点：`[b, c, d]`  
步骤：
1. 删除 `a`。
2. 复用 `b` 和 `c`。
3. 插入 `d`。

---

### 3. **Vue 3 的 diff 算法**

#### 在vue2x基础上优化：
1. **优化的静态提升**：
    - Vue 3 会在编译阶段将模板中的静态内容标记为静态节点，并直接提升为常量。
    - 静态节点在 diff 阶段被完全跳过。

```html
<div>
  <div>foo</div> <!-- 静态提升 -->
  <div>bar</div> <!-- 静态提升 -->
  <div>{{ dynamic }}</div>
</div>
```

2. **Block Tree (树结构打平) 优化**：
    - Vue 3 引入了 **Block Tree** 的概念，区别处理动态内容和静态内容。
    - 静态部分不参与 diff，对比时只关注动态节点，减少了对整个树的遍历。

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

3. **更高效的节点对比**：
    - Vue 3 使用更高效的 **PatchFlag**（补丁标记）机制，标记哪些地方需要更新。
    - 比较过程中可以直接定位到动态节点，避免了 Vue 2 中的大量递归。

```html
<!-- PatchFlag 标记的更加仔细,自动推断类型,提升对比性能 -->
<!-- 仅含 class 绑定 -->
<div :class="{ active }"></div>

<!-- 仅含 id 和 value 绑定 -->
<input :id="id" :value="value">

<!-- 仅含文本子节点 -->
<div>{{ dynamic }}</div>
```

4. **扁平化的优化策略**：
    - 子节点对比采用 **双端对比算法**：
        - 从两端向中间对比，尽量减少无效的遍历。
        - 常见于 `keyed` 的列表节点。
    - 双端对比减少对比的复杂度o(n)。

5. **响应式系统的改进**：
    - Vue 3 的响应式系统基于 `Proxy`，完全取代了 Vue 2 的 `Object.defineProperty`。
    - 动态追踪依赖，避免了 Vue 2 中的 `Watcher` 过多问题。

---

#### Vue 3 的 Diff 过程

Vue 3 在 Vue 2 的基础上对 diff 算法做了以下优化，具体对比过程如下：

1. **标记动态节点**
    - Vue 3 在编译阶段生成模板时，通过 `PatchFlag` 标记每个节点的动态部分，例如：
        - 动态属性。
        - 动态子节点。
        - 条件渲染和循环渲染的动态部分。
    - 在运行时 diff 时，只需要关注被标记的部分，直接跳过静态节点。

2. **Block Tree 优化**
    - Vue 3 的虚拟 DOM 树被分成 **Block Tree**，静态节点被直接提升到父级 block 外。
    - diff 时仅需要比较动态 block，极大减少了对静态节点的递归。

3. **对子节点的优化：双端对比算法**  
   Vue 3 对子节点列表使用 **双端对比算法**，即从两端同时开始比较：
    - 首先比较头节点和尾节点。
    - 如果无法匹配，则逐步从中间寻找匹配项。
    - 当两端指针重叠时，停止比较。

#### 示例：Keyed 列表的对比
旧节点：`[a, b, c, d, e]`  
新节点：`[b, c, e, f, g]`  
步骤：
1. 从两端开始对比，先匹配：
    - `a` 被删除（左端不匹配）。
    - `g` 被插入（右端不匹配）。
2. 继续对比：
    - `b` 和 `c` 复用。
    - `d` 被删除。
    - `f` 被插入。
3. 最终结果：高效完成列表更新。

4. **直接更新真实 DOM**  
   Vue 3 不再依赖递归，而是通过逐层比较动态节点进行更新，这种方式减少了递归调用的开销。

---

### 4. **对比总结**

| 特性          | Vue 2                      | Vue 3                |
|-------------|----------------------------|----------------------|
| **核心算法来源**  | 基于 Snabbdom                | 自研虚拟 DOM             |
| **静态节点处理**  | 全部递归比较                     | 静态提升，直接跳过            |
| **动态节点对比**  | 递归遍历子节点                    | `PatchFlag` 精确定位动态节点 |
| **子节点处理策略** | 线性搜索 Keyed 列表，按顺序比较        | 双端对比算法，性能更高          |
| **列表对比**    | 递归对比,依赖顺序和 key 匹配          | 双端对比减少无效操作           |
| **响应式实现**   | 基于 `Object.defineProperty` | 基于 `Proxy`           |
| **性能**      | 依赖watch/递归，依赖手动优化          | 减少递归更高效，默认优化效果更好     |


* * *

## 15. 能说一说双向绑定使用和原理吗？

### **1. Vue2 双向绑定**

**使用方式:**

```html
// 默认使用
// v-model是语法糖，默认情况下相当于`:value`和`@input`。
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

// 无setup自定义使用
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

// 推荐使用defineModel
<!-- Child.vue -->
<script setup>
    // model代表父组件传入属性countModel
    const model = defineModel()
    // model.value可以直接操作父组件属性countModel
    function update() {
        model.value++
    }
    // 自定动态绑定
    defineProps(['modelValue'])
    defineEmits(['update:modelValue'])
</script>

<template>
    <div>Parent bound v-model is: {{ model }}</div>
    <button @click="update">Increment</button>
    <!--自定动态绑定-->
    <!--v-model是语法糖，默认情况下相当于`:modelValue`和`@update:modelValue`。-->
    <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)"/>
</template>

<!-- Parent.vue -->
<Child v-model="countModel" />
<!--自定动态绑定-->
<Child :modelValue="countModel" @update:modelValue="$event => (countModel = $event)"/>
<Child v-model:modelValue="countModel" />

```

**底层原理**

Vue3 的响应式系统是基于 **Proxy** 实现的。与 Vue2 不同，Vue3 不再需要递归地遍历对象的每个属性，而是`通过 Proxy 对整个data/ref/reactive进行拦截`，这样可以实现对对象新增属性和数组变化的自动响应式支持。

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

## 16. 说一说你对vue响应式理解？

响应式系统是 Vue 框架的核心特性，实现数据驱动的视图更新.

### **一、Vue 2 的响应式**

1. **数据初始化**：
    - 在组件初始化时，Vue 会深度优先遍历 `data` 对象的所有属性，通过 `Object.defineProperty` 为其设置 `getter` 和 `setter`。
2. **依赖追踪**：
    - 当模板中使用数据时，Vue 会触发 `getter`，将对应的组件 `Watcher` 添加到依赖中。
3. **响应更新**：
    - 当数据变化时，触发 `setter`，通知 `Watcher` 重新执行更新操作。

**缺点：**
- **无法监听属性新增或删除**：
  Vue 2 无法通过 `Object.defineProperty` 监听对象新增/删除的属性，只能使用 `Vue.set()` / `Vue.delete()`手动添加响应式。
- **数组监听的局限性**：
  Vue 2 无法直接监听数组的索引变化或长度变化，需要通过特定的方法（如 `splice`）触发视图更新。
- **性能问题**：
  数据初始化时需要递归劫持整个对象，深层嵌套对象性能较低。

---

### **二、Vue 3 的响应式**

1. **代理对象**：
    - Vue 3 使用 `reactive` 或 `ref` 包装响应式数据，返回一个`Proxy`对象。
2. **依赖追踪和触发**：
    - 读取属性时（`get`），记录依赖的副作用（`effect`）。
    - 修改属性时（`set`），触发对应副作用更新视图。

**优势：**
- **对象代理**：Vue 3 使用 `Proxy` 对整个对象进行代理，而不是逐个属性进行劫持。
- **深层监听支持**：可以直接监听深层嵌套对象的变化，而无需递归劫持。
- **支持动态属性**：对象新增或删除属性可以被直接监听。
- **数组优化**：原生支持数组索引和长度变化监听。
- **性能更高**：不需要递归劫持，代理整个对象后动态追踪属性。

---

### **三、Vue 2 与 Vue 3 响应式对比**

| **特性**                    | **Vue 2**                                     | **Vue 3**                                   |
|-----------------------------|-----------------------------------------------|---------------------------------------------|
| **实现方式**                | `Object.defineProperty`                      | `Proxy`                                     |
| **深层对象监听**            | 递归遍历，每个属性单独劫持                   | 自动追踪，动态监听深层对象                 |
| **动态属性监听**            | 不支持（需手动使用 `Vue.set`）               | 支持，动态属性自动响应式                   |
| **数组监听**                | 需要特殊方法（如 `splice`）触发更新          | 原生支持索引和长度变化                     |
| **性能**                    | 初始化时递归遍历，性能较低                   | 代理整个对象，性能更高                     |
| **灵活性**                  | 固定属性劫持，动态性较差                     | 灵活代理，动态性更强                       |
| **代码复杂度**              | 较高，需手动递归处理深层嵌套                 | 较低，统一通过 `Proxy` 实现                |

* * *

## 17. 面试官：动态给vue的data添加一个新的属性时会发生什么？
1. vue2 中data直接添加新属性并不会又任何反应,因为使用`Object.defineProperty`的响应式机制,不会动态监听数据. 许使用`Vue.set()`/`Object.assign()`/`$forcecUpdated()` 等API触发页面更新.

2. vue3 中data直接添加新属性,因其使用`proxy`响应式机制,会自动更新页面.

* * *

## 18. 面试官：为什么data属性是一个函数而不是一个对象？

1. **数据隔离**：防止实例之间数据污染。
2. **支持复用**：函数构成单独的作用域(模块),每个组件实例有独立的状态。
3. **组件与根实例的功能区分**：根实例是单例，而组件需要支持多实例化。
4. **代码可维护性**：符合模块化和现代编程的最佳实践。

* * *

## 19. 面试官：Vue.observable你有了解过吗？说说看

`observable` API 提供了创建响应式对象的功能，用于实现响应式数据绑定。

---

### **一、Vue 2 的 `observable`**
Vue 2.6+ 引入的 API，用于创建一个可观察的响应式对象，主要用于共享状态管理（类似 Vuex 的轻量实现）。

**1. 使用方式**
```javascript
// 创建响应式对象
const state = Vue.observable({
  count: 0,
});

// 修改数据时会触发响应
state.count++;
console.log(state.count); // 1

// 在组件中使用
Vue.component("counter", {
  template: `<button @click="state.count++">{{ state.count }}</button>`,
  data() {
    return { state };
  },
});
```

**2. 缺陷**
- 不支持对对象属性的动态添加或删除，需使用 `Vue.set()`。
- 数组的索引或长度变化需要通过特定方法触发更新。
- 依赖 Vue 2 的响应式机制，性能和灵活性有限。

---

### **二、Vue 3 的 `reactive` (替代 `observable`)**

在 Vue 3 中`observable` 不再被推荐使用，使用基于`proxy`的 **`reactive`** 取代。

**1. 使用方式**
```javascript
import { reactive } from "vue";

// 创建响应式对象
const state = reactive({
  count: 0,
});

// 修改数据时会触发响应
state.count++;
console.log(state.count); // 1

// 在组件中使用
export default {
  setup() {
    return { state };
  },
  template: `<button @click="state.count++">{{ state.count }}</button>`,
};
```

**2. 优点**
- **基于 Proxy**：
    - 动态监听对象属性的添加、删除和嵌套对象的变化。
    - 监听数组索引和长度的变化。
- **适用场景**：
    - 组件内部状态管理。
    - 跨组件的全局状态共享（与 `provide/inject` 或第三方库结合）。

* * *

## 20. v-show 与 v-if 的区别

在 Vue 2 和 Vue 3 中，`v-show` 和 `v-if` 是控制元素显示与隐藏的两个常用指令。两者在作用、性能以及使用场景上有显著的区别。这些特性在 Vue 2 和 Vue 3 中保持一致。以下是两者的具体对比：

---

### **1. `v-show` 和 `v-if` 的核心区别**

| **特性**       | **`v-show`**                             | **`v-if`**                             |
|--------------|------------------------------------------|---------------------------------------|
| **实现方式**     | 通过修改元素的 `CSS display` 样式值block/none来控制显示 | 通过动态添加和删除 DOM 元素来控制显示 |
| **性能**       | 初次渲染成本高（元素始终被渲染到 DOM）                    | 初次渲染成本低（仅在需要时创建 DOM）  |
| **条件切换开销**   | 切换时开销低（仅修改样式，不操作 DOM 树）                  | 切换时开销高（需要创建或销毁 DOM）   |
| **大量元素**     | 不适合：初次渲染所有元素耗时较多      | 适合：只渲染满足条件的少量元素     |
| **使用场景**     | 适合频繁显示/隐藏的场景                             | 适合条件变化不频繁，且需要销毁元素的场景 |
| **条件评估时机**   | 在初次渲染后不再移除元素，切换时评估条件                     | 每次条件切换都会重新创建/销毁 DOM    |
| **组件生命周期影响** | 不会触发组件的销毁和重建                   | 切换时会触发组件的销毁和重建        |

* * *

## 21. v-if和v-for哪个优先级更高？

元素同时使用两个指令,vue2版本中v-for优先于v-if执行.vue3中则是v-if先与v-for执行.

1. **vue2 使用该错误用法**

先执行v-for后执行v-if, 造成极大的`性能浪费`.

2. **vue3 使用该错误用法**

先执行v-if后执行v-for, 可能引发if中的`判断错误`因为此时v-for所定义的变量并未生成.

3. **处理方法：**

    +   使用计算属性或提前过滤数据

    +   将两个指令不要放在同一元素

4. **问题原因:** vue源码判断循序造成的，vue2 判断中el.for快于el.if,vue3正好相反.

```js
// vue 2x 代码示例
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

## 22. v-once/v-memo的使用场景有哪些？

| 特性             | **`v-once`**      | **`v-memo`**                              |
|------------------|-------------------|------------------------------------------|
| **功能**         | 一次性渲染，永不更新        | 按依赖项缓存，依赖变化时重新渲染          |
| **更新条件**     | 不会更新              | 依赖项变化时重新渲染                     |
| **适用场景**     | 完全静态的内容(版权/固定文本等) | 复杂逻辑、条件性更新                      |
| **性能优化**     | 初次渲染后跳过后续的依赖跟踪    | 根据依赖项避免不必要的渲染               |
| **支持的 Vue 版本** | Vue 2 和 Vue 3     | Vue 3                                    |
| **灵活性**       | 固定（不可更新）          | 灵活（依赖变化时更新）                   |


* * *

## 23. 你写过自定义指令吗？使用场景有哪些？

### **Vue 2 中的自定义指令**

1. **全局指令**
```javascript
// 注册全局指令
Vue.directive('focus', {
  // 钩子函数
  inserted(el) {
    el.focus();
  },
});
```

2. **局部指令**
```javascript
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus();
      },
    },
  },
};
```

3. **使用方式**
```vue
<template>
  <input v-focus />
</template>
```

---

### **Vue 3 中的自定义指令**

1. **全局指令**
```javascript
const app = Vue.createApp({});

app.directive('focus', {
  mounted(el) {
    el.focus();
  },
});
```

2. **局部指令**
```javascript
// 选项式
export default {
  directives: {
    focus: {
      mounted(el) {
        el.focus();
      },
    },
  },
};
```

```vue
// 组合式
<script setup>
// 完整自定义指令
const vFocus = {
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
```

3. **使用方式**
```vue
<template>
  <div v-focus:foo.bar="baz"> </div>
</template>
```

---

### **钩子函数对比**

| 钩子函数         | Vue 2                     | Vue 3                     | 说明                                                                 |
|------------------|---------------------------|---------------------------|----------------------------------------------------------------------|
| **初始化**       | `bind(el, binding)`       | `created(el, binding)`    | 在元素绑定指令时调用。                                               |
| **插入到 DOM**   | `inserted(el)`            | `mounted(el)`             | 在绑定元素插入父节点时调用。                                         |
| **更新时**       | `update(el, binding)`     | `updated(el, binding)`    | 在绑定元素更新时调用（不包括子节点）。                              |
| **后续更新完成** | `componentUpdated(el)`    | -                         | Vue 3 中已移除该钩子，推荐使用 `updated` 代替。                     |
| **解绑**         | `unbind(el)`              | `beforeUnmount(el)`       | 在指令与元素解绑时调用。                                             |

---


### **使用场景**

- **自动聚焦：**
```javascript
app.directive('focus', {
  mounted(el) {
    el.focus();
  },
});
```

- **懒加载图片：**
```javascript
app.directive('lazy-load', {
  mounted(el, binding) {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        el.src = binding.value;
        observer.unobserve(el);
      }
    });
    observer.observe(el);
  },
});
```

- **拖拽功能**
实现元素的拖拽功能：
```javascript
app.directive('draggable', {
  mounted(el) {
    el.style.position = 'absolute';
    el.onmousedown = (event) => {
      const shiftX = event.clientX - el.getBoundingClientRect().left;
      const shiftY = event.clientY - el.getBoundingClientRect().top;

      const moveAt = (pageX, pageY) => {
        el.style.left = `${pageX - shiftX}px`;
        el.style.top = `${pageY - shiftY}px`;
      };

      const onMouseMove = (event) => moveAt(event.pageX, event.pageY);

      document.addEventListener('mousemove', onMouseMove);

      el.onmouseup = () => {
        document.removeEventListener('mousemove', onMouseMove);
        el.onmouseup = null;
      };
    };
  },
});
```

- **权限控制**
根据用户权限动态控制元素的显示：
```javascript
app.directive('permission', {
  mounted(el, binding) {
    const userPermissions = getUserPermissions(); // 假设获取用户权限的方法
    if (!userPermissions.includes(binding.value)) {
      el.parentNode?.removeChild(el);
    }
  },
});
```
使用：
```html
<button v-permission="'admin'">Delete</button>
```

- **淡入淡出效果**
为元素添加进入/离开动画：
```javascript
app.directive('fade', {
  mounted(el) {
    el.style.transition = 'opacity 0.5s';
    el.style.opacity = '0';
    setTimeout(() => {
      el.style.opacity = '1';
    }, 0);
  },
});
```

* * *

## 24. 说下$attrs和$listeners的使用场景

`$attrs` 父子组件中`传递未显式声明的属性`.

`$listeners` 父子组件中传递`事件监听器`.

### Vue2 `$attrs`和`$listeners`
**`$attrs`**
- **Vue 2**: `$attrs` 是一个对象，包含**未被组件声明为 `props`** 的特性（包括动态绑定的属性）。
- **Vue 3**: 移除`$listeners`,`$attrs` 的功能不变，但同时包含了 `$listeners` 的功能。

**使用场景:**
**动态传递未声明的属性**
用于将父组件传递的非 `props` 属性绑定到子组件或其 DOM 上。
```vue
<!-- 父组件 -->
<template>
  <ChildComponent id="my-id" title="Hello" class="custom-class" />
</template>
```

```vue
<!-- 子组件 -->
<template>
  // 将`id` 和 `class` 绑定到元素
  <div v-bind="$attrs">I'm a child!</div>
</template>

<script>
export default {
  props: ['title'], // `title` 被声明为 `props`，`id` 和 `class` 会存放在 `$attrs` 中。 
};
</script>
```

---

**高阶组件 (HOC) 属性透传**
在封装组件时，需要透传父组件传入的所有非 `props` 属性到底层组件。

**示例：属性透传**
```vue
<template>
  <BaseInput v-bind="$attrs" />
</template>

<script>
export default {
  inheritAttrs: false, // 禁用自动绑定到根元素
};
</script>
```

---

**`$listeners`**

- `$listeners` 是一个对象，包含传递给组件但未显式绑定的**事件监听器**。

**使用场景:**
** 动态传递事件监听器**
用于将父组件传递的事件动态绑定到子组件。
```vue
<!-- 父组件 -->
<template>
  <ChildComponent @click="handleClick" @focus="handleFocus" />
</template>
```

```vue
<!-- 子组件 -->
<template>
  // 父组件的 `@click` 和 `@focus` 会被动态绑定到按钮上
  <button v-on="$listeners">Click me</button>
</template>
```
---

### **Vue 3 使用 `$attrs` `$listeners`**
在 Vue 3 中，**`$listeners` 被移除**，相关功能被整合进 `$attrs`。

- `$attrs` 包含：
    - 未声明的 `props`。
    - 事件监听器（如 `@click`、`@focus` 等）。

**Vue 3 示例：事件与属性同时透传**
```vue
<template>
  <button v-bind="$attrs">Click me</button>
</template>
```

* * *

## 25. watch/watchEffect/computed的区别及使用?

| 特性                    | `computed`                        | `watch`                | `watchEffect`         |
|-------------------------|-----------------------------------|------------------------|-----------------------|
| **缓存**                | 自动缓存计算结果                  | 不缓存                    | 不缓存                   |
| **依赖收集**            | 手动指定依赖                      | 手动指定依赖                 | 自动依赖收集                |
| **前后值对比**          | 不提供                            | 提供 `newVal` 和 `oldVal` | 不提供                   |
| **返回值**  | 返回计算后的新值，直接用作模板或逻辑。 | 不返回值，常用于执行副作用（如异步操作）。 | 不返回值，常用于执行副作用  |
| **异步支持**            | 不适合                            | 支持异步操作                 | 支持异步操作，但需手动管理回调中的异步逻辑 |
| **执行时机**            | 在模板或计算属性中访问时触发      | 数据变化时触发(批处理后)          | 数据变化时触发，初次执行时立即触发     |
| **适用场景**            | 计算派生数据                      | 执行副作用和复杂监听             | 简单副作用和自动依赖追踪          |
| **深度监听**            | 不支持                            | 支持（使用 `deep: true`）    | 不支持                   |

* * *

## 26. 说说nextTick的使用和原理？

`nextTick` 是 Vue 中用于延迟执行某个回调函数的 API，它会在下次 DOM 更新循环结束后执行，确保在执行时，DOM 已经被更新。

**使用场景：**
+   created中想要获取DOM时
+   响应式数据变化后获取DOM更新后的状态,如在更新数据后进行元素的滚动或焦点操作。
+   vue3可在组件外部使用时，不依赖于 Vue 实例

### **Vue 2 使用nextTick**

在 Vue 2 中，`nextTick` 是 Vue 实例的方法，可以在 Vue 实例的上下文中使用，也可以通过 `Vue.nextTick()` 来调用。

**使用示例**：
```javascript
// Vue 实例上下文中
this.$nextTick(() => {
  console.log('DOM 已更新');
});

// 全局使用
Vue.nextTick(() => {
  console.log('DOM 已更新');
});
```

**Vue 2 的实现原理**
1. Vue 内部对数据的修改触发了 DOM 更新。
2. `nextTick` 将回调函数加入一个队列，等待当前调用栈清空。
3. 浏览器在下一个宏任务中调用 `setTimeout` 或 `MutationObserver` 来触发 `nextTick` 的回调。


### **Vue 3 使用nextTick**

在 Vue 3 中，`nextTick` 是从 `vue` 包中导出的独立函数，而不是 Vue 实例的方法。

**使用示例**：
```javascript
import { nextTick } from 'vue';

nextTick(() => {
  console.log('DOM 已更新');
});
```

**Vue 3 的实现原理**
1. Vue 3 的响应式系统检测到数据变化并更新虚拟 DOM。
2. `nextTick` 将回调加入微任务队列（通过 `Promise`）或宏任务队列（通过 `MutationObserver` / `setTimeout`）。
3. 在当前任务结束后，`nextTick` 的回调被执行，确保 DOM 已更新。

* * *

## 27. 能说说key的作用吗？

**`key` 的作用:**

- **唯一标识,性能优化**：`key` 是用于标识虚拟 DOM 节点的唯一标识符,提升复用,降低比对算法开销。
- **避免不必要的重新渲染**：在动态渲染列表时，`key` 帮助 Vue 正确地匹配和更新元素，而不会将整个列表重渲染或进行错误的元素重新排列。
- **避免使用索引key**：会导致倒序插入页面更新错误问题等。

**使用场景**
`key` 主要用于 `v-for` 指令中，以确保每个子元素具有唯一标识符。

**规范使用:**

- **唯一性**：`key` 应该是列表中每个元素的唯一标识符，最好使用数据中的唯一 ID（如数据库中的主键）作为 `key`。
- **避免使用索引**：尽量避免使用数组的索引作为 `key`，因为在数据动态变化时（例如插入或删除元素），使用索引会导致 `key` 的重复或不稳定，可能引发渲染错误或性能问题。
- **稳定性**：`key` 应该是稳定的，不会随时间改变的属性。

* * *

## 28. 你是怎么处理vue项目中的错误的？

分为`逻辑错误`与`请求错误`,再根据错误类型进行`捕获上报`.

### **Vue 2 项目中的错误处理**

+ **全局错误处理**
使用 Vue 提供的 `Vue.config.errorHandler` 方法，可以捕获所有未被处理的错误。

```javascript
import Vue from 'vue';

Vue.config.errorHandler = (err, vm, info) => {
  console.error(`[Vue error]: ${err.message}`);
  console.error(`[Component name]: ${vm.$options.name || 'anonymous'}`);
  console.error(`[Error info]: ${info}`);
  // 可以上报错误到服务端
};
```

+ **组件级错误处理**
在单个组件中，可以使用 `errorCaptured` 钩子捕获子组件抛出的错误。

```javascript
export default {
  name: 'MyComponent',
  errorCaptured(err, vm, info) {
    console.error(`[Captured error]: ${err.message}`);
    // 返回 false 阻止错误向上传递
    return false;
  },
};
```

---

### **Vue 3 项目中的错误处理**

+ **全局错误处理**
在 Vue 3 中，可以通过 `app.config.errorHandler` 设置全局错误处理逻辑。

```javascript
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

app.config.errorHandler = (err, instance, info) => {
  console.error(`[Vue error]: ${err.message}`);
  console.error(`[Component name]: ${instance?.$options?.name || 'anonymous'}`);
  console.error(`[Error info]: ${info}`);
  // 错误上报
};

app.mount('#app');
```

+ **选项式组件级错误处理 errorCaptured**

```javascript
export default {
  name: 'MyComponent',
  errorCaptured(err, instance, info) {
    console.error(`[Captured error]: ${err.message}`);
    return false;
  },
};
```

+ **组合式组件级错误处理 onErrorCaptured**

```vue
<script setup> 
import { onErrorCaptured } from 'vue';

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

</script>
```

---

### **Promise 错误处理**
监听未捕获的 Promise 错误：

```javascript
window.addEventListener('unhandledrejection', (event) => {
  console.error(`[Unhandled promise rejection]: ${event.reason}`);
});
```

---

### **Axios全局网络请求错误处理**

```javascript
import axios from 'axios';

const instance = axios.create({
  baseURL: '/api',
  timeout: 10000,
});

// 请求拦截器
instance.interceptors.request.use(
  (config) => {
    // 添加全局请求逻辑
    return config;
  },
  (error) => {
    console.error(`[Request error]: ${error.message}`);
    return Promise.reject(error);
  }
);

// 响应拦截器
instance.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error(`[Response error]: ${error.response?.data?.message || error.message}`);
    // 可以在这里进行统一的错误提示或上报
    return Promise.reject(error);
  }
);

export default instance;
```

---

### **区别总结**

| 功能                | Vue 2                                                  | Vue 3                                   |
|---------------------|--------------------------------------------------------|-----------------------------------------|
| 全局错误处理         | `Vue.config.errorHandler`                              | `app.config.errorHandler`               |
| 组件级错误捕获       | `errorCaptured`                                        | 选项式`errorCaptured`,组合式`onErrorCaptured` |
| 未捕获 Promise 错误  | `window.addEventListener('unhandledrejection',()=>{})` | `window.addEventListener('unhandledrejection',()=>{})`               |


* * *

## 29. Vue要做权限管理该怎么做？控制到按钮级别的权限怎么做？

+ **页面权限**

    `前端方案:`**前端存储所有路由信息**，通过`路由守卫`要求用户登录，用户**登录后根据角色过滤出路由表**,最后通过`router.addRoutes(accessRoutes)`方式动态添加路由。

    `后端方案:`**把所有页面路由信息存在数据库**中，用户登录的时候根据其角色**查询得到其能访问的所有页面路由信息**返回给前端，前端**再通过`addRoutes`动态添加路由**信息
    
    `可维护性:`前端方案实现简单,后续修改需重新打包. 后端方案实现较为复杂,但是维护方便.

+ **按钮权限**

    按钮权限的控制通常会**实现一个指令**，例如`v-permission`，**将按钮要求角色通过值传给v-permission指令**，在指令的`moutned`钩子中可以**判断当前用户角色和按钮是否存在交集**，有则保留按钮，无则移除按钮。

* * *

## 30. SPA、SSR的区别是什么

| 特性                | **SPA（单页应用）**                 | **SSR（服务端渲染）**               |
|-------------------|-------------------------------|------------------------------|
| **定义**            | 通常由一个 HTML 页面组成，页面内容的更新通过 JavaScript 动态加载并渲染        | 服务端生成完整的 HTML 页面并直接发送到客户端浏览器 |
| **页面渲染**          | 前端渲染（浏览器执行 JavaScript）        | 服务端渲染（HTML 在服务器生成）           |
| **首屏加载速度**        | 较慢（需要加载 JavaScript）           | 较快（直接返回 HTML）                |
| **SEO 支持**        | 较差（需要额外的优化，如 prerender 或动态渲染） | 良好（服务端返回完整 HTML）             |
| **用户体验**          | 页面切换流畅，无需刷新                   | 页面切换可能需要重新加载                 |
| **开发复杂度**         | 较低                            | 较高（需要同构或特定框架支持）              |
| **服务器压力**         | 低（静态资源可缓存，渲染在客户端完成）           | 高（每次请求需服务端生成 HTML）           |
| **JavaScript 依赖** | 强（必须支持 JavaScript）            | 弱（基本内容可无 JS 支持，动态交互需 JS）     |
| **适用场景**          | SEO要求不高的页面,如后台管理              | SEO要求高,如电商网站、新闻网站、博客         |


**替代方案:**
+ 混合模式(SSR + SPA): 使用Nuxt.js 和 Next.js,首屏采用 SSR,其他页面使用SPA.
+ 预渲染: webpack打包时调用插件直接渲染静态内容,降低客户端开销.

* * *

## 31. 你了解哪些Vue性能优化方法？

+   路由懒加载/异步组件,利于代码分割

+   `keep-alive`缓存组件

+   条件渲染频繁的使用`v-show`

+   避免元素同时使用`v-for` / `v-if`

+   避免不必要的深度监听`watch`

+   使用v-once和v-memo忽略更新

+   长列表性能优化(虚拟列表/分片渲染)

+   组件销毁时,清除定时器/事件等.

+   图片懒加载

+   第三方插件按需引入

+   复杂组件分割为小组件

+   服务端渲染/静态网站生成

+   vue3 使用使用 `shallowRef 或 shallowReactive`，避免深层次数据监听

+   vue3 使用`Teleport`,将复杂 DOM 或性能敏感的元素移出当前 DOM 层级，减少影响

+   vue3 使用`Fragment`,减少无意义的 DOM 节点

+   vue3 使用`Suspense`, 延迟渲染组件，提升性能



* * *

## 32. 面试官：vue项目本地开发完成后部署到服务器后报404是什么原因呢？
**原因:** 单页面应用只有一个HTML,在页面切换时nginx就会去访问对应HTML,这些HTML不存在所以404.

当路由为HASH模式时即使不进行以下配置,依然能够正确访问,因为HASH改变页面并不会去访问新的HTML.

**SPA nginx正确配置:**
```text
server {
  listen  80;
  server_name  www.xxx.com;

  location / {
    /* 不论访问任何路由,都是使用以下路径的HTML作为返回 */
    index  /data/dist/index.html;
    try_files $uri $uri/ /index.html;
  }
}
```


## 33. vue-loader是什么？它有什么作用？

`vue-loader` 是 Webpack 的一个加载器 (Loader)，专门用于处理 `.vue` 文件,将 Vue 单文件组件 (Single File Components, SFC) 转换成标准的 JavaScript 模块，便于浏览器执行。

**工作流程:**

1. **解析 `.vue` 文件：**
    - 将文件分为 `<template>`、`<script>` 和 `<style>` 等块。

2. **分块处理：**
    - 每个块通过不同的 Loader 处理：
        - `<template>`：使用 `vue-template-compiler` 编译为渲染函数。
        - `<script>`：交由 `babel-loader` 或 `ts-loader` 编译。
        - `<style>`：使用 `css-loader`、`postcss-loader`、`sass-loader` 等处理。

3. **生成模块：**
    - 将处理后的模板、脚本、样式合并为一个 JavaScript 模块，供 Webpack 使用。


| 功能             | 作用                                               |
|----------------|--------------------------------------------------|
| **处理 `.vue` 文件** | 将 Vue 单文件组件拆分并转换为 JavaScript 模块。                 |
| **模板编译**      | 将`<template>`模板转为渲染函数，提高渲染性能。                              |
| **支持 CSS 预处理器** | 允许在 `<style>` 中使用 SCSS、LESS 等工具。                 |
| **样式作用域**      | 通过 `scoped` 属性确保组件样式隔离。                          |
| **自定义块扩展**     | 支持解析 `.vue` 文件中的自定义块(如 `<docs>`、`<i18n>`)，增强灵活性。 |
| **热模块替换 (HMR)** | 实现开发环境下的热更新，无需手动刷新页面。                            |

* * *

## 34. 从0到1自己构架一个vue项目，说说有哪些步骤、哪些重要插件、目录结构你会怎么组织

**项目构建:**

+ pnpm create-vue

+ 必要插件：vue-router,vuex/pinia,ui库,axios

+ 常用库: vueuse, nprogress,图标vite-svg-loader

+ 代码规范：使用prettier/eslint

+ 提交规范: 使用husky/lint-staged/commitlint

**推荐目录结构:**

```plaintext
├── public/               # 静态资源文件，直接输出到最终打包目录
│   ├── favicon.ico       # 网站图标
│   └── robots.txt        # 爬虫配置文件
├── src/                  # 源代码
│   ├── assets/           # 静态资源，经过 Webpack/Vite 构建处理
│   │   ├── images/       # 图片资源
│   │   ├── styles/       # 全局样式
│   │   └── fonts/        # 字体文件
│   ├── components/       # 公共组件（通用组件）
│   │   ├── base/         # 基础组件（如 Button、Input 等）
│   │   └── layout/       # 布局组件（如 Header、Footer 等）
│   ├── composables/      # 可复用的 Composition API 函数
│   │   └── useAuth.js    # 用户认证函数
│   ├── directives/       # 自定义指令
│   │   └── v-focus.js    # 示例：自动聚焦指令
│   ├── layouts/          # 页面布局文件（如多种页面结构的模板）
│   │   └── DefaultLayout.vue
│   ├── pages/            # 页面级组件
│   │   ├── Home.vue
│   │   └── NotFound.vue  # 404 页面
│   ├── router/           # 路由配置
│   │   └── index.js
│   ├── store/            # 状态管理（如 Vuex 或 Pinia）
│   │   ├── index.js      # 状态管理入口
│   │   └── auth.js       # 示例：用户状态模块
│   ├── utils/            # 工具函数
│   │   ├── helpers.js    # 示例：通用工具函数
│   │   └── validators.js # 示例：表单验证
│   ├── api/              # HTTP接口配置
│   ├── App.vue           # 根组件
│   ├── main.js           # 应用入口文件
│   └── env.d.ts          # 环境变量类型声明（如使用 TypeScript）
├── .env                  # 环境变量文件
├── .gitignore            # Git 忽略配置
├── package.json          # 项目依赖和脚本配置
├── vite.config.js        # Vite 配置文件
└── tsconfig.json         # TypeScript 配置文件（如使用 TS）
```

* * *

## 35. 实际工作中，你总结的vue最佳实践有哪些？

遵循vue官方文档推荐风格.

**编码风格方面：**
+   组件/属性命名避免与HTML元素冲突
+   使用v-for时务必加上key，且不要跟v-if写在一起

**性能方面：**
+   路由懒加载减少应用尺寸
+   利用SSR减少首屏加载时间
+   利用v-once渲染那些不需要更新的内容
+   一些长列表可以利用虚拟滚动技术避免内存过度占用
+   对于深层嵌套对象的大数组可以使用shallowRef或shallowReactive降低开销
+   避免不必要的组件抽象

**安全：**
+   不使用不可信模板，例如使用用户输入拼接模板：`template: <div> + userProvidedString + </div>`
+   谨慎使用v-html，:url，:style等，避免html、url、样式等注入


# Vue3 API & 面试题

## API参考: [vue 3.x](vue3.md)

## 36. Composition API 与 Options API 有什么不同

| **特性**     | **选项式 API**                                     | **组合式 API**                      |
|------------|------------------------------------------------|----------------------------------|
| **代码组织方式** | 按功能类型分组，例如 `data`、`methods` 等               | 按逻辑分组，将相关逻辑集中在一起。                |
| **逻辑复用**   | 通过 `mixins` 或 `extends` 实现，但可能会导致命名冲突或难以追踪 | 使用自定义复用逻辑，清晰且易于管理。  |
| **类型支持**   | 较弱，对 TypeScript 支持有限                          | 原生支持 TypeScript，类型推断更强大。         |
| **学习曲线**   | 易于上手，适合初学者                                  | 需要学习 Composition API，适合复杂项目和高级开发者。 |
| **代码可读性**  | 简单场景下代码易读，但复杂逻辑会分散在多个选项中               | 复杂逻辑集中，代码更清晰，但初学者可能不习惯。          |
| **性能**     | 两者性能无明显差异，底层运行机制相同                     | 性能与选项式 API 相同，主要提供更好的组织逻辑能力。     |
| **适用场景**   |          小型项目或简单组件            | 中大型项目                            |

**注意:** `Composition API`能和`Options API`可以一起使用, 但是作用域并不共享,不推荐使用.

* * *

## 37. ref和reactive异同

| **特性**      | **`ref`**                                    | **`reactive`**         |
|-------------|----------------------------------------------|------------------------|
| **模板解包**    | 自动解包,无需使用 `.value`                           | 自动解包                   |
| **响应性**     | 可创建响应式数据                                     | 可创建响应式数据               |
| **数据类型支持**  | 适合基本类型（如数字、字符串、布尔值），也可用于对象或数组（需通过 `.value`）。 | 适合对象和数组，直接操作内部属性。      |
| **深度响应式**   | 对于对象和数组，只是引用响应式，需手动深层处理。                     | 默认对深层嵌套对象具有响应式支持。      |
| **实现机制**    | 内部是一个包含 `value` 属性的已被getter/setter的对象。       | 使用 `Proxy` 实现对整个对象的代理。 |
| **访问方式**    | 修改或读取值时需要使用 `.value`。                        | 直接访问或修改属性即可。           |
| **解构后的响应性** | 解构后响应性失效，需显式解包。                              | 解构后依然保持响应性。            |
| **适用场景**    | 管理单个变量或基本数据类型。                               | 管理复杂对象或数组。             |
| **逻辑复用**    | 使用时需多次解包 `.value`，稍显繁琐。                      | 返回完整的响应式对象，操作更加方便。     |


* * *

## 38. 你知道哪些vue3新特性及优化

**新特性:**

1. **Composition API**
    - 更灵活的逻辑组织方式，支持逻辑复用。
    - 提供 `ref`、`reactive`、`computed`、`watch` 等响应式工具。

2. **新的响应式系统**
    - 基于 `Proxy`，支持深层响应式，动态添加属性无需额外处理。

3. **Teleport**
    - 内容可以渲染到 DOM 的任意位置，适用于模态框、通知等场景。

4. **Fragments**
    - 支持组件返回多个根节点，减少不必要的 DOM 层级。

5. **Suspense**
    - 异步组件渲染时提供加载占位内容的能力。

6. **新的全局 API**
    - 将全局配置迁移到 `app` 实例，增强模块化。

7. **Composition API Hooks**
    - 提供 `onMounted`、`onUpdated` 等更灵活的生命周期管理。

8. **Emits 和 Props 验证**
    - 显式定义事件和灵活的 `props` 验证，增强类型安全。

9. **增强单文件组件（SFC）**
    - `<script setup>` 提供简洁语法。
    - `<style scoped>` 支持 CSS Variables。


**优化:**

1. **性能提升**
    - 编译器优化：静态提升、静态节点合并。
    - 虚拟 DOM 改进：更高效的依赖追踪和更新。

2. **Tree-shaking 支持**
    - 按需加载功能，减少打包体积。

3. **更小的运行时体积**
    - 核心库经过优化，默认打包体积更小。

4. **TypeScript 支持**
    - 内置支持更好的类型推导和开发体验。

5. **API易用/扩展性提升**
    - `v-model`包含了`sync`修饰符的作用
    - 独立的响应式创建(`reactivity`/`ref`)
    - 提供自定义渲染器(`custom renderer`)

* * *

## 39. Vue3.0 性能提升体现在哪些方面？

| **优化点**                     | **提升效果**                               |
|-------------------------------|----------------------------------------|
| **Proxy 响应式**                | 动态属性、深层对象响应式性能提升，减少了初始化递归的开销。          |
| **编译器优化**                  | 静态提升、静态节点合并和 Patch Flag 减少了渲染时的diff计算。 |
| **虚拟 DOM 改进**               | 更精准的依赖追踪和更高效的 Diff 算法提高了更新性能。          |
| **Tree-shaking 支持**           | 按需引入减少了打包体积，优化了加载性能。                   |
| **运行时体积减少**               | 核心库更小，减轻了页面加载压力。                       |
| **打包体积优化**            | 核心运行时更小，默认打包体积约 13KB gzip。             |
| **Fragments**                  | 减少 DOM 层级，降低渲染复杂度。                     |
| **异步加载优化（Suspense）**      | 更优雅的异步处理和首屏加载性能提升。                     |
| **SSR 优化**              | 提升服务端渲染性能，支持流式渲染，降低白屏时间。               |
| **依赖追踪机制优化**         | 精确依赖跟踪，减少冗余的响应式触发。                     |



## 40. Vue3.0里为什么要用 Proxy 替代 defineProperty ？

**1.常用数据劫持方式区别:**

| **特性**       | **Object.defineProperty**                     | **Proxy**                             | **Getter/Setter**                   |
|--------------|----------------------------------------------|---------------------------------------|-------------------------------------|
| **拦截范围**     | 仅能拦截对象已存在的属性                      | 整个对象，包括新增/删除属性            | 特定属性，需显式定义                  |
| **动态属性监听**   | 不支持                                      | 支持                                 | 不支持                              |
| **深层嵌套对象支持** | 需递归实现                                  | 内置支持                             | 不支持                              |
| **兼容性**      | 支持 IE9 及以上                              | 不支持 IE，需现代浏览器支持             | 支持 IE9 及以上                      |
| **性能**       | 较优，适合较少操作的场景                      | 略低于 `defineProperty`，适合复杂场景 | 性能最佳，适合简单场景                |
| **应用场景**     | Vue 2 响应式系统，单属性定制化拦截            | Vue 3 响应式系统，全面代理整个对象      | 简单属性控制（如计算属性、缓存属性） |

---
**2.使用proxy的优势:**
+ 无需递归实现响应式,降低性能开销.
+ 能够动态监听属性,属性新增/删除都能自动监听. (defineProperty不会自动监听,vue2中需要手动调用 vue.set()/vue.delete())

* * *

# Vue Router

## API参考: [vue router](vue_router.md)

## 41. History模式和Hash模式有何区别？

| **模式**      | **API**                    | **URL 格式**         | **刷新页面支持** | **SEO 支持**  | **适用场景**                  |
|-------------|----------------------------|----------------------|------------|---------------|------------------------------|
| **History** | **`createWebHistory`**     | `/about`            | 需web服务器支持  | 支持           | SSR、SEO 友好的 Web 应用       |
| **Hash**    | **`createWebHashHistory`** | `/#/about`         | 无需web服务器支持 | 不支持         | 纯前端项目，静态文件托管         |
| **SSR**     | **`createMemoryHistory`**  | 不显示 URL         | 不适用        | 不适用         | SSR、测试环境                  |

* * *

## 42. 怎么定义动态路由？怎么获取传过来的动态参数？

**动态路由篇配置:**
```text
// :id 即为动态路由参数
{ path: '/users/:id', component: User }
```

**获取:**
```vue
// 组合式
<script setup>
  import { useRouter } from 'vue-router';
  const router = useRouter(); // 获取路由实例
  console.log(router.currentRoute.params) // 输出:id值
</script>

```

```vue
// 选项式
<script>
  export default {
      mounted() {
          console.log(this.$route.params) // 输出:id值
      }
  }
</script>

```

**注意:**

+ 通过watch router/route的变化,可以动态获取路由参数

+ 404页面通过`放置于路由列表最后`进行正则匹配

* * *

## 43. 如果让你从零开始写一个vue路由，说说你的思路

+ 模块化路由配置：按功能模块划分路由，保持清晰和易维护。
+ 路由懒加载：优化性能，只加载用户访问的模块。
+ 命名路由和路径别名：便于管理和跳转。
+ 嵌套路由：适合复杂页面结构。
+ 404 处理：确保未匹配路由有友好的提示页面。
+ 路由守卫：添加全局、中间件或单个路由级别的守卫，增强安全性和功能性。

* * *

## 44. 怎么实现路由懒加载呢？

**优点:** 利于代码分割,仅加载用户浏览的模块,降低开销.

```javascript
const router = createRouter({
  routes: [{ path: '/users/:id', component: () => import(/* webpackChunkName: "这个分割块的名字" */'./views/UserDetails') }],
  // ...
})
```

* * *

## 45. router-link和router-view是如何起作用的？

### **<router-link>常用属性:**

| 属性/事件         | 说明                                      |
|-------------------|-----------------------------------------|
| `to`             | 跳转目标，可以是路径字符串或路由对象       |
| `replace`        | 替换当前历史记录，不会增加新记录           |
| `custom`         | 是否自定义内容（用于完全自定义链接样式）    |
| `active-class`   | 激活时的 CSS 类，默认 `router-link-active` |
| `exact-active-class` | 精确匹配激活时的 CSS 类，默认 `router-link-exact-active` |
| `v-slot` (custom) | 自定义内容时使用插槽                     |

**示例:**
```vue
<template>
  <div>
    <!-- 基本用法 -->
    <router-link to="/home">Go to Home</router-link>
    
    <!-- 路由对象用法 -->
    <router-link :to="{ name: 'User', params: { id: 1 }}">User 1</router-link>
    
    <!-- 替换当前历史记录 -->
    <router-link to="/about" replace>About (Replace)</router-link>
    
    <!-- 自定义样式 -->
    <router-link to="/profile" active-class="active-link">Profile</router-link>
  </div>
</template>
```


### **<router-view>常用属性:**

| 属性            | 说明                                         |
|------------------|--------------------------------------------|
| `name`          | 命名视图的名称（默认显示默认路由视图）        |
| `v-slot`        | 用于自定义嵌套路由或视图内容                 |

**示例:**
```vue
<template>
  <div>
    <!-- 默认视图 -->
    <router-view></router-view>

    <!-- 命名视图 -->
    <router-view name="header"></router-view>
    <router-view name="footer"></router-view>
  </div>
</template>
```
---

### **作用与原理:**

| **组件**      | **作用**                                                                                 | **实现原理**                                                   | **常用场景**                     |
|---------------|-----------------------------------------------------------------------------------------|------------------------------------------------------------|-----------------------------------|
| **`<router-link>`** | 创建导航链接，支持动态样式与行为，触发路由跳转                                           | 渲染为 `<a>`，绑定 `href` 属性，调用 `router.push` 或 `router.replace` | 导航菜单、面包屑、跳转按钮        |
| **`<router-view>`** | 显示与当前路径匹配的组件，支持嵌套路由和命名视图                                          | 根据Vue Router 的 `matched` 路由记录,渲染对应路由组件，支持递归渲染子路由           | 页面主体内容展示，嵌套路由        |


* * *

## 46. Vue-router 除了 router-link 怎么实现跳转

vue-router导航有两种方式：`声明式导航`和`编程方式导航`

**声明式导航示例:**

```text
<router-link to="/about">Go to About</router-link>
```

**编程导航示例:**

```vue
// 选项式
<script>
  export default {
      mounted(){
        // literal string path
        this.$router.push('/users/eduardo')

        // object with path
        this.$router.push({ path: '/users/eduardo' })

        // named route with params to let the router build the url
        this.$router.push({ name: 'user', params: { username: 'eduardo' } }) 
      }
  }
</script>
```

```vue
// 组合式
<script setup>
  import { useRouter } from 'vue-router'
  let router = useRouter();
  // literal string path
  router.push('/users/eduardo')
</script>
```


## 47. 在什么场景下会用到嵌套路由？

适用公用的页面布局,如顶部导航栏/左侧菜单栏/主内容区等,部分内容跟随路由切换,而公用部分不变.

**路由配置示例:**

```js
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
    ],
  },
]
```

* * *

## 48. vue-router中如何保护路由？

参考:[路由守卫](vue_router.md#导航守卫)

* * *

# Pinia & Vuex

## vue3推荐状态管理 Pinia API参考: [pinia](pinia.md)

## Vuex API参考: [vuex](vuex.md)

## 49. pinia的作用与优缺点?

Vue3 官方推荐的状态管理库，用来替代 Vuex，提供更轻量、更灵活的状态管理方案。

**作用:**

1. **集中式状态管理**
    - Pinia 和 Vuex 类似,但代码量更少，提供一个中心化的存储，用于跨组件共享和管理状态。

2. **模块化设计**
    - 不再使用vuex命名空间模块,每个状态模块被定义为一个独立的 Store，对应不同的功能模块，方便组织和维护。

3. **动态模块注册**
   - 支持动态创建和注册 store，适用于按需加载模块的场景。

4. **响应式数据**
    - Pinia 的状态是基于 Vue 的响应式系统实现的，数据变化时，所有依赖这些数据的组件会自动更新。

5. **简化开发**
    - Pinia 通过直观的 API 提供更简洁的状态定义和访问方式，如直接使用 `state` 和 `actions`，无需像 Vuex 那样强制使用 mutation。
    - 支持函数式定义 store，方便开发者灵活地组织代码。

6. **Composition API 集成**
   - Pinia 与 Vue 3 的 Composition API 无缝结合，使得状态管理与组件逻辑可以更自然地整合。

7. **支持服务端渲染**
    - Pinia 完全支持 Vue 3 的服务端渲染 (SSR)，并且在这方面的实现更加简单。

8. **内置 TypeScript 支持**
   - Pinia 的设计充分考虑了 TypeScript 支持，所有 API 都能通过类型推断实现高度类型安全。


---

**Pinia 的缺点:**

1. **与 Vue 2 的兼容性**
    - Pinia 是为 Vue 3 设计的状态管理工具，虽然可以通过插件支持 Vue 2，但功能和体验不如 Vue 3 原生环境。

2. **生态系统相对较小**
    - 相较于 Vuex，Pinia 的社区生态和第三方插件支持还在发展中，部分功能可能需要自行实现。

3. **学习曲线（对 Vuex 用户）**
    - 对已经熟悉 Vuex 的开发者来说，迁移到 Pinia 需要适应新的 API 和开发方式。

4. **潜在依赖问题**
    - 由于 Pinia 完全依赖 Vue 3 的响应式系统，可能在某些复杂场景中出现意料之外的状态更新问题。

---

**Pinia 与 Vuex 的区别**

| 特性               | Vuex                         | Pinia                        |
|--------------------|------------------------------|-----------------------------|
| 定义复杂度         | 较高（state, getters, mutations, actions 分离） | 较低（actions 修改状态）      |
| 类型支持           | 需要手动配置类型             | 内置 TypeScript 支持         |
| 模块化支持         | 通过命名空间实现             | 原生模块化设计               |
| 开发体验           | 偏繁琐                      | 简洁直观                    |
| 响应式系统         | 基于 Vue.observable          | 基于 Vue 3 的响应式系统      |

---




## 50. 简单说一说你对vuex理解？

Vuex 是 Vue2 官方提供的状态管理库，主要用于集中管理应用的状态（数据）和跨组件的状态共享。

---

**Vuex 的作用:**

1. **集中式状态管理**
    - Vuex 提供一个全局的 `store` 对象，用于存储应用中的共享状态，解决了组件之间的数据共享和传递问题。

2. **响应式数据**
    - Vuex 的状态是响应式的，当状态更新时，所有引用该状态的组件会自动重新渲染。

3. **组件通信简化**
    - 在复杂应用中，父子组件之间可以通过 props 和 events 通信，但对于跨层级或兄弟组件，通信会变得复杂。Vuex 提供了统一的状态管理，组件之间可以直接访问或修改全局状态，而无需层层传递。

4. **统一的状态更新逻辑**
    - Vuex 强制通过 **mutation** 来更新状态，并且可以在开发环境中追踪每次状态的变化，便于调试和维护。
    - Vuex 的状态变更是透明的，所有状态更新通过 mutation 记录下来，可用 Vue DevTools 进行调试和时间旅行（time-travel debugging）。

5. **模块化管理**
    - Vuex 支持将状态、getters、mutations、actions 划分为多个模块，适合大型项目的组织和扩展。

6. **适合中大型应用**
    - 当应用规模增大，组件通信和状态管理变得复杂时，Vuex 提供了模块化的管理方案，使代码结构更清晰，逻辑更易维护。

---

**Vuex 的缺点:**

1. **学习成本**
    - 对于小型项目或新手来说，引入 Vuex 可能会显得复杂。需要理解 actions、mutations、getters 的概念及其配合使用的方式。

2. **增加代码量**
    - 使用 Vuex 会增加额外的模板代码，例如定义 state、mutation、action 等，这可能会让小型项目的开发变得繁琐。
    - 获取对应模块的数据流程繁琐。

    ```js
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

3. **灵活性有限**
    - Vuex 强制使用 mutation 来同步更新状态，虽然有助于调试和控制，但在某些情况下会显得限制较多。例如，直接修改状态可能更简单快速。

4. **性能问题**
    - 当状态规模增大，且组件大量依赖全局状态时，可能会导致性能问题，因为状态的变化会触发相关组件的重新渲染。

5. **支持有限**
    - 在 Vue 3 中，**Composition API** 提供了一种更轻量的状态管理方式；此外，社区中也有更灵活的状态管理库（如 Pinia）作为 Vuex 的替代方案。
    - 对typescript支持有限


* * *

## 51. pinia与vuex中 module作用与区别？

**作用:** 在大型应用中，状态管理会随着功能的增多变得复杂,分割模块可提高可维护性.

---

### **Pinia 的模块化**

**Pinia 模块化的特点:**

- Pinia 的模块化是通过**独立 Store 定义**实现的，每个模块对应一个 Store，互相独立。
- Store 是基于函数定义的，灵活且简单。
- 模块化直接依赖文件系统结构，无需额外配置命名空间。

**使用示例:**

```javascript
// 创建store
// stores/counter.js
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0,
  }),
  actions: {
    increment() {
      this.count++;
    },
  },
});

// stores/user.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
    state: () => ({
        name: 'John Doe',
        isLoggedIn: false,
    }),
    actions: {
        login() {
            this.isLoggedIn = true;
        },
        logout() {
            this.isLoggedIn = false;
        },
    },
});
```

```vue
<!--在组件中使用多个 Store-->
<template>
  <div>
    <h1>Counter: {{ counter.count }}</h1>
    <button @click="counter.increment">Increase Counter</button>

    <h1>User: {{ user.name }}</h1>
    <button @click="user.login">Login</button>
    <button @click="user.logout">Logout</button>
  </div>
</template>

<script>
import { useCounterStore } from '@/stores/counter';
import { useUserStore } from '@/stores/user';

export default {
  setup() {
    const counter = useCounterStore();
    const user = useUserStore();

    return { counter, user };
  },
};
</script>
```

```javascript
// 动态注册 Store, 适用于按需加载
const userStore = useUserStore();
if (!userStore.$state) {
  userStore.$state = { name: '', isLoggedIn: false }; // 初始化状态
}
```

---

### **Vuex 的模块化**

**Vuex 模块化的特点:**
- Vuex 模块化通过 `modules` 属性实现。
- 模块可以嵌套、定义命名空间（namespaced）。
- Vuex 的模块化设计适合传统的命名空间式状态管理。

**使用示例:**

```javascript
// vuex模块定义
// store/modules/counter.js
export const counterModule = {
  namespaced: true,
  state: () => ({
    count: 0,
  }),
  mutations: {
    increment(state) {
      state.count++;
    },
  },
  actions: {
    asyncIncrement({ commit }) {
      setTimeout(() => {
        commit('increment');
      }, 1000);
    },
  },
  getters: {
    doubleCount(state) {
      return state.count * 2;
    },
  },
};

// store/modules/user.js
export const userModule = {
  namespaced: true,
  state: () => ({
    name: 'John Doe',
    isLoggedIn: false,
  }),
  mutations: {
    login(state) {
      state.isLoggedIn = true;
    },
    logout(state) {
      state.isLoggedIn = false;
    },
  },
};
```

```javascript
// 注册模块
// store/index.js
import { createStore } from 'vuex';
import { counterModule } from './modules/counter';
import { userModule } from './modules/user';

export const store = createStore({
  modules: {
    counter: counterModule,
    user: userModule,
  },
});
```

```vue
// 组件中使用
<template>
  <div>
    <h1>Counter: {{ count }}</h1>
    <button @click="increment">Increase Counter</button>

    <h1>User: {{ name }}</h1>
    <button @click="login">Login</button>
    <button @click="logout">Logout</button>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex';

export default {
  computed: {
    ...mapState('counter', ['count']),
    ...mapState('user', ['name']),
  },
  methods: {
    ...mapMutations('counter', ['increment']),
    ...mapMutations('user', ['login', 'logout']),
  },
};
</script>
```

---

### **Pinia 与 Vuex 模块化的对比**

| 特性      | Pinia                              | Vuex                     |
|---------|------------------------------------|--------------------------|
| 模块定义方式  | 独立文件，类似模块化 JavaScript    | 嵌套在 `modules` 中，需定义命名空间  |
| API 简洁性 | 简洁，基于函数的方式              | 较复杂，需定义多种属性              |
| 命名空间支持  | 无需手动设置命名空间，文件隔离即模块化 | 通过 `namespaced: true` 实现 |
| TS支持    | 内置强类型支持                    | 需手动定义类型                  |
| 动态加载    | 支持                              | 需手动实现                    |
| vue3支持  | 支持                              | 有限支持                     |


* * *

## 52. 如果让你从零开始写一个vuex，说说你的思路

**设计核心机制:**
    - 状态存储 (`state`)：管理全局共享数据。
    - 状态修改 (`mutation`)：定义更新状态的逻辑，确保状态变更是可追踪的。
    - 异步操作 (`action`)：处理异步任务并最终通过 mutation 更新状态。
    - 派发 (`dispatch`) 和 提交 (`commit`)：触发 action 和 mutation 的方法。
    - 订阅和响应式更新：使用 Vue 的响应式系统，确保状态变化时组件能够重新渲染。


**Vuex 核心实现:**

```javascript
// 简单的 Vuex 实现
class Store {
  constructor(options) {
    // 保存状态
    this.state = Vue.observable(options.state); // 响应式状态

    // 保存 mutations
    this._mutations = options.mutations;

    // 保存 actions
    this._actions = options.actions;

    // 绑定上下文
    this.commit = this.commit.bind(this);
    this.dispatch = this.dispatch.bind(this);

    // 处理 getters
    this.getters = {};
    const getters = options.getters || {};
    Object.keys(getters).forEach((key) => {
      Object.defineProperty(this.getters, key, {
          get: () => getters[key](this.state),
      });
    });

    // 模块化合并
    this.state = Vue.observable(this._mergeModules(options.modules || {}, options.state));
    this._mutations = this._mergeModules(options.modules || {}, options.mutations);
    this._actions = this._mergeModules(options.modules || {}, options.actions);
  }

  // commit 方法：同步修改 state
  commit(type, payload) {
    const mutation = this._mutations[type];
    if (!mutation) {
      console.error(`[Vuex] Mutation not found: ${type}`);
      return;
    }
    mutation(this.state, payload);
  }

  // dispatch 方法：处理异步逻辑
  dispatch(type, payload) {
    const action = this._actions[type];
    if (!action) {
      console.error(`[Vuex] Action not found: ${type}`);
      return;
    }
    return action(this, payload);
  }

  // 模块化
    _mergeModules(modules, root) {
        const merged = { ...root };
        Object.keys(modules).forEach((key) => {
            const module = modules[key];
            Object.assign(merged, module);
        });
        return merged;
    }
}

// 简单的 Vuex 安装函数,实现响应式
function install(Vue) {
  Vue.mixin({
    beforeCreate() {
      if (this.$options.store) {
        Vue.prototype.$store = this.$options.store;
      }
    },
  });
}

// 导出 Vuex 模拟
export default { Store, install };
```

---

```javascript
import Vue from 'vue';
import Vuex from './simple-vuex'; // 使用我们自定义的简单 Vuex

Vue.use(Vuex);

// 创建 Store 实例
const store = new Vuex.Store({
  state: {
    count: 0,
  },
  mutations: {
    increment(state) {
      state.count++;
    },
    decrement(state) {
      state.count--;
    },
  },
  actions: {
    asyncIncrement({ commit }) {
      setTimeout(() => {
        commit('increment');
      }, 1000);
    },
  },
  getters: {
      doubleCount(state) {
          return state.count * 2;
      },
  },
});

export default store;
```

```vue
<template>
  <div>
    <p>Count: {{ $store.state.count }}</p>
    <button @click="$store.commit('increment')">Increment</button>
    <button @click="$store.commit('decrement')">Decrement</button>
    <button @click="$store.dispatch('asyncIncrement')">Async Increment</button>
  </div>
</template>

<script>
export default {
  name: 'Counter',
};
</script>
```

---

## 53. 实现pinia的思路?

[Pinia较为完整的实现及使用](create_pinia.md)

**Pinia 核心设计思想:**
1. **响应式状态 (`state`)**  
   使用 Vue 3 的 `reactive` 或 `ref` 实现响应式数据管理。

2. **派发 (`action`)**  
   定义方法（函数）来封装业务逻辑，可以同步或异步修改状态。

3. **计算属性 (`getter`)**  
   使用 Vue 的 `computed` 提供派生状态，简化组件的状态访问逻辑。

4. **模块化 Store**  
   每个模块是一个独立的 Store，可以单独注册和使用。

5. **动态注册 **
    能够动态注册store.

6. **类型安全（可选）**  
   设计时考虑 TypeScript 支持，为开发者提供类型推断。

7. **插件支持**
   扩展 Store 功能的灵活机制。

---

**简单实现:**

```javascript
import { reactive, computed } from 'vue';

class Store {
  constructor(options) {
    // 创建响应式状态
    this.$state = reactive(options.state ? options.state() : {});

    // 创建 getters
    this.getters = {};
    if (options.getters) {
      for (const [key, getterFn] of Object.entries(options.getters)) {
        this.getters[key] = computed(() => getterFn(this.$state));
      }
    }

    // 创建 actions
    this.actions = {};
    if (options.actions) {
      for (const [key, actionFn] of Object.entries(options.actions)) {
        this.actions[key] = actionFn.bind(this); // 绑定上下文
      }
    }
  }

  // 提供对外访问接口
  install(app) {
    app.provide('store', this);
  }
}

export function createStore(options) {
  return new Store(options);
}
```

* * *

## 54. vuex中actions和mutations有什么区别？

| 特性         | Mutations                | Actions                  |
|------------|--------------------------|--------------------------|
| **职责**     | 直接修改状态                   | 执行业务逻辑，分发 `mutations`    |
| **操作类型**   | 必须是同步操作                  | 可以包含异步操作                 |
| **调用方式**   | `commit('mutationName')` | `dispatch('actionName')` |
| **调试支持**   | 可被 Vue DevTools 追踪       | 无法直接追踪状态修改               |
| **是否修改状态** | 直接修改状态                   | 不直接修改状态，通过 `mutation`    |
| **适用场景**   | 简单的同步操作                  | 复杂的业务逻辑或异步操作             |


* * *

## 55.怎么监听vuex / pinia 数据的变化？

### 监听vuex数据变化

1. 使用 Vue 的 `watch` 监听组件中的计算属性:

```javascript
import { mapState } from 'vuex';

export default {
  computed: {
    ...mapState({
      someState: state => state.someState, // 将 Vuex 的某个 state 映射到计算属性
    }),
  },
  watch: {
    someState(newValue, oldValue) {
      console.log('someState 发生变化:', oldValue, '=>', newValue);
    },
  },
};
```

2. 使用 `store.subscribe` 方法:

```javascript
import store from './store'; // 导入你的 Vuex store

store.subscribe((mutation, state) => {
  console.log('mutation 发生:', mutation.type, 'payload:', mutation.payload);
  console.log('新的状态:', state);
});
```

3. 使用 `store.watch` 方法:

```javascript
import store from './store'; // 导入你的 Vuex store

const unwatch = store.watch(
  (state) => state.someState, // 要监听的状态
  (newValue, oldValue) => {
    console.log('someState 变化:', oldValue, '=>', newValue);
  }
);

// 停止监听
// unwatch();
```

### 监听pinia的数据变化

1. 使用 Vue 的 `watch` 函数:

```javascript
import { watch } from 'vue';
import { useStore } from './stores/store'; // 导入你的 Pinia store

const store = useStore();

watch(
  () => store.someState, // 监听 store 中的某个属性
  (newValue, oldValue) => {
    console.log('someState 发生变化:', oldValue, '=>', newValue);
  }
);
```

2. 使用 `store.$subscribe` 方法:

```javascript
import { useStore } from './stores/store'; // 导入你的 Pinia store

const store = useStore();

// 监听整个 store 的变化
store.$subscribe((mutation, state) => {
  console.log('store 状态变化:', mutation, state);
});
```

3. 使用 `computed` 进行响应式绑定:

```javascript
import { computed } from 'vue';
import { useStore } from './stores/store'; // 导入你的 Pinia store

const store = useStore();

const someState = computed(() => store.someState);

// 在模板或逻辑中使用
console.log('someState 的值:', someState.value);
```

## 56. 页面刷新后pinia/vuex的数据丢失怎么解决？
可以通过将数据保存在浏览器的本地存储（如 `localStorage` 或 `sessionStorage`）中来实现。

### **Vuex 数据持久化**

1. 使用 `localStorage` 或 `sessionStorage`:

```javascript
// store.js
const store = new Vuex.Store({
  state: {
    // 在 Vuex 的 `state` 中初始化时，从 `localStorage` 获取数据。
    someState: JSON.parse(localStorage.getItem('someState')) || 'defaultValue',
  },
  mutations: {
    setSomeState(state, payload) {
      state.someState = payload;
      localStorage.setItem('someState', JSON.stringify(state.someState)); // 在 `mutations` 中使用 `localStorage` 保存数据。
    },
  },
});

export default store;
```

2. 使用`插件`vuex-persist、vuex-persistedstate, 或使用vuex提供的`subscribe统一的处理`.



### **Pinia 数据持久化**

1.使用 `localStorage` 或 `sessionStorage`

```javascript
import { defineStore } from 'pinia';

export const useStore = defineStore('store', {
  state: () => ({
    someState: JSON.parse(localStorage.getItem('someState')) || 'defaultValue',
  }),
  actions: {
    setSomeState(payload) {
      this.someState = payload;
      localStorage.setItem('someState', JSON.stringify(this.someState));
    },
  },
});
```

2. 使用插件 `pinia-plugin-persistedstate`,或使用pinia提供的`$subscribe统一处理`


