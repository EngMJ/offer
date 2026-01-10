# Vue 面试题

## 1. MVVM架构 & 渐进式框架
Vue 的 MVVM 架构代表 Model-View-ViewModel，它是一种分离关注点的设计模式，当` Model（数据层）`修改时会通知` ViewModel（视图模型层)`修改 `View（视图层）`,反之亦然。

![mvvm](https://upload.wikimedia.org/wikipedia/commons/8/87/MVVMPattern.png)

Vue 的“渐进式框架”设计理念：可以根据项目的实际需要，逐步引入 Vue 的功能，而不必一次性改变整个项目架构,使用vue生态的全部功能。

## 2. Vue实例挂载的过程中发生了什么?

### Vue 3 挂载过程

1. 创建 Vue 应用实例,并挂载DOM元素
   ```javascript
   const app = Vue.createApp(App);
   app.mount('#app');
   ```
2. 创建根组件实例（如 `App` 组件），并初始化根组件,调用`setup`。

3. 执行`setup` 函数，通过 `Proxy` 实现创建响应式数据,并设置计算属性、方法以及副作用。

4. 将`template` 编译为 `render` 函数，执行渲染函数生成虚拟 DOM,更新到真实DOM。

5. 组件挂载前会触发 `beforeMount` 生命周期钩子，此时虚拟 DOM 已经准备好，但还未插入到实际 DOM 中.

6. 创建渲染 `effect` 追踪组件的响应式数据，每次响应式数据变化时，`effect` 会重新执行，生成新的虚拟 DOM。

7. `mounted` 生命周期钩子在组件挂载完成后触发，此时组件的 DOM 已经插入到页面中。

8. 数据变化时触发响应式更新,进行新旧虚拟 DOM diff，仅更新变化部分的虚拟 DOM。

---

***

## 3. 简述 Vue 的生命周期以及每个阶段做的事

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

    Vue 3 推荐在 setup 中直接请求，因为 setup 在所有生命周期之前调用。

+ setup中为什么没有beforeCreate和created？

   setup 函数最先执行,本身已经承担了初始化阶段的职责，因此 beforeCreate 和 created 钩子不再单独存在。

* * *

## 4. 说一下 Vue 子组件和父组件创建和挂载顺序

**原则**：创建先父后子，挂载先子后父

**Vue 3 生命周期顺序**：
```
父 setup → 父 beforeMount → 子 setup → 子 beforeMount → 子 mounted → 父 mounted
```


* * *

## 5. 说说从 template 到 render 处理过程

+ 作用: Vue的编译器模块“compiler”，将template编译为render函数。编译器流程是先对template进行parse，获得抽象语法树AST，然后标记静态节点,最后将AST转换render函数进行调用渲染.
+ 用途: 让开发者可以使用模板编码降低开发成本,渲染函数编码成本高。

* * *

## 6. Vue 3 多根元素组件

Vue 3 支持多根元素组件，编译时自动使用 `Fragment` 虚拟节点进行包裹，把多个根节点作为其 children，patch 时直接遍历 children 创建或更新。

```vue
<!-- Vue 3 多根元素 -->
<template>
  <header>Header</header>
  <main>Content</main>
  <footer>Footer</footer>
</template>
```

* * *

## 7. Vue组件之间通信方式有哪些

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf775050e1f948bfa52f3c79b3a3e538~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

###  组件通信常用方式： (vue3去除了删除线的api)

+   props
+   $emit / ~~$on~~
+   $parent / ~~$children~~
+   $attrs(父组件声明的自定义属性) / ~~$listeners(父组件声明的所有@事件)~~
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

    +   `eventbus` / `pinia` / `vuex` / `provide`+`inject`

* * *

## 8.子组件可以直接改变父组件的数据么，说明原因

+ 原因: vue组件化开发遵循**单向数据流原则**，如果能互相修改将状态就难以控制与朔源,修改props也会报错.
+ 正规修改方法: 事件传递/$parent

* * *

## 9.Vue中如何扩展一个组件
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
    // 选项合并时,优先原组件的选项
    // 与extends相比能混入多个选项
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
    // 跟minxins的不同是它只能扩展单个对象
    // 跟minxins选项发生冲突，该选项优先级较高，优先起作用
    // 跟原组件选项发生冲突时，优先原组件选项
    const Comp = {
       extends: myextends
    }
    ```

* * *

## 10.怎么缓存当前的组件？缓存后怎么更新？

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

## 11. 什么是递归组件？举个例子说明下？

组件通过组件名称引用它自己，这种情况就是递归组件,必须要设置组件 `name`。
实际开发中类似Tree、Menu这类组件，它们的节点往往包含子节点，子节点结构和父节点往往是相同的。这类组件的数据往往也是树形结构，这种都是使用递归组件的典型场景。

```vue
// 选项式api
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

```vue
// 组合式api
<template>
  <div class="hello">
    {{ msg }}
  </div>
</template>

<script setup>
import { defineOptions } from 'vue';
// 设置组件name
defineOptions({
  name: 'compositionCmp',
});
</script>

```

* * *

## 12. 异步组件是什么？使用场景有哪些？

异步组件,可以控制懒加载,利于打包器的代码分割。

```javascript
// vue3 使用异步组件
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

## 13. 组件与插件的区别
+ 组件：用于构建 UI,独立单元可复用，局部或全局注册，主要用于页面渲染。
+ 插件：用于扩展 Vue 的全局功能，通过 Vue.use() 注册，一般作用于整个应用。

* * *

## 14.说说你对虚拟 DOM 的理解？

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

+ Vue 3 虚拟 DOM 优化

    Vue 3 的编译器会在生成虚拟 DOM 时加入 **静态标记**，跳过不变的节点，减少 Diff 计算，显著提高渲染性能。

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

## 15. 你了解diff算法吗？

### 执行时机:
`响应式数据变更`触发实例执行其更新函数时，更新函数会再次执行render函数获得最新的虚拟DOM，然后`执行patch函数`，并传入新旧两次虚拟DOM对比变化，最后更新变化的DOM。

* * *

### **Vue 3 的 diff 算法**

自研虚拟DOM的Diff算法，时间复杂度 O(n)，主要优化：

1. 静态提升: Vue 3 会在编译阶段将模板中的静态内容标记为静态节点，并直接提升为常量,静态节点在 diff 阶段被完全跳过
```html
<div>
  <div>foo</div> <!-- 静态提升 -->
  <div>bar</div> <!-- 静态提升 -->
  <div>{{ dynamic }}</div>
</div>
```

2. Block Tree (树结构打平) 优化： 区别处理动态内容和静态内容,将动态内容分离出来.
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

3. 优化PatchFlag：标记更新内容,对比能够直接定位动态节点.

```html
<!-- PatchFlag 标记的更加仔细,自动推断类型,提升对比性能 -->
<!-- 仅含 class 绑定 -->
<div :class="{ active }"></div>

<!-- 仅含 id 和 value 绑定 -->
<input :id="id" :value="value">

<!-- 仅含文本子节点 -->
<div>{{ dynamic }}</div>
```

4. 优化双端对比,使用最长递增子序列(LIS)算法：确定哪些旧节点已处于正确顺序，从而减少移动操作.

5. 响应式系统使用 `Proxy` 动态追踪依赖，性能更优

---

#### Vue 3 的 Diff 过程

1. **组件层面**
    - 数据变化触发组件重新渲染。
    - 生成新的组件 VNode 树，并与旧 VNode 树进行 patch 对比。

2. **节点层面**
    - 判断新旧 VNode 是否为同一类型（tag、key、isComment 等）,若不相同，直接替换整个节点。 若相同，则复用已有的 DOM 元素，并继续 patch。
    - 相同节点下,对比属性 进行增删改。
    - 相同节点下,对比子节点。
      - 如果旧节点和新节点均为文本节点，则直接比较文本内容，若不同则更新文本。
      - 如果存在子节点数组，则进入子节点 diff 流程。
3. **子节点diff**
    - 先进行新旧节点列表的头部和尾部同步匹配，快速匹配连续相同部分,进行排除更新；
    - 对剩余未匹配的新节点列表的中间部分创建 `新节点 key 映射列表`,再遍历旧节点列表剩余部分的key构造`新旧节点索引映射数组`，然后通过新旧节点索引映射数组获取`最长递增子序列 (LIS)`,找出无需移动的部分；
    - 通过新旧节点索引映射数组操作节点
      - 将对应值为0的新节点,进行新增创建
      - 对应值不为0的新节点,不在 LIS 内，进行移动
      - 对应值不为0的新节点,在 LIS 内，直接跳过
      - 对不在新旧节点索引映射数组的旧节点进行删除

4. **最终 DOM 更新**
    - 经过以上流程，最终真实 DOM 中仅发生了必要的节点更新、移动、插入与删除，从而达到最小化 DOM 操作的目的。

* * *

## 16. 能说一说双向绑定使用和原理吗？

### **Vue 3 双向绑定**

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
    // 方式1: 使用defineModel
    // model代表父组件传入属性countModel
    const model = defineModel()
    // model.value可以直接操作父组件属性countModel
    function update() {
        model.value++
    }
    // 方式2: 自定动态绑定
    // defineProps(['modelValue'])
    // defineEmits(['update:modelValue'])
</script>

<template>
    <div>Parent bound v-model is: {{ model }}</div>
    <button @click="update">Increment</button>
    <!--自定动态绑定-->
    <!--v-model是语法糖，默认情况下相当于`:modelValue`和`@update:modelValue`。-->
<!--    <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)"/>-->
</template>

<!-- Parent.vue -->
<Child v-model="countModel" />
<!--自定动态绑定-->
<Child :modelValue="countModel" @update:modelValue="$event => (countModel = $event)"/>
<Child v-model:modelValue="countModel" />

```

**底层原理**

Vue 3 的响应式系统基于 **Proxy** 实现，通过 Proxy 对整个 data/ref/reactive 进行拦截，可以实现对对象新增属性和数组变化的自动响应式支持。

- **Proxy** 拦截对对象的访问（如读取、修改、删除等），并相应地处理依赖收集和更新
- 可以直接监听对象的结构变化（新增/删除属性），更高效地实现双向绑定

> 注：Vue 3 中 `sync` 修饰符已被移除，改为使用 `v-model:propName` 语法实现多属性绑定。

* * *

## 17. 说一说你对vue响应式理解？

响应式系统是 Vue 框架的核心特性，实现数据驱动的视图更新。

### **Vue 3 的响应式**

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

### **响应式原理对比**

| **特性**         | **Vue 3 (Proxy)**                    |
|------------------|--------------------------------------|
| **实现方式**     | 使用 `Proxy` 代理整个对象           |
| **深层监听**     | 自动追踪，动态监听                   |
| **动态属性**     | 自动响应式，无需手动处理             |
| **数组监听**     | 原生支持索引和长度变化               |
| **性能**         | 代理整个对象，性能更优               |
| **兼容性**       | 不支持 IE，需现代浏览器              |


* * *

## 18. 动态给vue的data添加一个新的属性时会发生什么？

Vue 3 使用 `Proxy` 响应式机制，动态添加新属性会自动触发更新，无需额外 API。

* * *

## 19. 为什么data属性是一个函数而不是一个对象？

1. **数据隔离**：防止实例之间数据污染。
2. **支持复用**：函数构成单独的作用域(模块),每个组件实例有独立的状态。
3. **组件与根实例的功能区分**：根实例是单例，而组件需要支持多实例化。
4. **代码可维护性**：符合模块化和现代编程的最佳实践。

* * *

## 20. Vue 3 的 reactive

Vue 3 使用 **`reactive`** 创建响应式对象（替代 Vue 2 的 `Vue.observable`）。

```javascript
import { reactive } from "vue";

const state = reactive({
  count: 0,
});

// 修改数据时会触发响应
state.count++;

// 在组件中使用
export default {
  setup() {
    return { state };
  },
  template: `<button @click="state.count++">{{ state.count }}</button>`,
};
```

**优点**：
- 动态监听对象属性的添加、删除和嵌套对象的变化
- 原生监听数组索引和长度的变化
- 适用于组件状态管理和跨组件共享

* * *

## 21. v-show 与 v-if 的区别

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

## 22. v-if和v-for哪个优先级更高？

**Vue 3 中 `v-if` 优先于 `v-for` 执行。**

**注意**: 不推荐同时在同一元素上使用两者，会引发判断错误（`v-if` 执行时 `v-for` 变量尚未定义）。

**正确做法：**
- 使用计算属性或提前过滤数据
- 将两个指令放在不同元素（`v-for` 在外层 `<template>`）

```vue
<!-- 推荐 -->
<template v-for="item in items" :key="item.id">
  <div v-if="item.visible">{{ item.name }}</div>
</template>

<!-- 或使用计算属性 -->
<div v-for="item in visibleItems" :key="item.id">{{ item.name }}</div>
```

* * *

## 23. v-once/v-memo的使用场景有哪些？

| 特性         | **`v-once`**          | **`v-memo`**                |
|--------------|----------------------|----------------------------|
| **功能**     | 一次性渲染，永不更新   | 按依赖项缓存，依赖变化时重新渲染 |
| **更新条件** | 不会更新              | 依赖项变化时重新渲染          |
| **适用场景** | 完全静态的内容         | 复杂逻辑、条件性更新          |
| **灵活性**   | 固定（不可更新）        | 灵活（依赖变化时更新）        |


* * *

## 24. 你写过自定义指令吗？使用场景有哪些？

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
      // userPermissions 包含用户的权限列表
      // ['view', 'edit']
    if (!userPermissions.includes(binding.value)) {
      el.parentNode?.removeChild(el);
    }
  },
});
```
使用：
```vue
<template>
    <button v-permission="['delete', 'admin']">Delete</button>
