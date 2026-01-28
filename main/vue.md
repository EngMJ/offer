# Vue 3 面试题

## 1. MVVM架构 & 渐进式框架

Vue 的 MVVM 架构代表 Model-View-ViewModel，它是一种分离关注点的设计模式，当`Model（数据层）`修改时会通知`ViewModel（视图模型层）`修改 `View（视图层）`，反之亦然。

![mvvm](https://upload.wikimedia.org/wikipedia/commons/8/87/MVVMPattern.png)

Vue 的"渐进式框架"设计理念：可以根据项目的实际需要，逐步引入 Vue 的功能，而不必一次性改变整个项目架构，使用 Vue 生态的全部功能。

---

## 2. Vue 实例挂载的过程中发生了什么？

1. 创建 Vue 应用实例，并挂载 DOM 元素
   ```javascript
   const app = Vue.createApp(App);
   app.mount('#app');
   ```

2. 创建根组件实例（如 `App` 组件），并初始化根组件，调用 `setup`

3. 执行 `setup` 函数，通过 `Proxy` 实现创建响应式数据，并设置计算属性、方法以及副作用

4. 将 `template` 编译为 `render` 函数，执行渲染函数生成虚拟 DOM，更新到真实 DOM

5. 组件挂载前会触发 `beforeMount` 生命周期钩子，此时虚拟 DOM 已经准备好，但还未插入到实际 DOM 中

6. 创建渲染 `effect` 追踪组件的响应式数据，每次响应式数据变化时，`effect` 会重新执行，生成新的虚拟 DOM

7. `mounted` 生命周期钩子在组件挂载完成后触发，此时组件的 DOM 已经插入到页面中

8. 数据变化时触发响应式更新，进行新旧虚拟 DOM diff，仅更新变化部分的虚拟 DOM

---

## 3. Vue 3 的生命周期

分为8个阶段：**创建前后、载入前后、更新前后、销毁前后**，以及特殊场景的生命周期。

### 基础生命周期

| 生命周期 | 描述 |
|---------|------|
| beforeCreate | 组件实例被创建之初 |
| created | 组件实例已完全创建，各种数据可以使用，常用于异步数据获取 |
| beforeMount | 组件挂载之前 |
| mounted | 组件挂载到实例后，DOM 已创建，可用于获取访问数据和 DOM 元素 |
| beforeUpdate | 组件数据发生变化更新之前，可用于获取更新前各种状态 |
| updated | 数据更新之后 |
| beforeUnmount | 组件实例销毁之前，可用于定时器或订阅的取消 |
| unmounted | 组件实例销毁之后 |

### 特殊生命周期

| 生命周期 | 描述 |
|---------|------|
| activated | keep-alive 缓存的组件激活时 |
| deactivated | keep-alive 缓存的组件停用时调用 |
| errorCaptured | 捕获一个来自子孙组件的错误时被调用 |
| renderTracked | 调试钩子，响应式依赖被收集时调用 |
| renderTriggered | 调试钩子，响应式依赖被触发时调用 |
| serverPrefetch | SSR only，组件实例在服务器上被渲染前调用 |

![vue3 生命周期图](https://cn.vuejs.org/assets/lifecycle_zh-CN.W0MNXI0C.png)

### 生命周期相关说明

- **数据请求**：Vue 3 推荐在 setup 中直接请求，因为 setup 在所有生命周期之前调用
- **setup 中为什么没有 beforeCreate 和 created**：setup 函数最先执行，本身已经承担了初始化阶段的职责，因此这两个钩子不再单独存在

---

## 4. 父子组件创建和挂载顺序

**原则**：创建先父后子，挂载先子后父

```
父 setup → 父 beforeMount → 子 setup → 子 beforeMount → 子 mounted → 父 mounted
```

---

## 5. 说说从 template 到 render 处理过程

- **作用**：Vue 的编译器模块 "compiler"，将 template 编译为 render 函数。编译器流程是先对 template 进行 parse，获得抽象语法树 AST，然后标记静态节点，最后将 AST 转换 render 函数进行调用渲染
- **用途**：让开发者可以使用模板编码降低开发成本，渲染函数编码成本高

---

## 6. Vue 3 多根元素组件

Vue 3 支持多根元素组件，编译时自动使用 `Fragment` 虚拟节点进行包裹，把多个根节点作为其 children，patch 时直接遍历 children 创建或更新。

```vue
<template>
  <header>Header</header>
  <main>Content</main>
  <footer>Footer</footer>
</template>
```

---

## 7. Vue 组件之间通信方式有哪些

### 组件通信常用方式

- props
- $emit
- $parent
- $attrs（父组件声明的自定义属性）
- $root
- ref
- Provide / Inject
- Pinia / Vuex
- eventbus（需使用第三方库如 mitt）

### 组件关系通信

- **父子组件**：`props` / `$emit` / `$parent` / `ref` / `$attrs`
- **兄弟组件**：`$parent` / `$root`
- **任意关系**：`eventbus (mitt)` / `pinia` / `provide` + `inject`

---

## 8. 子组件可以直接改变父组件的数据么？

- **原因**：Vue 组件化开发遵循**单向数据流原则**，如果能互相修改将状态就难以控制与溯源，修改 props 也会报错
- **正规修改方法**：事件传递 / $parent

---

## 9. Vue 中如何扩展一个组件

- **逻辑扩展**：mixins、extends、composition api
- **内容扩展**：slots

```js
// 混入 mixins（不推荐，Vue 3 推荐使用 Composition API）
const mymixin = {
   methods: {
      dosomething(){}
   }
}

// 局部混入
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
// extends（只能扩展单个对象）
const myextends = {
   methods: {
      dosomething(){}
   }
}
const Comp = {
   extends: myextends
}
```

---

## 10. 怎么缓存当前的组件？缓存后怎么更新？

使用 `keep-alive` 组件缓存不活动的组件实例，在组件切换过程中将状态保留在内存中，防止重复渲染 DOM。

### Vue 3 中结合 vue-router 的用法

```vue
<router-view v-slot="{ Component }">
  <keep-alive>
    <component :is="Component"></component>
  </keep-alive>
</router-view>
```

### 缓存后获取数据

```js
// 使用 beforeRouteEnter 路由守卫
beforeRouteEnter(to, from, next) {
  next(vm => {
    vm.getData()
  })
}
```

```js
// 使用 activated 钩子（选项式 API）
activated() {
  this.getData()
}
```

```vue
<!-- 使用 onActivated 钩子（组合式 API，推荐） -->
<script setup>
import { onActivated } from 'vue'

onActivated(() => {
  getData()
})
</script>
```

---

## 11. 什么是递归组件？

组件通过组件名称引用它自己，这种情况就是递归组件。实际开发中类似 Tree、Menu 这类组件，它们的节点往往包含子节点，子节点结构和父节点往往是相同的。

```vue
<!-- 选项式 API -->
<template>
  <li>
    <div>{{ model.name }}</div>
    <ul v-show="isOpen" v-if="isFolder">
      <TreeItem
        class="item"
        v-for="model in model.children"
        :key="model.id"
        :model="model">
      </TreeItem>
    </ul>
  </li>
</template>
<script>
export default {
  name: 'TreeItem',
}
</script>
```

```vue
<!-- 组合式 API -->
<script setup>
// 设置组件 name
defineOptions({
  name: 'TreeItem',
});
</script>
```

---

## 12. 异步组件是什么？使用场景有哪些？

异步组件可以控制懒加载，利于打包器的代码分割。

```javascript
import { defineAsyncComponent } from 'vue'

// 方式1: defineAsyncComponent 定义异步组件
const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    resolve(/* loaded component */)
  })
})

// 方式2: import 函数
const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)
```

---

## 13. 组件与插件的区别

- **组件**：用于构建 UI，独立单元可复用，局部或全局注册，主要用于页面渲染
- **插件**：用于扩展 Vue 的全局功能，通过 `app.use()` 注册，一般作用于整个应用

---

## 14. 说说你对虚拟 DOM 的理解？

虚拟 DOM 是一种在 JavaScript 中用对象树来模拟真实 DOM 的技术，Vue 使用虚拟 DOM 来模拟页面结构的状态，当数据发生变化时，通过虚拟 DOM 更新真实 DOM，实现高效的页面更新。

### 为什么使用虚拟 DOM？

- **直接操作真实 DOM 代价较高**：尤其是页面复杂、DOM 节点多的时候
- **提升性能**：通过"最小化 DOM 操作"优化渲染性能，有效减少 DOM 重绘和重排的次数
- **提高开发效率**：开发者不必手动操作 DOM，Vue 会在数据变化时自动更新界面
- **跨平台支持**：虚拟 DOM 能独立于浏览器环境，实现服务端渲染（SSR）和原生应用渲染

### 虚拟 DOM 的工作流程

1. **初始渲染**：将模板编译为渲染函数，生成虚拟 DOM 树
2. **创建真实 DOM**：将虚拟 DOM 转换为真实 DOM 并挂载到页面上
3. **数据更新**：数据变化时，通过 `render` 函数生成新的虚拟 DOM 树
4. **虚拟 DOM 比对（Diff 算法）**：将新树和旧树进行对比，找出不同点
5. **更新真实 DOM**：将补丁应用到真实 DOM 中，完成页面更新

### Vue 3 虚拟 DOM 优化

编译器会在生成虚拟 DOM 时加入**静态标记**，跳过不变的节点，减少 Diff 计算，显著提高渲染性能。

---

## 15. 你了解 diff 算法吗？

### 执行时机

响应式数据变更触发实例执行其更新函数时，更新函数会再次执行 render 函数获得最新的虚拟 DOM，然后执行 patch 函数，传入新旧两次虚拟 DOM 对比变化，最后更新变化的 DOM。

### Vue 3 的 diff 算法优化

自研虚拟 DOM 的 Diff 算法，时间复杂度 O(n)，主要优化：

**1. 静态提升**

```html
<div>
  <div>foo</div> <!-- 静态提升 -->
  <div>bar</div> <!-- 静态提升 -->
  <div>{{ dynamic }}</div>
</div>
```

**2. Block Tree（树结构打平）优化**

```html
<div> <!-- root block -->
  <div>...</div>         <!-- 不会追踪 -->
  <div :id="id"></div>   <!-- 要追踪 -->
  <div>
    <div>{{ bar }}</div> <!-- 要追踪 -->
  </div>
</div>
```

**3. PatchFlag 优化**

```html
<!-- 仅含 class 绑定 -->
<div :class="{ active }"></div>

<!-- 仅含 id 和 value 绑定 -->
<input :id="id" :value="value">

<!-- 仅含文本子节点 -->
<div>{{ dynamic }}</div>
```

**4. 最长递增子序列（LIS）算法**

确定哪些旧节点已处于正确顺序，从而减少移动操作。

**5. 响应式系统使用 Proxy**

动态追踪依赖，性能更优。

### Vue 3 的 Diff 过程

1. **组件层面**：数据变化触发组件重新渲染，生成新的组件 VNode 树，并与旧 VNode 树进行 patch 对比

2. **节点层面**：
   - 判断新旧 VNode 是否为同一类型（tag、key、isComment 等）
   - 若不相同，直接替换整个节点
   - 若相同，则复用已有的 DOM 元素，并继续 patch
   - 对比属性进行增删改
   - 对比子节点

3. **子节点 diff**：
   - 先进行新旧节点列表的头部和尾部同步匹配
   - 对剩余未匹配的部分创建 key 映射
   - 通过最长递增子序列（LIS）找出无需移动的部分
   - 对节点进行新增、移动、删除操作

---

## 16. 能说一说双向绑定使用和原理吗？

### 使用方式

```html
<!-- 默认使用 -->
<input v-model="message">
```

```vue
<!-- 推荐使用 defineModel -->
<script setup>
const model = defineModel()

function update() {
  model.value++
}
</script>

<template>
  <div>Parent bound v-model is: {{ model }}</div>
  <button @click="update">Increment</button>
</template>

<!-- 父组件 -->
<Child v-model="countModel" />
```

```vue
<!-- 多个 v-model 绑定 -->
<UserName v-model:first-name="first" v-model:last-name="last" />
```

### 底层原理

Vue 3 的响应式系统基于 **Proxy** 实现，通过 Proxy 对整个 data/ref/reactive 进行拦截，可以实现对对象新增属性和数组变化的自动响应式支持。

- **Proxy** 拦截对对象的访问（如读取、修改、删除等），并相应地处理依赖收集和更新
- 可以直接监听对象的结构变化（新增/删除属性），更高效地实现双向绑定

---

## 17. 说一说你对 Vue 响应式理解？

响应式系统是 Vue 框架的核心特性，实现数据驱动的视图更新。

### 响应式原理

1. **代理对象**：使用 `reactive` 或 `ref` 包装响应式数据，返回一个 `Proxy` 对象
2. **依赖追踪和触发**：
   - 读取属性时（`get`），记录依赖的副作用（`effect`）
   - 修改属性时（`set`），触发对应副作用更新视图

### 特性

| 特性 | 说明 |
|------|------|
| 实现方式 | 使用 `Proxy` 代理整个对象 |
| 深层监听 | 自动追踪，动态监听 |
| 动态属性 | 自动响应式，无需手动处理 |
| 数组监听 | 原生支持索引和长度变化 |
| 性能 | 代理整个对象，性能更优 |
| 兼容性 | 不支持 IE，需现代浏览器 |

---

## 18. 动态给 data 添加一个新属性时会发生什么？

Vue 3 使用 `Proxy` 响应式机制，动态添加新属性会自动触发更新，无需额外 API。

---

## 19. 为什么 data 属性是一个函数而不是一个对象？

1. **数据隔离**：防止实例之间数据污染
2. **支持复用**：函数构成单独的作用域，每个组件实例有独立的状态
3. **组件与根实例的功能区分**：根实例是单例，而组件需要支持多实例化
4. **代码可维护性**：符合模块化和现代编程的最佳实践

---

## 20. reactive 的使用

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

---

## 21. v-show 与 v-if 的区别

| 特性 | v-show | v-if |
|------|--------|------|
| 实现方式 | 修改 CSS display 样式值 | 动态添加和删除 DOM 元素 |
| 初次渲染成本 | 高（元素始终被渲染） | 低（仅在需要时创建） |
| 条件切换开销 | 低（仅修改样式） | 高（需要创建或销毁 DOM） |
| 大量元素 | 不适合 | 适合 |
| 使用场景 | 频繁显示/隐藏 | 条件变化不频繁 |
| 生命周期影响 | 不会触发组件销毁和重建 | 会触发组件销毁和重建 |

---

## 22. v-if 和 v-for 哪个优先级更高？

**Vue 3 中 `v-if` 优先于 `v-for` 执行。**

**注意**：不推荐同时在同一元素上使用两者，会引发判断错误（`v-if` 执行时 `v-for` 变量尚未定义）。

**正确做法**：
- 使用计算属性或提前过滤数据
- 将两个指令放在不同元素

```vue
<!-- 推荐 -->
<template v-for="item in items" :key="item.id">
  <div v-if="item.visible">{{ item.name }}</div>
</template>

<!-- 或使用计算属性 -->
<div v-for="item in visibleItems" :key="item.id">{{ item.name }}</div>
```

---

## 23. v-once / v-memo 的使用场景

| 特性 | v-once | v-memo |
|------|--------|--------|
| 功能 | 一次性渲染，永不更新 | 按依赖项缓存，依赖变化时重新渲染 |
| 更新条件 | 不会更新 | 依赖项变化时重新渲染 |
| 适用场景 | 完全静态的内容 | 复杂逻辑、条件性更新 |
| 灵活性 | 固定 | 灵活 |

---

## 24. 你写过自定义指令吗？使用场景有哪些？

### 全局指令

```javascript
const app = Vue.createApp({});

app.directive('focus', {
  mounted(el) {
    el.focus();
  },
});
```

### 局部指令（组合式 API）

```vue
<script setup>
const vFocus = {
  created(el, binding, vnode) {},
  beforeMount(el, binding, vnode) {},
  mounted(el, binding, vnode) {},
  beforeUpdate(el, binding, vnode, prevVnode) {},
  updated(el, binding, vnode, prevVnode) {},
  beforeUnmount(el, binding, vnode) {},
  unmounted(el, binding, vnode) {}
}
</script>
```

### 使用方式

```vue
<template>
  <div v-focus:foo.bar="baz"></div>
</template>
```

### 钩子函数

| 钩子函数 | 说明 |
|---------|------|
| created | 在元素绑定指令时调用 |
| mounted | 在绑定元素插入父节点时调用 |
| updated | 在绑定元素更新时调用 |
| beforeUnmount | 在指令与元素解绑时调用 |

### 使用场景示例

**自动聚焦**

```javascript
app.directive('focus', {
  mounted(el) {
    el.focus();
  },
});
```

**懒加载图片**

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

**权限控制**

```javascript
app.directive('permission', {
  mounted(el, binding) {
    const userPermissions = getUserPermissions();
    if (!userPermissions.includes(binding.value)) {
      el.parentNode?.removeChild(el);
    }
  },
});
```

---

## 25. 说下 $attrs 的使用场景

`$attrs` 用于父子组件中传递未显式声明的属性和事件（Vue 3 中 `$listeners` 已合并到 `$attrs`）。

### 使用场景：属性透传

```vue
<!-- 父组件 -->
<ChildComponent id="my-id" title="Hello" @click="handleClick" />

<!-- 子组件 -->
<template>
  <div v-bind="$attrs">I'm a child!</div>
</template>

<script setup>
defineOptions({
  inheritAttrs: false
})
</script>
```

---

## 26. watch / watchEffect / computed 的区别及使用

### 使用示例

```js
import { ref, computed, watch, watchEffect } from 'vue';

const count = ref(0);

// computed - 计算属性，依赖 count，自动缓存
const doubleCount = computed(() => count.value * 2);

// 读写型 computed
const doubleCountRW = computed({
  get: () => count.value * 2,
  set: (val) => { count.value = val / 2; }
});

// watch - 观察特定数据
watch(count, (newValue, oldValue) => {
  console.log(`count 从 ${oldValue} 变为 ${newValue}`);
});

// watch 深度监听
watch(count, (newValue, oldValue) => {
  console.log('立即执行: ', newValue);
}, { immediate: true });

// watchEffect - 自动追踪依赖
watchEffect(() => {
  console.log(`The count is: ${count.value}`);
});
```

### 对比

| 特性 | computed | watch | watchEffect |
|------|----------|-------|-------------|
| 缓存 | 自动缓存计算结果 | 不缓存 | 不缓存 |
| 依赖收集 | 自动收集依赖 | 手动指定依赖 | 自动收集依赖 |
| 前后值对比 | 不提供 | 提供 newVal 和 oldVal | 不提供 |
| 返回值 | 返回计算后的新值 | 不返回值 | 不返回值 |
| 异步支持 | 不适合 | 支持异步操作 | 支持异步操作 |
| 执行时机 | 在模板或计算属性中访问时触发 | 数据变化时触发 | 数据变化时触发，初次立即执行 |
| 适用场景 | 计算派生数据 | 执行副作用和复杂监听 | 简单副作用和自动依赖追踪 |
| 深度监听 | 不支持 | 支持（deep: true） | 不支持 |

### 侦听器的执行时机（flush 选项）

| flush 值 | 时机 | 说明 |
|----------|------|------|
| pre（默认） | 组件更新前 | 批处理，避免重复调用 |
| post | DOM 更新后 | 适用于需要访问更新后 DOM 的场景 |
| sync | 立即同步执行 | 可能导致性能问题，谨慎使用 |

### 快捷函数

- `watchEffect` - 等同 `{ flush: 'pre' }`
- `watchPostEffect` - 等同 `{ flush: 'post' }`
- `watchSyncEffect` - 等同 `{ flush: 'sync' }`

---

## 27. 插槽 slot 的使用

### 默认 slot

```vue
<!-- 子组件 -->
<slot></slot>

<!-- 父组件 -->
<ChildComponent>默认内容</ChildComponent>
```

### 具名 slot

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

### 作用域 slot

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

---

## 28. 说说 nextTick 的使用和原理？

`nextTick` 在下次 DOM 更新循环结束后执行回调，确保 DOM 已更新。

**使用场景**：
- 响应式数据变化后获取 DOM 更新后的状态
- 在 setup 或 created 中需要获取 DOM 时

```javascript
import { nextTick } from 'vue';

nextTick(() => {
  console.log('DOM 已更新');
});
```

**原理**：通过 `Promise` 微任务将回调加入队列，批处理执行以减少性能开销。

---

## 29. 能说说 key 的作用吗？

**作用**：
- **唯一标识，对比性能优化**：标识虚拟 DOM 节点的唯一标识符，提升 diff 性能，利于复用，降低性能开销

**使用场景**：
- `key` 主要用于 `v-for` 指令中，确保每个子元素具有唯一标识符

**规范使用**：
- **唯一性**：`key` 应该是列表中每个元素的唯一标识符，最好使用数据中的唯一 ID
- **稳定性**：`key` 应该是稳定的，不会随时间改变的属性
- **避免使用索引**：尽量避免使用数组的索引作为 `key`

---

## 30. Composition API 与 Options API 有什么不同

| 特性 | 选项式 API | 组合式 API |
|------|-----------|-----------|
| 代码组织方式 | 按功能类型分组（data、methods 等） | 按逻辑分组，将相关逻辑集中 |
| 逻辑复用 | 通过 mixins 或 extends，可能导致命名冲突 | 使用自定义复用逻辑，清晰易管理 |
| 类型支持 | 较弱，对 TypeScript 支持有限 | 原生支持 TypeScript |
| 学习曲线 | 易于上手，适合初学者 | 适合复杂项目和高级开发者 |
| 代码可读性 | 简单场景下代码易读 | 复杂逻辑集中，代码更清晰 |
| 适用场景 | 小型项目或简单组件 | 中大型项目 |

**注意**：`Composition API` 和 `Options API` 可以一起使用，但作用域并不共享，不推荐混用。

---

## 31. ref 和 reactive 异同

| 特性 | ref | reactive |
|------|-----|----------|
| 模板解包 | 自动解包，无需使用 .value | 自动解包 |
| 数据类型支持 | 适合基本类型，也可用于对象/数组 | 适合对象和数组 |
| 实现机制 | 包含 value 属性的对象 | 使用 Proxy 实现对整个对象的代理 |
| 访问方式 | 需要使用 .value | 直接访问或修改属性 |
| 解构后的响应性 | 解构 .value 后失效，可使用 toRef | 解构后失效，需使用 toRefs |
| 适用场景 | 管理单个变量或基本数据类型 | 管理复杂对象或数组 |

---

## 32. Vue 3 常用 Hooks 有哪些？如何自定义 Hook？

### 什么是 Vue Hooks（Composables）

Vue 3 中的 Hooks 通常称为 **Composables**（组合式函数），是利用 Composition API 封装的可复用逻辑函数，以 `use` 开头命名。

### 内置 Hooks

| Hook | 作用 |
|------|------|
| `ref` / `reactive` | 创建响应式数据 |
| `computed` | 创建计算属性 |
| `watch` / `watchEffect` | 监听数据变化 |
| `onMounted` / `onUnmounted` | 生命周期钩子 |
| `provide` / `inject` | 依赖注入 |
| `toRef` / `toRefs` | 响应式转换 |
| `shallowRef` / `shallowReactive` | 浅层响应式 |
| `useTemplateRef` | 模板引用（3.5+） |
| `useId` | 生成唯一 ID（3.5+） |

### 自定义 Hook 示例

**useFetch - 数据请求**

```js
import { ref, watchEffect, toValue } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const loading = ref(false)

  async function fetchData() {
    loading.value = true
    try {
      const res = await fetch(toValue(url))
      data.value = await res.json()
    } catch (e) {
      error.value = e
    } finally {
      loading.value = false
    }
  }

  watchEffect(() => fetchData())
  return { data, error, loading, refetch: fetchData }
}
```

### 自定义 Hook 最佳实践

1. **命名规范**：以 `use` 开头
2. **单一职责**：每个 Hook 只负责一个功能
3. **返回响应式数据**：返回 ref 或 reactive 对象
4. **清理副作用**：在 `onUnmounted` 中清理事件监听、定时器等

---


## 33. 你是怎么处理 Vue 项目中的错误的？

### 全局错误处理

```javascript
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

app.config.errorHandler = (err, instance, info) => {
  console.error(`[Vue error]: ${err.message}`);
  console.error(`[Component name]: ${instance?.$options?.name || 'anonymous'}`);
  console.error(`[Error info]: ${info}`);
};

app.mount('#app');
```

### 组件级错误处理（onErrorCaptured）

```vue
<script setup>
import { onErrorCaptured } from 'vue';

onErrorCaptured((err, instance, info) => {
  // 返回 false 停止向上传递错误
  return false;
})
</script>
```

### Promise 错误处理

```javascript
window.addEventListener('unhandledrejection', (event) => {
  console.error(`[Unhandled promise rejection]: ${event.reason}`);
});
```

### Axios 全局网络请求错误处理

```javascript
import axios from 'axios';

const instance = axios.create({
  baseURL: '/api',
  timeout: 10000,
});

instance.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error(`[Response error]: ${error.response?.data?.message || error.message}`);
    return Promise.reject(error);
  }
);

export default instance;
```

---

## 34. Vue 要做权限管理该怎么做？

### 页面权限

- **前端方案**：前端存储所有路由信息，通过路由守卫要求用户登录，登录后根据角色过滤出路由表，通过 `router.addRoute()` 动态添加路由
- **后端方案**：把所有页面路由信息存在数据库中，用户登录时根据角色查询得到其能访问的页面路由信息返回给前端

### 按钮权限

实现一个 `v-permission` 指令，将按钮要求角色通过值传给指令，在指令的 `mounted` 钩子中判断当前用户角色和按钮是否存在交集，有则保留，无则移除。

---





## 35. 你知道哪些 Vue 3 新特性及优化

### 新特性

1. **Composition API**：更灵活的逻辑组织方式，支持逻辑复用
2. **新的响应式系统**：基于 Proxy，支持深层响应式，动态添加属性无需额外处理
3. **Teleport**：内容可以渲染到 DOM 的任意位置
4. **Fragments**：支持组件返回多个根节点
5. **Suspense**：异步组件渲染时提供加载占位内容的能力
6. **新的全局 API**：将全局配置迁移到 app 实例，增强模块化
7. **Composition API Hooks**：提供 onMounted、onUpdated 等更灵活的生命周期管理
8. **Emits 和 Props 验证**：显式定义事件和灵活的 props 验证
9. **增强单文件组件**：`<script setup>` 提供简洁语法，`<style>` 新增 v-bind() 等

### 优化

1. 编译器优化：静态提升、静态节点合并
2. 按需加载功能，增加 Tree-shaking 支持
3. SSR 优化，支持流式渲染
4. 核心库更小的运行时/打包体积
5. 增强 TypeScript 支持
6. API 易用/扩展性提升

---


## 36. Vue 3.5 有哪些新特性？

| 特性 | 说明 |
|------|------|
| 响应式 Props 解构 | `defineProps` 解构的变量自动具有响应式，无需 `toRefs` |
| `useTemplateRef()` | 通过字符串 key 获取模板引用，支持动态 key，类型推断更好 |
| `useId()` | 生成 SSR/CSR 一致的唯一 ID，用于表单 label 关联、ARIA 属性等 |
| `onWatcherCleanup()` | 在 watcher 回调中注册清理函数，下次执行前或组件卸载时调用 |
| watch pause/resume | `watch()` 返回 `{ pause, resume, stop }`，可暂停/恢复监听 |
| Deferred Teleport | `<Teleport defer>` 延迟到目标元素存在后再传送内容 |
| Lazy Hydration | 懒惰水合，按需激活 SSR 渲染的静态 HTML |
| 性能优化 | 响应式内存减少 56%，computed 依赖追踪更精确，数组操作更快 |

### 代码示例

```js
// 响应式 Props 解构（3.5 之前需要 toRefs）
const { count = 0, msg = 'hi' } = defineProps(['count', 'msg'])

// useTemplateRef - 比 ref() 更清晰
const inputRef = useTemplateRef('input')
onMounted(() => inputRef.value?.focus())

// useId - SSR 安全
const id = useId()  // 如 "v-0", "v-1"

// onWatcherCleanup - 自动取消请求
watch(query, (q) => {
  const ctrl = new AbortController()
  fetch(`/api?q=${q}`, { signal: ctrl.signal })
  onWatcherCleanup(() => ctrl.abort())
})

// watch pause/resume
const { pause, resume } = watch(data, handler)
pause()   // 暂停
resume()  // 恢复
```


## 37. VueUse 库有哪些常用函数？

VueUse 是基于 Composition API 的实用函数集合库，提供 200+ 个开箱即用的 Hooks。

```bash
npm install @vueuse/core
```

### 常用函数

| 分类 | 函数 | 作用 |
|------|------|------|
| 状态 | `useStorage` | 响应式本地存储 |
| 状态 | `useToggle` | 布尔值切换 |
| 状态 | `useRefHistory` | 撤销/重做 |
| 浏览器 | `useClipboard` | 剪贴板操作 |
| 浏览器 | `useTitle` | 响应式页面标题 |
| 浏览器 | `useColorMode` | 深色/浅色模式切换 |
| 传感器 | `useMouse` | 鼠标位置 |
| 传感器 | `useWindowSize` | 窗口尺寸 |
| 传感器 | `useScroll` | 滚动位置 |
| 网络 | `useFetch` | 响应式 Fetch API |
| 网络 | `useWebSocket` | WebSocket 连接 |
| 元素 | `onClickOutside` | 点击元素外部 |
| 元素 | `onKeyStroke` | 按键监听 |
| 工具 | `useVModel` | 简化 v-model 实现 |
| 工具 | `useDebounceFn` | 防抖函数 |

### 使用示例

```js
import { useStorage, useDark, onClickOutside, useDebounceFn } from '@vueuse/core'

// 响应式本地存储
const user = useStorage('user', { name: '' })

// 深色模式
const isDark = useDark()

// 点击外部关闭
const modal = ref(null)
onClickOutside(modal, () => isOpen.value = false)

// 防抖搜索
const search = useDebounceFn(() => fetchData(), 500)
```

---

## 38. Vue 3 生态常用库有哪些？

### 核心库

| 库 | 作用 |
|---|------|
| Vue Router | 官方路由管理 |
| Pinia | 官方状态管理（替代 Vuex） |
| VueUse | 实用 Composition API 函数集 |

### UI 组件库

| 库 | 特点 | 适用场景 |
|---|------|---------|
| Element Plus | 功能全面，企业级 | 后台管理系统 |
| Ant Design Vue | 蚂蚁设计规范 | 中后台应用 |
| Naive UI | TypeScript 编写，性能好 | 现代化项目 |
| Vant | 轻量移动端组件 | 移动端 H5 |
| Vuetify | Material Design 风格 | 跨平台应用 |

### 开发工具

| 库 | 作用 |
|---|------|
| Vite | 新一代构建工具 |
| Nuxt 3 | SSR/SSG 框架 |
| VitePress | 静态文档站点生成 |
| unplugin-vue-components | 组件自动按需导入 |
| unplugin-auto-import | API 自动导入 |

### 其他常用库

| 库 | 作用 |
|---|------|
| Axios | HTTP 客户端 |
| @tanstack/vue-query | 服务端状态管理 |
| VeeValidate | 表单验证 |
| vue-i18n | 国际化 |
| ECharts | 可视化图表 |
| vue-draggable-plus | 拖拽排序 |

### 技术选型建议

```
后台管理: Vite + Vue 3 + Pinia + Vue Router + Element Plus + Axios
移动端 H5: Vite + Vue 3 + Pinia + Vant + Axios
SSR 应用: Nuxt 3 + Pinia
文档站点: VitePress
```

---

## 39. SPA、SSR 的区别是什么

| 特性 | SPA（单页应用） | SSR（服务端渲染） |
|------|----------------|------------------|
| 定义 | 一个 HTML 页面，内容通过 JavaScript 动态加载 | 服务端生成完整的 HTML 页面 |
| 页面渲染 | 前端渲染 | 服务端渲染 |
| 首屏加载速度 | 较慢 | 较快 |
| SEO 支持 | 较差 | 良好 |
| 用户体验 | 页面切换流畅 | 可能需要重新加载 |
| 服务器压力 | 低 | 高 |
| 适用场景 | 后台管理 | 电商、新闻、博客 |

**替代方案**：
- 混合模式（SSR + SPA）：使用 Nuxt.js，首屏采用 SSR，其他页面使用 SPA
- 预渲染：打包时直接渲染静态内容

---

## 40. Vue 项目部署后报 404 是什么原因？

**原因**：单页面应用只有一个 HTML，在页面切换时 nginx 会去访问对应 HTML，这些 HTML 不存在所以 404。

**SPA nginx 正确配置**：

```nginx
server {
  listen 80;
  server_name www.xxx.com;

  location / {
    index /data/dist/index.html;
    try_files $uri $uri/ /index.html;
  }
}
```

**注意**：当路由为 Hash 模式时即使不进行配置，依然能够正确访问，因为 Hash 改变页面并不会去访问新的 HTML。

---


---

# Vue Router

## 41. History 模式和 Hash 模式有何区别？

| 模式 | API | URL 格式 | 刷新页面支持 | SEO 支持 | 适用场景 |
|------|-----|----------|------------|---------|---------|
| History | createWebHistory | /about | 需 Web 服务器支持 | 支持 | SSR、SEO 友好的 Web 应用 |
| Hash | createWebHashHistory | /#/about | 无需服务器支持 | 不支持 | 纯前端项目，静态文件托管 |
| SSR | createMemoryHistory | 不显示 URL | 不适用 | 不适用 | SSR、测试环境 |

```javascript
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(),
  routes
});
```

---

## 42. 怎么定义动态路由？怎么获取传过来的动态参数？

### 配置

```javascript
{ path: '/users/:id', component: User }
```

### 获取

```vue
<script setup>
import { useRoute } from 'vue-router';
const route = useRoute();
console.log(route.params.id);
</script>
```

**注意**：
- 通过 watch route 的变化，可以动态获取路由参数
- 404 页面通过放置于路由列表最后进行正则匹配

---

## 43. 如果让你从零开始写一个 Vue 路由，说说你的思路

- 模块化路由配置：按功能模块划分路由
- 路由懒加载：优化性能，只加载用户访问的模块
- 命名路由和路径别名：便于管理和跳转
- 嵌套路由：适合复杂页面结构
- 404 处理：确保未匹配路由有友好的提示页面
- 路由守卫：添加全局、中间件或单个路由级别的守卫

---

## 44. 怎么实现路由懒加载呢？

```javascript
const router = createRouter({
  routes: [
    { 
      path: '/users/:id', 
      component: () => import('./views/UserDetails.vue') 
    }
  ],
});
```

**优点**：利于代码分割，仅加载用户浏览的模块，降低开销。

---

## 45. router-link 和 router-view 是如何起作用的？

### router-link 常用属性

| 属性 | 说明 |
|------|------|
| to | 跳转目标 |
| replace | 替换当前历史记录 |
| custom | 是否自定义内容 |
| active-class | 激活时的 CSS 类 |

```vue
<template>
  <router-link to="/home">Go to Home</router-link>
  <router-link :to="{ name: 'User', params: { id: 1 }}">User 1</router-link>
  
  <!-- custom 使 RouterLink 不渲染成 a 标签 -->
  <RouterLink to="/about" custom v-slot="{ href, isActive, navigate }">
    <button :class="{ active: isActive }" @click="navigate">Go to About</button>
  </RouterLink>
</template>
```

### router-view 常用属性

| 属性 | 说明 |
|------|------|
| name | 命名视图的名称 |
| v-slot | 用于自定义嵌套路由或视图内容 |

```vue
<template>
  <!-- 默认视图 -->
  <router-view></router-view>

  <!-- 命名视图 -->
  <router-view name="header"></router-view>
  
  <!-- v-slot -->
  <RouterView v-slot="{ Component }">
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </RouterView>
</template>
```

---

## 46. Vue-router 除了 router-link 怎么实现跳转

### 编程导航

```vue
<script setup>
import { useRouter } from 'vue-router'
const router = useRouter();

router.push('/users/eduardo')
router.push({ path: '/users/eduardo' })
router.push({ name: 'user', params: { username: 'eduardo' } })
router.replace('/login')
router.go(-1)
</script>
```

---

## 47. 在什么场景下会用到嵌套路由？

需要公用的页面布局，如顶部导航栏/左侧菜单栏/主内容区等，部分内容跟随路由切换，而公用部分不变。

```js
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        path: 'profile',
        component: UserProfile,
      },
    ],
  },
]
```

---

## 48. vue-router 中如何保护路由？

### 路由守卫流程

1. `beforeRouteLeave`：离开当前组件时调用（组件内守卫）
2. `beforeEach`：全局前置守卫
3. `beforeRouteUpdate`：组件复用时调用（组件内守卫）
4. `beforeEnter`：路由独享守卫
5. `beforeRouteEnter`：进入组件时调用（组合式 API 不支持）
6. `beforeResolve`：导航被确认之前
7. `afterEach`：全局后置守卫

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

---

# Pinia

## 49. 对 Pinia 的理解？

Vue 3 官方推荐的状态管理库，用来替代 Vuex，提供更轻量、更灵活的状态管理方案。

### 特点

1. **集中式状态管理**：代码量更少
2. **模块化设计**：不再使用命名空间模块，每个状态模块是独立的 Store
3. **动态模块注册**：支持动态创建和注册 store
4. **响应式数据**：基于 Vue 的响应式系统
5. **简化开发**：直接使用 state 和 actions，无需 mutation
6. **Composition API 集成**：与 Vue 3 无缝结合
7. **支持服务端渲染**
8. **内置 TypeScript 支持**

### Pinia 与 Vuex 的区别

| 特性 | Pinia | Vuex |
|------|-------|------|
| 定义复杂度 | 较低 | 较高 |
| 类型支持 | 内置 TypeScript 支持 | 需要手动配置 |
| 模块化支持 | 原生模块化设计 | 通过命名空间实现 |
| 开发体验 | 简洁直观 | 偏繁琐 |

---

## 50. Pinia 的使用

### 定义 Store（组合式 API，推荐）

```javascript
// stores/counter.js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0);
  
  function increment() {
    count.value++;
  }
  
  const doubleCount = computed(() => count.value * 2);
  
  return { count, increment, doubleCount };
});
```

### 定义 Store（选项式 API）

```javascript
// stores/user.js
import { defineStore } from 'pinia';

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
  },
});
```

### 在组件中使用

```vue
<template>
  <div>
    <h1>Counter: {{ counter.count }}</h1>
    <button @click="counter.increment">Increase Counter</button>
  </div>
</template>

<script setup>
import { useCounterStore } from '@/stores/counter';
import { storeToRefs } from 'pinia';

const counter = useCounterStore();

// 解构保持响应式
const { count, doubleCount } = storeToRefs(counter);

// 直接修改
counter.count++;

// 批量修改
counter.$patch({ count: 10 });

// 重置（仅选项式 API 定义的 store 支持）
// counter.$reset();

// 订阅变化
counter.$subscribe((mutation, state) => {
  console.log('state changed:', state);
});
</script>
```

---

## 51. 怎么监听 Pinia 数据的变化？

### 使用 Vue 的 watch 函数

```javascript
import { watch } from 'vue';
import { useStore } from './stores/store';

const store = useStore();

watch(
  () => store.someState,
  (newValue, oldValue) => {
    console.log('someState 发生变化:', oldValue, '=>', newValue);
  }
);
```

### 使用 store.$subscribe 方法

```javascript
store.$subscribe((mutation, state) => {
  console.log('store 状态变化:', mutation, state);
});
```

---

## 52. 页面刷新后 Pinia 数据丢失怎么解决？

可以通过将数据保存在浏览器的本地存储中来实现。

### 手动实现

```javascript
import { defineStore } from 'pinia';
import { ref, watch } from 'vue';

export const useStore = defineStore('store', () => {
  const someState = ref(JSON.parse(localStorage.getItem('someState')) || 'defaultValue');
  
  watch(someState, (newValue) => {
    localStorage.setItem('someState', JSON.stringify(newValue));
  }, { deep: true });
  
  return { someState };
});
```

### 使用插件

使用 `pinia-plugin-persistedstate` 或使用 pinia 提供的 `$subscribe` 统一处理。
