# Pinia 2.2.2 API参考

## 选项式 API Store 示例

### 1. 定义选项式 store（`stores/counter.js`）
```javascript
// stores/counter.js
import { defineStore } from 'pinia'

// 定义一个选项式 store，类似于 Vuex 的写法
export const useCounterStore = defineStore({
  // Store 的唯一 ID
  id: 'counter',
  
  // 定义状态 state，必须是一个函数，返回初始状态
  state: () => ({
    count: 0,
  }),

  // 定义 getter，用于派生状态
  getters: {
    doubleCount: (state) => state.count * 2, // 计算 count 的两倍
  },

  // 定义 actions，用于修改 state 或执行异步操作
  actions: {
    increment() {
      this.count++ // 增加 count
    },
    decrement() {
      this.count-- // 减少 count
    },
  },
})
```

### 2. 在组件中使用选项式 store

```vue
<template>
  <div>
    <p>Count: {{ store.count }}</p> <!-- 直接访问 store 中的状态 -->
    <p>Double Count: {{ store.doubleCount }}</p> <!-- 通过 getter 访问派生状态 -->
    <button @click="store.increment">Increment</button> <!-- 通过 action 增加 count -->
    <button @click="store.decrement">Decrement</button> <!-- 通过 action 减少 count -->
  </div>
</template>

<script>
import { useCounterStore } from './stores/counter'

export default {
  setup() {
    // 使用 useCounterStore 获取 store 实例
    const store = useCounterStore()

    // $patch
    
    // $reset()
    
    // 将 store 返回给模板
    return { store }
  },
}
</script>
```

## 组合式 API Store 示例

### 1. 定义组合式 store（`stores/counter.js`）

```javascript
// stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

// 使用组合式 API 定义 store
export const useCounterStore = defineStore('counter', () => {
  // 定义状态 ref
  const count = ref(0)

  // 定义 getter，使用 computed 来派生状态
  const doubleCount = computed(() => count.value * 2)

  // 定义 action，可以是任何修改状态的函数
  function increment() {
    count.value++
  }

  function decrement() {
    count.value--
  }

  // 返回 store 中的所有属性和方法
  return {
    count,
    doubleCount,
    increment,
    decrement,
  }
})
```

### 2. 在组件中使用组合式 store

```vue
<template>
  <div>
    <p>Count: {{ count }}</p> <!-- 直接访问 store 中的状态 -->
    <p>Double Count: {{ doubleCount }}</p> <!-- 通过 getter 访问派生状态 -->
    <button @click="increment">Increment</button> <!-- 通过 action 增加 count -->
    <button @click="decrement">Decrement</button> <!-- 通过 action 减少 count -->
  </div>
</template>

<script>
import { useCounterStore } from './stores/counter'

export default {
  setup() {
    // 使用 useCounterStore 获取 store 实例
    const { count, doubleCount, increment, decrement } = useCounterStore()

    // 返回 store 的各个属性和方法到模板
    return { count, doubleCount, increment, decrement }
  },
}
</script>
```

## 三、两种方式的对比与总结

1. **选项式 store**：
    - 类似于 Vue 2.x 或 Vuex 的写法，主要通过 `state`, `getters` 和 `actions` 来定义 store 的功能。
    - 更适合习惯 Vue 选项式 API 的开发者。
    - 例子中，store 的状态是通过 `state` 定义的，`getters` 用于计算派生状态，`actions` 用于修改状态。

2. **组合式 store**：
    - 直接使用 Vue 3 的 `ref`, `reactive`, `computed` 和函数来定义 store，完全遵循组合式 API 的思想。
    - 代码结构更灵活，适合使用 Vue 3 的开发者。
    - 例子中，状态通过 `ref` 定义，getter 使用 `computed`，actions 是普通的函数。

## 选项式Store在选项式组件中使用示例

在 Vue 3 中，Pinia 提供函数：

- **`mapStores`**：将多个 store 映射到组件中`computed` 属性。
- **`mapState`**：将 store 的 `state` 映射为组件中的 `computed` 属性。
- **`mapWritableState`**：将 store 的 `state` 映射为可写的 `computed` 属性。
- **`mapActions`**：将 store 的 `actions` 映射为组件中的`methods`方法。

### 1. 定义选项式 Pinia Store（`stores/counter.js`）

```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore({
  id: 'counter',
  state: () => ({
    count: 0,
    name: 'Pinia Store',
  }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    },
    
    updateState(payload) {
      this.$patch(payload)
    },
    
    resetStore() {
      this.$reset()
    },
  },
})
```

### 2. 在选项式组件中使用

```vue
<template>
  <div>
    <p>Count: {{ count }}</p> <!-- 通过 mapState 映射 count -->
    <p>Double Count: {{ doubleCount }}</p> <!-- 通过 mapState 映射 doubleCount -->
    <p>Name: {{ name }}</p> <!-- 通过 mapWritableState 映射 name 可直接修改 -->

    <button @click="increment">Increment</button> <!-- 通过 mapActions 映射 increment -->
    <button @click="decrement">Decrement</button> <!-- 通过 mapActions 映射 decrement -->

    <!-- 直接修改 count 和 name -->
    <button @click="count = 100">Set Count to 100</button>
    <button @click="name = 'New Name'">Set Name</button>

    <!-- 使用 $patch 批量修改状态 -->
    <button @click="updateState({ count: 50, name: 'Updated Name' })">Patch Count and Name</button>
    
    <!-- 重置 store 的状态 -->
    <button @click="resetStore">Reset Store</button>
  </div>
</template>

<script>
import { mapStores, mapState, mapWritableState, mapActions } from 'pinia'
import { useCounterStore } from './stores/counter'

export default {
  computed: {
    // 使用 mapStores 将 store 映射为 this.$counterStore
    ...mapStores(useCounterStore),

    // 使用 mapState 映射 state 和 getters
    ...mapState(useCounterStore, ['count', 'doubleCount']),
    
    // 使用 mapWritableState 映射可直接修改的 state（例如 name）
    ...mapWritableState(useCounterStore, ['name']),
  },
  
  methods: {
    // 使用 mapActions 将 actions 映射为方法
    ...mapActions(useCounterStore, ['increment', 'decrement', 'updateState', 'resetStore']),
  },
}
</script>
```

### 3. 功能说明

3. **`mapWritableState`**：
    - `mapWritableState` 是 `mapState` 的可写版本，它允许映射的 `state` 可以直接在模板或组件中进行修改。
    - 在这个示例中，我们将 `name` 映射为一个可写的 `computed` 属性，可以直接在模板中通过 `name = 'New Name'` 来修改 `name` 的值。