</template>
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

## 25. 说下$attrs的使用场景

`$attrs` 用于父子组件中传递未显式声明的属性和事件（Vue 3 中 `$listeners` 已合并到 `$attrs`）。

**使用场景：属性透传**

```vue
<!-- 父组件 -->
<ChildComponent id="my-id" title="Hello" @click="handleClick" />

<!-- 子组件 -->
<template>
  <div v-bind="$attrs">I'm a child!</div>
</template>

<script setup>
defineOptions({
  inheritAttrs: false // 禁用自动绑定到根元素
})
</script>
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

## 26. watch/watchEffect/computed的区别及使用?

### vue3中使用

```js
import { ref, computed } from 'vue';

const count = ref(0);
// 计算属性，依赖 count，自动缓存值，只有 count 变化时才重新计算
const doubleCount = computed(() => count.value * 2);

// 使用
console.log(doubleCount.value); // 输出: 0
count.value++;
console.log(doubleCount.value); // 输出: 2

// 读写型computed
const count = ref(0);
const doubleCount = computed({
    get: () => count.value * 2,
    set: (val) => {
        count.value = val / 2;
    }
});
```

```js
import { ref, watch } from 'vue';

const count = ref(0);

// 观察 count 的变化
watch(count, (newValue, oldValue) => {
  console.log(`count 从 ${oldValue} 变为 ${newValue}`);
});

