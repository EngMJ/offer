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

## 47. 在什么场景下会用到嵌套路由？

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

## 48. vue-router中如何保护路由？

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




# Pinia & Vuex

## vue3推荐状态管理 Pinia API参考: [pinia](pinia.md)

## Vuex API参考: [vuex](vuex.md)


## 49. 简单说一说你对vuex理解？

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

## 50. 你有使用过vuex的module吗？

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

## 51. 如果让你从零开始写一个vuex，说说你的思路

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

## 52. vuex中actions和mutations有什么区别？

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

## 53. 使用vue渲染大量数据时应该怎么优化？说下你的思路！

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


## 54.怎么监听vuex数据的变化？

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

## 55. 页面刷新后vuex的state数据丢失怎么解决？

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

## 56. 你觉得vuex有什么缺点？

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