// 改变 count 值会触发 watch 回调
count.value = 5;

// 深度监听
watch(
    count,
    (newValue, oldValue) => {
        console.log('立即执行: ', newValue);
    },
    { immediate: true }
);


```

```js
import { ref, watchEffect } from 'vue';

const count = ref(0);

// 自动追踪 count，当 count 变化时重新运行回调
watchEffect(() => {
  console.log(`The count is: ${count.value}`);
});

// 修改 count 会自动触发 watchEffect 回调
count.value = 10;

```


### 对比

| 特性                    | `computed`                        | `watch`                | `watchEffect`        |
|-------------------------|-----------------------------------|------------------------|----------------------|
| **缓存**                | 自动缓存计算结果                  | 不缓存                    | 不缓存                  |
| **依赖收集**            | 手动指定依赖                      | 手动指定依赖                 | 自动收集`会执行的响应式依赖`        |
| **前后值对比**          | 不提供                            | 提供 `newVal` 和 `oldVal` | 不提供                  |
| **返回值**  | 返回计算后的新值，直接用作模板或逻辑。 | 不返回值，常用于执行副作用（如异步操作）。 | 不返回值，常用于执行副作用        |
| **异步支持**            | 不适合                            | 支持异步操作                 | 支持异步操作，但需手动管理回调中的异步逻辑 |
| **执行时机**            | 在模板或计算属性中访问时触发      | 数据变化时触发(批处理后)          | 数据变化时触发，初次执行时立即触发    |
| **适用场景**            | 计算派生数据                      | 执行副作用和复杂监听             | 简单副作用和自动依赖追踪         |
| **深度监听**            | 不支持                            | 支持（使用 `deep: true`）    | 不支持                  |


### 侦听器的执行时机
概述：Vue 3 中的 `watch` 侦听器回调执行时机主要由其 `flush` 选项控制，可选值包括：

- `pre`（默认）：在组件更新并渲染之前调用回调；
- `post`：在组件渲染并更新 DOM 之后调用回调；
- `sync`：在响应式依赖变更时同步（立即）执行回调；

此外，`watch` 还支持 `immediate` 选项用于在侦听器创建时立即触发一次回调，以及 `deep` 选项用于对对象或数组的深度监听。对于 Composition API，还提供了 `watchEffect`（等同 `flush: 'pre'`）、`watchPostEffect`（等同 `flush: 'post'`）和 `watchSyncEffect`（等同 `flush: 'sync'`）等快捷函数，可根据需求选择最合适的执行时机。citeturn0search9turn0search13

 一、`flush` 执行时机

 1. 默认：`flush: 'pre'`
- **时机**：当侦听的响应式依赖发生变化后，但在组件渲染并更新 DOM 之前执行回调。此时组件尚未应用新的视图更新，因此如果在回调中访问 DOM，仍然是变更前的状态。
- **批处理**：与组件更新一样，多个依赖在同一次“tick”内变化只会触发一次回调，避免重复调用。citeturn0search3turn0search14

 2. 延后：`flush: 'post'`
- **时机**：在响应式依赖变化并完成 DOM 更新之后执行回调，通常基于内部的 `nextTick` 实现。适用于需要访问更新后 DOM 或者依赖最新渲染结果的场景。
- **用法**：可通过 `{ flush: 'post' }` 或直接使用 `watchPostEffect` 代替。citeturn0search1turn0search13

 3. 同步：`flush: 'sync'`
- **时机**：在依赖变化时立即同步执行回调，不等待批处理或渲染阶段。
- **注意**：同步执行虽然可以保证“马上响应”，但在多个依赖同时变化时会多次触发，可能导致性能下降或状态不一致，应谨慎使用。citeturn0search4turn0search9

 二、`immediate` 与 `deep` 选项

- **`immediate`**：默认为 `false`，表示侦听器在创建时不会立即触发回调；当设置为 `true` 时，侦听器在绑定之初会执行一次回调，旧值为 `undefined`。
- **`deep`**：默认为 `false`，仅监听顶层属性；当源为对象或数组且需要监听深层次变更时，可设置 `deep: true`，Vue 会递归遍历所有子属性。citeturn0search9turn0search11

 三、`watchEffect` 及快捷函数

- **`watchEffect`**：无须指定依赖源，内部自动追踪所用响应式属性，默认行为等同 `{ flush: 'pre' }`。
- **`watchPostEffect`**：等同 `{ flush: 'post' }`，在 DOM 更新后执行副作用。
- **`watchSyncEffect`**：等同 `{ flush: 'sync' }`，在依赖变化时同步执行副作用。
- **清理函数**：在异步副作用中，可通过 `onWatcherCleanup` 注册清理逻辑，回调在下次执行前运行。citeturn0search1turn0search15turn0search8


* * *

## 27. 插槽 slot的使用

### 1. 默认 slot
```vue
<!-- 子组件 -->
<slot></slot>

<!-- 父组件 -->
<ChildComponent>默认内容</ChildComponent>
```

### 2. 具名 slot
```vue
<!-- 子组件 -->
<slot name="header"></slot>
<slot name="footer"></slot>

<!-- 父组件使用 # 缩写 -->
<ChildComponent>
  <template #header>头部内容</template>
  <template #footer>底部内容</template>
</ChildComponent>
```

### 3. 作用域 slot（Scoped Slot）
```vue
<!-- 子组件 -->
<script setup>
const items = ref([{ id: 1, name: 'Item 1' }])
</script>
<template>
  <slot name="item" v-for="item in items" :item="item" :key="item.id"></slot>
</template>

<!-- 父组件 -->
<ChildComponent>
  <template #item="{ item }">
    {{ item.name }}
  </template>
</ChildComponent>
```

***

## 28. 说说nextTick的使用和原理？

`nextTick` 在下次 DOM 更新循环结束后执行回调，确保 DOM 已更新。

**使用场景：**
- 响应式数据变化后获取 DOM 更新后的状态
- 在 setup 或 created 中需要获取 DOM 时

**Vue 3 使用**：
```javascript
import { nextTick } from 'vue';

nextTick(() => {
  console.log('DOM 已更新');
});
```

**原理**：通过 `Promise` 微任务将回调加入队列，批处理执行以减少性能开销。

* * *

## 29. 能说说key的作用吗？

**`key` 的作用:**

- **唯一标识,对比性能优化**：标识虚拟 DOM 节点的唯一标识符,提升diff性能,利于复用,降低性能开销。

**使用场景**
`key` 主要用于 `v-for` 指令中，以确保每个子元素具有唯一标识符。

**规范使用:**

- **唯一性**：`key` 应该是列表中每个元素的唯一标识符，最好使用数据中的唯一 ID（如数据库中的主键）作为 `key`。
- **稳定性**：`key` 应该是稳定的，不会随时间改变的属性。
- **避免使用索引**：尽量避免使用数组的索引作为 `key`，因为在数据动态变化时（例如插入或删除元素），使用索引会导致 `key` 的重复或不稳定，可能引发渲染错误或性能问题。

* * *

## 30. Composition API 与 Options API 有什么不同

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

## 31. ref和reactive异同

| **特性**      | **`ref`**                                    | **`reactive`**         |
|-------------|----------------------------------------------|------------------------|
| **模板解包**    | 自动解包,无需使用 `.value`                           | 自动解包                   |
| **响应性**     | 可创建响应式数据                                     | 可创建响应式数据               |
| **数据类型支持**  | 适合基本类型（如数字、字符串、布尔值），也可用于对象或数组（需通过 `.value`）。 | 适合对象和数组，直接操作内部属性。      |
| **实现机制**    | 内部是一个包含 `value` 属性的已被getter/setter的对象。       | 使用 `Proxy` 实现对整个对象的代理。 |
| **访问方式**    | 修改或读取值时需要使用 `.value`。                        | 直接访问或修改属性即可。           |
| **解构后的响应性** | 解构后响应性失效                              | 解构后响应性失效,需使用`toRefs`进行解构 |
| **适用场景**    | 管理单个变量或基本数据类型。                               | 管理复杂对象或数组。             |
| **逻辑复用**    | 使用时需多次解包 `.value`，稍显繁琐。                      | 返回完整的响应式对象，操作更加方便。     |


* * *

## 32. 你知道哪些vue3新特性及优化

**新特性:**

1. **Composition API**
    - 更灵活的逻辑组织方式，支持逻辑复用。
    - 提供 `ref`、`reactive`、`computed`、`watch` 等响应式工具。

2. **新的响应式系统**
    - 基于 `Proxy`，支持深层响应式，动态添加属性无需额外处理。

3. **Teleport**
    - 内容可以渲染到 DOM 的任意位置，适用于模态框、通知等场景。

4. **fragments**
    - 支持组件返回多个根节点,vue内容会自动使用虚拟节点包裹处理，减少不必要的 DOM 层级。

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
    - `<style>` 新增支持选择器、CSS Modules、`useCssModule()`、`v-bind()` 等


**优化:**

1. 编译器优化：静态提升、静态节点合并;虚拟 DOM 改进：更高效的依赖追踪和更新

2. 按需加载功能, 增加 Tree-shaking 支持

3. SSR 优化,提升服务端渲染性能，支持流式渲染，降低白屏时间

4. 核心库更小的运行时/打包体积

5. 增加 TypeScript 支持

6. **API易用/扩展性提升**
    - `v-model`包含了`sync`修饰符的作用
    - 独立的响应式创建(`reactivity`/`ref`)
    - 提供自定义渲染器(`custom renderer`)

* * *


## 33. 你是怎么处理vue项目中的错误的？

分为`逻辑错误`与`请求错误`,再根据错误类型进行`捕获上报`.

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

## 34. Vue要做权限管理该怎么做？控制到按钮级别的权限怎么做？

+ **页面权限**

    `前端方案:`**前端存储所有路由信息**，通过`路由守卫`要求用户登录，用户**登录后根据角色过滤出路由表**,最后通过`router.addRoutes(accessRoutes)`方式动态添加路由。

    `后端方案:`**把所有页面路由信息存在数据库**中，用户登录的时候根据其角色**查询得到其能访问的所有页面路由信息**返回给前端，前端**再通过`addRoutes`动态添加路由**信息
    
    `可维护性:`前端方案实现简单,后续修改需重新打包. 后端方案实现较为复杂,但是维护方便.

+ **按钮权限**

    按钮权限的控制通常会**实现一个指令**，例如`v-permission`，**将按钮要求角色通过值传给v-permission指令**，在指令的`moutned`钩子中可以**判断当前用户角色和按钮是否存在交集**，有则保留按钮，无则移除按钮。

* * *

## 35. SPA、SSR的区别是什么

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


## 36. vue项目本地开发完成后部署到服务器后报404是什么原因呢？
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


## 37. vue-loader是什么？它有什么作用？

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

## 38. 从0到1自己构架一个vue项目，说说有哪些步骤、哪些重要插件、目录结构你会怎么组织

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

## 39. 实际工作中，你总结的vue最佳实践有哪些？

遵循vue官方文档推荐风格.

**编码风格方面：**
+   组件/属性命名避免与HTML元素冲突
+   使用v-for时务必加上key，且不要跟v-if写在一起

**性能方面：**
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

+   vue3 使用`Teleport`,常用于弹窗、模态框等或移出复杂 DOM 减少性能影响

+   vue3 使用`Fragment`,减少无意义的 DOM 节点

+   vue3 使用`Suspense`, 延迟渲染组件，提升性能

**安全：**
+   不使用不可信模板，例如使用用户输入拼接模板：`template: <div> + userProvidedString + </div>`
+   谨慎使用v-html，:url，:style等，避免html、url、样式等注入


# Vue Router

## [API参考: vue router](vue_router.md)

## 40. History模式和Hash模式有何区别？

| **模式**      | **API**                    | **URL 格式**         | **刷新页面支持** | **SEO 支持**  | **适用场景**                  |
|-------------|----------------------------|----------------------|------------|---------------|------------------------------|
| **History** | **`createWebHistory`**     | `/about`            | 需web服务器支持  | 支持           | SSR、SEO 友好的 Web 应用       |
| **Hash**    | **`createWebHashHistory`** | `/#/about`         | 无需web服务器支持 | 不支持         | 纯前端项目，静态文件托管         |
| **SSR**     | **`createMemoryHistory`**  | 不显示 URL         | 不适用        | 不适用         | SSR、测试环境                  |

---

**示例:**

```javascript
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    { path: '/', component: Home },
    { path: '/about', component: About },
];

const router = createRouter({
    history: createWebHistory(), // createWebHistory浏览器环境/createWebHashHistory浏览器hash/createMemoryHistory服务端渲染环境
    // history: createWebHistory('/基础路径/'), // 为路由设置基础路径，如果应用部署在子路径下非常有用。
    routes
});
```

* * *

## 41. 怎么定义动态路由？怎么获取传过来的动态参数？

**动态路由参数配置:**
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

## 42. 如果让你从零开始写一个vue路由，说说你的思路

+ 模块化路由配置：按功能模块划分路由，保持清晰和易维护。
+ 路由懒加载：优化性能，只加载用户访问的模块。
+ 命名路由和路径别名：便于管理和跳转。
+ 嵌套路由：适合复杂页面结构。
+ 404 处理：确保未匹配路由有友好的提示页面。
+ 路由守卫：添加全局、中间件或单个路由级别的守卫，增强安全性和功能性。

* * *

## 43. 怎么实现路由懒加载呢？

**优点:** 利于代码分割,仅加载用户浏览的模块,降低开销.

```javascript
const router = createRouter({
  routes: [{ path: '/users/:id', component: () => import(/* webpackChunkName: "这个分割块的名字" */'./views/UserDetails') }],
  // ...
})
```

* * *

## 44. router-link和router-view是如何起作用的？

### **router-link 常用属性:**

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
    
    <!-- custom 使RouterLink不渲染成a标签,形成自定义组件-->
    <RouterLink to="/about" custom v-slot="{route, href, isActive, isExactActive, navigate}">
      <button :class="{ active: isActive, exact: isExactActive }" @click="navigate">
        Go to About
      </button>
    </RouterLink>
  </div>
</template>
```


### **router-view 常用属性:**

| 属性            | 说明                                         |
|------------------|--------------------------------------------|
| `name`          | 命名视图的名称（默认显示默认路由视图）        |
| `v-slot`        | 用于自定义嵌套路由或视图内容                 |

**示例:**

```js
const routes = [
    {
        path: '/',
        components: {
            // 它们与 `<router-view>` 上的 `name` 属性匹配
            default: Home, // 默认视图
            header, // 渲染到 header命名视图
            footer, // 渲染到 footer命名视图
        },
    },
]
// ...
```

```vue
<template>
  <div>
    <!-- 默认视图 -->
    <router-view></router-view>

    <!-- 命名视图 -->
    <router-view name="header"></router-view>
    <router-view name="footer"></router-view>
    
    <!--v-slot-->
    <RouterView v-slot="{ Component, Route }">
      <keep-alive>
        <component :is="Component" />
      </keep-alive>
    </RouterView>
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

## 45. Vue-router 除了 router-link 怎么实现跳转

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


## 46. 在什么场景下会用到嵌套路由？

需要公用的页面布局,如顶部导航栏/左侧菜单栏/主内容区等,部分内容跟随路由切换,而公用部分不变.

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

## 47. vue-router中如何保护路由？

**路由守卫流程:**
1. beforeRouteLeave: 导航触发失活组件的对应路由时调用.(组件内守卫)
2. beforeEach: 全局前置守卫,在路由切换前调用.(`router.beforeEach((to, from, next) => {})`)
3. beforeRouteUpdate: 导航到当前路由,但是该路由的组件被复用时调用.(组件内守卫)
4. beforeEnter: 路由独享守卫,在`路由配置中定义`.(`{path: '/profile', component: Profile, beforeEnter: (to, from, next) => {}}`)
5. beforeRouteEnter: 导航进入该组件的对应路由时调用.(组件内守卫, 组合式API不支持)
6. beforeResolve: 导航被确认之前,在所有组件内守卫和异步路由组件被解析之后调用.(`router.beforeResolve((to, from, next) => {})`)
7. afterEach: 全局后置守卫,在导航成功完成后调用.(`router.afterEach((to, from) => {})`)

[参考:路由守卫](vue_router.md#导航守卫)

* * *

# Pinia & Vuex

## [vue3推荐状态管理 Pinia API参考](pinia.md)

## [Vuex API参考](vuex.md)

## 48. 对pinia的理解?

Vue3 官方推荐的状态管理库，用来替代 Vuex，提供更轻量、更灵活的状态管理方案。

**作用:**

1. **集中式状态管理**
    - Pinia 和 Vuex 类似,但代码量更少，提供一个中心化的存储，用于跨组件共享和管理状态。

2. **模块化设计**
    - 不再使用vuex命名空间模块,每个状态模块被定义为一个独立的 Store，对应不同的功能模块，方便组织和维护。

3. **动态模块注册**
   - 支持动态创建和注册 store，适用于按需加载模块的场景。

```js
// stores/dynamicStore.js
import { defineStore } from 'pinia'

export const useDynamicStore = defineStore('dynamicStore', () => {
    const count = ref(0)
    function increment() {
        count.value++
    }
    return { count, increment}
})

```

```vue
<template>
  <div>
    <!-- 等待 store 加载完成后展示数据 -->
    <div v-if="store">
      <p>Count: {{ store.count }}</p>
      <button @click="store.increment">Increment</button>
    </div>
    <div v-else>
      加载中...
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// 定义一个 ref 用来保存动态加载的 store
const store = ref(null)

// 动态加载 store
onMounted(async () => {
  const module = await import('@/stores/dynamicStore') // 根据项目路径调整
  const useDynamicStore = module.useDynamicStore
  store.value = useDynamicStore()  // 调用注册Pinia模块
})
</script>

```


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




## 49. 对vuex理解？

Vuex 是 Vue 官方提供的状态管理库，用于集中管理应用的状态和跨组件的状态共享（Vue 3 推荐使用 Pinia）。

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

## 50. pinia与vuex中 module作用与区别？

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
import { ref, computed } from 'vue';


// 函数式定义pinia 模块, 更符合 Vue3 Composition API 风格
export const useCounterStore = defineStore('counter', () => {
    const count = ref(0); // 等价选项式定义 state: () => ({ count: 0 })
    function increment() { // 等价选项式定义 actions: { increment() { this.count++ } }
        count.value++;
    }
    const doubleCount = computed(() => count.value * 2); // 等价选项式定义 getters: { doubleCount() { return this.count * 2 } }
    return { count, increment, doubleCount }; // 函数式需要返回状态和方法
});

// stores/user.js
import { defineStore } from 'pinia';

// 选项式定义pinia 模块
export const useUserStore = defineStore('user', {
    state: () => ({
        name: 'John Doe',
        isLoggedIn: false,
    }),
    getters: {
        isAdmin(state) {
            return state.name === 'admin';
        },
    },
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

## 51. vuex中actions和mutations有什么区别？

| 特性         | Mutations                | Actions                  |
|------------|--------------------------|--------------------------|
| **职责**     | 直接修改状态                   | 执行业务逻辑，分发 `mutations`    |
| **操作类型**   | 必须是同步操作                  | 可以包含异步操作                 |
| **调用方式**   | `commit('mutationName')` | `dispatch('actionName')` |
| **调试支持**   | 可被 Vue DevTools 追踪       | 无法直接追踪状态修改               |
| **是否修改状态** | 直接修改状态                   | 不直接修改状态，通过 `mutation`    |
| **适用场景**   | 简单的同步操作                  | 复杂的业务逻辑或异步操作             |


* * *

## 52.怎么监听vuex / pinia 数据的变化？

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

## 53. 页面刷新后pinia/vuex的数据丢失怎么解决？
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

export const useStore = defineStore('store', () => {
    let someState = JSON.parse(localStorage.getItem('someState')) || 'defaultValue';
    
    function setSomeState(payload) {
        someState = payload;
        localStorage.setItem('someState', JSON.stringify(someState));
    }
    return { someState, setSomeState };
});
```

2. 使用插件 `pinia-plugin-persistedstate`,或使用pinia提供的`$subscribe统一处理`


## 54. 实现vuex思路?

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

## 55. 实现pinia的思路?

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

5. **动态注册**
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

## 56. Vue 3.5 新特性

> Vue 3.5 代号 "Tengen Toppa Gurren Lagann"，是一个功能丰富的次要版本更新

### 56.1 响应式 Props 解构（稳定）

在 Vue 3.5 中，`defineProps` 的解构变量现在自动具有响应式，无需额外配置：

```vue
<script setup>
// Vue 3.5 之前需要启用实验性功能
// Vue 3.5 中已稳定，自动响应式
const { count = 0, msg = 'hello' } = defineProps(['count', 'msg'])

// 解构后的变量是响应式的
// 可以在 watch、computed 中正常使用
watch(() => count, (newVal) => {
  console.log('count changed:', newVal)
})
</script>
```

### 56.2 useTemplateRef()

新增 `useTemplateRef()` API，提供更灵活的模板引用方式：

```vue
<script setup>
import { useTemplateRef, onMounted } from 'vue'

// 通过字符串 key 获取模板引用
const inputRef = useTemplateRef('input')

onMounted(() => {
  inputRef.value?.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```

**相比 `ref` 的优势：**
- 可以在组件逻辑中使用动态的 key
- 更好的 TypeScript 类型推断
- 更清晰的语义

```vue
<script setup>
import { useTemplateRef } from 'vue'

// 动态 key
const dynamicRef = useTemplateRef(() => props.refKey)
</script>
```

### 56.3 useId()

生成在服务端和客户端渲染中稳定的唯一 ID：

```vue
<script setup>
import { useId } from 'vue'

// 生成唯一 ID，SSR 安全
const id = useId()
</script>

<template>
  <form>
    <label :for="id">用户名</label>
    <input :id="id" type="text" />
  </form>
</template>
```

**应用场景：**
- 表单字段关联（label-input）
- 无障碍访问（ARIA 属性）
- SSR 水合时保持一致性

### 56.4 onWatcherCleanup()

在 watcher 中注册清理回调，优雅处理副作用：

```vue
<script setup>
import { watch, onWatcherCleanup } from 'vue'
import { ref } from 'vue'

const searchQuery = ref('')

watch(searchQuery, (query) => {
  const controller = new AbortController()
  
  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then(res => res.json())
    .then(data => {
      // 处理数据
    })
  
  // 注册清理函数，在下次 watch 执行前或组件卸载时调用
  onWatcherCleanup(() => {
    controller.abort()
  })
})
</script>
```

### 56.5 watch pause/resume

watch 返回对象新增 `pause` 和 `resume` 方法：

```vue
<script setup>
import { watch, ref } from 'vue'

const count = ref(0)

const { pause, resume, stop } = watch(count, (val) => {
  console.log('count:', val)
})

// 暂停监听
function pauseWatch() {
  pause()
}

// 恢复监听
function resumeWatch() {
  resume()
}

// 完全停止监听
function stopWatch() {
  stop()
}
</script>
```

### 56.6 Deferred Teleport

`<Teleport>` 组件支持 `defer` 属性，延迟到目标元素存在时再传送：

```vue
<template>
  <!-- defer 属性允许 Teleport 延迟到目标元素存在时再传送 -->
  <Teleport defer to="#late-container">
    <div>这个内容会等待 #late-container 存在后再传送</div>
  </Teleport>
  
  <!-- 目标容器可以在 Teleport 之后渲染 -->
  <div id="late-container"></div>
</template>
```

### 56.7 SSR 改进 / Lazy Hydration

Vue 3.5 改进了服务端渲染和水合：

```vue
<script setup>
import { defineAsyncComponent, hydrateOnVisible, hydrateOnIdle } from 'vue'

// 懒惰水合 - 当组件可见时才水合
const LazyComponent = defineAsyncComponent({
  loader: () => import('./HeavyComponent.vue'),
  hydrate: hydrateOnVisible()
})

// 懒惰水合 - 浏览器空闲时水合
const IdleComponent = defineAsyncComponent({
  loader: () => import('./IdleComponent.vue'),
  hydrate: hydrateOnIdle()
})

// 懒惰水合 - 满足条件时水合
const ConditionalComponent = defineAsyncComponent({
  loader: () => import('./ConditionalComponent.vue'),
  hydrate: hydrateOnInteraction(['click', 'mouseover'])
})
</script>

<template>
  <LazyComponent />
  <IdleComponent />
  <ConditionalComponent />
</template>
```

**水合策略：**
- `hydrateOnVisible()` - 元素可见时水合
- `hydrateOnIdle()` - 浏览器空闲时水合
- `hydrateOnInteraction(events)` - 用户交互时水合
- `hydrateOnMediaQuery(query)` - 媒体查询匹配时水合

### 56.8 响应式系统性能优化

Vue 3.5 对响应式系统进行了重大优化：

1. **内存优化**
   - 响应式系统内存使用减少 56%
   - 大型数组的响应式处理更高效

2. **计算属性优化**
   - computed 依赖追踪更加精确
   - 减少不必要的重新计算

3. **数组性能优化**
   - 数组操作（push、pop 等）性能提升
   - 大数组的响应式追踪更高效

```vue
<script setup>
import { ref, computed, shallowRef } from 'vue'

// Vue 3.5 中大数组性能更好
const bigArray = ref(new Array(10000).fill(0))

// 对于不需要深度响应式的场景，使用 shallowRef 进一步优化
const shallowArray = shallowRef([])
</script>
```

### 56.9 其他改进

1. **自定义元素改进**
```js
import { defineCustomElement } from 'vue'

const MyElement = defineCustomElement({
  // 支持 configureApp 配置应用实例
  configureApp(app) {
    app.config.errorHandler = (err) => {
      console.error(err)
    }
  },
  // ...组件选项
})
```

2. **Transition 组件改进**
```vue
<template>
  <!-- 新增 enterFromClass 和 leaveToClass 属性 -->
  <Transition
    enterFromClass="custom-enter-from"
    leaveToClass="custom-leave-to"
  >
    <div v-if="show">内容</div>
  </Transition>
</template>
```

3. **app.onUnmount() 钩子**
```js
const app = createApp(App)

// 应用卸载时执行清理
app.onUnmount(() => {
  console.log('App unmounted')
})
```

### 56.10 从 Vue 3.4 升级到 Vue 3.5

1. **安装更新**
```bash
npm install vue@3.5
```

2. **破坏性变更**
   - 极少的破坏性变更，大多数项目可直接升级
   - 部分内部 API 类型变更

3. **弃用警告**
   - 旧的 `$refs` 访问方式将在未来版本中弃用
   - 建议迁移到 `useTemplateRef()`

---

## 57. Vue Router 核心知识

### 57.1 History 和 Hash 模式区别

| 特性 | Hash 模式 | History 模式 |
|------|----------|-------------|
| URL 形式 | `/#/path` | `/path` |
| 服务器配置 | 无需配置 | 需要配置回退 |
| SEO | 不友好 | 友好 |
| 兼容性 | 更好 | IE10+ |

```js
import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(), // 或 createWebHashHistory()
  routes: [...]
})
```

### 57.2 动态路由与路由懒加载

```js
const routes = [
  // 动态路由参数
  { path: '/user/:id', component: UserProfile },
  
  // 路由懒加载
  { 
    path: '/about', 
    component: () => import('./views/About.vue') 
  },
  
  // 嵌套路由
  {
    path: '/parent',
    component: Parent,
    children: [
      { path: 'child', component: Child }
    ]
  }
]
```

### 57.3 导航守卫

**守卫类型及执行顺序**：
1. `beforeRouteLeave` - 离开当前组件
2. `beforeEach` - 全局前置守卫
3. `beforeRouteUpdate` - 组件复用时
4. `beforeEnter` - 路由独享守卫
5. `beforeRouteEnter` - 进入组件（无法访问 this）
6. `beforeResolve` - 全局解析守卫
7. `afterEach` - 全局后置钩子

```js
// 全局守卫
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isAuthenticated) {
    next({ name: 'login' })
  } else {
    next()
  }
})

// 组合式 API
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

onBeforeRouteLeave((to, from) => {
  const answer = window.confirm('确定离开？')
  if (!answer) return false
})
```

### 57.4 useRoute 和 useRouter

```js
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()   // 当前路由信息
const router = useRouter() // 路由实例

// 路由信息
console.log(route.path, route.params, route.query, route.meta)

// 编程式导航
router.push('/home')
router.push({ name: 'user', params: { id: 123 }})
router.replace('/login')
router.go(-1)
```

---

## 58. Pinia 状态管理

### 58.1 Pinia vs Vuex 区别

| 特性 | Pinia | Vuex |
|------|-------|------|
| 架构 | 无 mutations | actions/mutations 分离 |
| TypeScript | 原生支持 | 需要额外配置 |
| 模块化 | 扁平化，无命名空间 | 需要 modules 和命名空间 |
| 体积 | 更小（~1KB） | 较大 |
| DevTools | 支持 | 支持 |

### 58.2 定义 Store（组合式 API）

```js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // state
  const count = ref(0)
  
  // getters
  const doubleCount = computed(() => count.value * 2)
  
  // actions
  function increment() {
    count.value++
  }
  
  async function fetchData() {
    const res = await fetch('/api/data')
    count.value = await res.json()
  }

  return { count, doubleCount, increment, fetchData }
})
```

### 58.3 在组件中使用

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'
import { storeToRefs } from 'pinia'

const store = useCounterStore()

// 解构保持响应式
const { count, doubleCount } = storeToRefs(store)

// 直接修改
store.count++

// 批量修改
store.$patch({ count: 10 })

// 重置
store.$reset()

// 订阅变化
store.$subscribe((mutation, state) => {
  console.log('state changed:', state)
})
</script>
```

### 58.4 Store 间交互

```js
import { useUserStore } from './user'

export const useCartStore = defineStore('cart', () => {
  const userStore = useUserStore()
  
  const items = ref([])
  
  const total = computed(() => {
    // 可以直接访问其他 store
    if (userStore.isVip) {
      return items.value.reduce((sum, i) => sum + i.price * 0.9, 0)
    }
    return items.value.reduce((sum, i) => sum + i.price, 0)
  })

  return { items, total }
})
```

---

## 59. Vuex 核心概念

### 59.1 核心概念

```js
import { createStore } from 'vuex'

const store = createStore({
  state: () => ({ count: 0 }),
  
  getters: {
    doubleCount: state => state.count * 2
  },
  
  mutations: {
    INCREMENT(state, payload) {
      state.count += payload
    }
  },
  
  actions: {
    async fetchAndIncrement({ commit }) {
      const data = await fetch('/api')
      commit('INCREMENT', data.value)
    }
  },
  
  modules: {
    user: userModule,
    cart: cartModule
  }
})
```

### 59.2 actions 和 mutations 的区别

| 特性 | mutations | actions |
|------|----------|---------|
| 同步/异步 | 必须同步 | 可以异步 |
| 调用方式 | commit | dispatch |
| 作用 | 直接修改 state | 提交 mutation |
| DevTools | 可追踪 | 可追踪 |

### 59.3 组合式 API 中使用

```vue
<script setup>
import { useStore } from 'vuex'
import { computed } from 'vue'

const store = useStore()

const count = computed(() => store.state.count)
const doubleCount = computed(() => store.getters.doubleCount)

const increment = () => store.commit('INCREMENT', 1)
const fetchData = () => store.dispatch('fetchAndIncrement')
</script>
```

---

## 60. 图片懒加载指令实现

```js
// directives/lazy.js
const lazyLoad = {
  mounted(el, binding) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          el.setAttribute('src', binding.value)
          observer.unobserve(el)
        }
      })
    }, { threshold: 0.1 })
    observer.observe(el)
  }
}

// 注册指令
app.directive('lazy', lazyLoad)

// 使用
// <img v-lazy="imageUrl" />
```

---