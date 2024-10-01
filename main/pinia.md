# Pinia 2.2.2 API参考

## 创建应用

### 1.Vue3示例:

```js

import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const pinia = createPinia()
const app = createApp(App)

app.use(pinia)
app.mount('#app')

```

### 2.Vue2示例:

```js

import { createPinia, PiniaVuePlugin } from 'pinia'

Vue.use(PiniaVuePlugin)
const pinia = createPinia()

new Vue({
  el: '#app',
  // 其他配置...
  // ...
  // 请注意，同一个`pinia'实例
  // 可以在同一个页面的多个 Vue 应用中使用。
  pinia,
})

```

## 订阅

### 1.state订阅:

```js
<script setup>
import { useCounterStore } from './stores/counter'
import { watch } from 'vue'

// 获取 store 实例
const store = useCounterStore()

// 1. 使用 $subscribe 订阅变化
store.$subscribe((mutation, state) => {
   // import { MutationType } from 'pinia'
   // 返回 'direct' | 'patch object' | 'patch function', 可与MutationType对比
   console.log(mutation.type)
   
   // 返回state对应的key值
   console.log(mutation.storeId) 
        
   // 在mutation.type === 'patch object'时,返回$patch的参数对象
   console.log(mutation.payload)

   // 每当状态发生变化时，将整个 state 持久化到本地存储。
   localStorage.setItem('key', JSON.stringify(state))
        
   // 输出变化后的状态
   console.log('Current state:', state)

})

// 此订阅器即便在组件卸载之后仍会被保留
store.$subscribe(callback, { detached: true })

   
   
// 2. 使用watch 订阅变化
watch(
   pinia.state,
   (state) => {
   // 每当状态发生变化时，将整个 state 持久化到本地存储。
   localStorage.setItem('piniaState', JSON.stringify(state))
   },
   { deep: true }
)
</script>


```

### 2.action订阅:

```js

<script setup>
import { useCounterStore } from './stores/counter'

// 获取 store 实例
const store = useCounterStore()

// 1. 使用 $onAction 订阅action变化
// 在所有 action 调用前调用
const unsubscribe = store.$onAction(({ name, store, args, after, onError }) => {
   // name action 方法名
   // store store 实例
   // args 传递给 action 的参数数组
   // after  在 action 执行完后的回调
   // onError action 抛出错误回调
   console.log(`Action "${name}" was called with args:`, args)
   
   // 在 action 成功完成后调用
   after((result) => {
        console.log(`Action "${name}" finished successfully with result:`, result)
   })
   
   // 在 action 发生错误时调用
   onError((error) => {
        console.error(`Action "${name}" failed with error:`, error)
   })
})
   
// 手动删除监听器
unsubscribe()

// 2. 此订阅器即便在组件卸载之后仍会被保留
someStore.$onAction(callback, true)
   
</script>

```

## 插件示例


## 选项式 API Store 示例

### 1. 定义选项式 store（`stores/counter.js`）
```javascript
// stores/counter.js
import { defineStore } from 'pinia'
// 访问其他store的任意数据
import { useOtherStore } from './other-store'

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
    doubleCount: (state) => {
      // 通过this访问自身任意 state/getters/actions, 进行使用调用
      return state.count * 2;   
    },
  },

  // 定义 actions，用于修改 state 或执行异步操作
  actions: {
    increment(arg) {
      // 通过this访问自身任意 state/getters/actions, 进行使用调用
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
    <p>Count: {{ count }}</p> <!-- 直接访问 store 中的状态 -->
    <p>Double Count: {{ store.doubleCount }}</p> <!-- 通过 getter 访问派生状态 -->
    <button @click="store.increment">Increment</button> <!-- 通过 action 增加 count -->
    <button @click="store.decrement">Decrement</button> <!-- 通过 action 减少 count -->
  </div>
</template>

<script setup>
import { useCounterStore } from './stores/counter'
import { computed } from 'vue'

// 使用 useCounterStore 获取 store 实例
const store = useCounterStore()

// !不能直接使用解构,会失去响应式
// const { name, doubleCount } = store

const count = computed(() => store.count)

// 可直接访问和修改, 修改未被 state() 中被定义的属性是无效的
store.count++

// !赋值会造成丢失响应式
// store.$state = { count: 24 }
// 要调用 `$patch()`：
store.$patch({ count: 24 })

// $patch 修改多个属性,接受对象或函数参数
store.$patch({
   count: store.count + 1,
   age: 120,
   name: 'DIO',
})
// store.$patch((state) => {
//    state.items.push({ name: 'shoes', quantity: 1 })
//    state.hasChanged = true
// })


// $reset() 重置初始值,选项式可以直接调用,组合式store需要自己写代码
store.$reset()

// $subscribe   

</script>
```

## 组合式 API Store 示例

### 1. 定义组合式 store（`stores/counter.js`）

```javascript
// stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
// 访问其他store的任意数据
import { useOtherStore } from './other-store'

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
  
  function $reset() {
     // 重置数据.... 
  }

  // 返回 store 中的所有属性和方法
  return {
    count,
    doubleCount,
    increment,
    decrement,
    reset
  }
})
```

### 2. 在组件中使用组合式 store

```vue
<template>
  <div>
    <p>Count: {{ count }}</p> <!-- 直接访问 store 中的状态 -->
    <p>Double Count: {{ store.doubleCount }}</p> <!-- 通过 getter 访问派生状态 -->
    <button @click="store.increment">Increment</button> <!-- 通过 action 增加 count -->
    <button @click="store.decrement">Decrement</button> <!-- 通过 action 减少 count -->
  </div>
</template>

<script setup>
import { useCounterStore } from './stores/counter'
import { computed } from 'vue'

// 使用 useCounterStore 获取 store 实例
const store = useCounterStore()

// !不能直接使用解构,会失去响应式
// const { name, doubleCount } = store

const count = computed(() => store.count)

// 可直接访问和修改, 修改未被 state() 中被定义的属性是无效的
store.count++

// !赋值会造成丢失响应式
// store.$state = { count: 24 }
// 要调用 `$patch()`：
store.$patch({ count: 24 })

// $patch 修改多个属性,接受对象或函数参数
store.$patch({
   count: store.count + 1,
   age: 120,
   name: 'DIO',
})
// store.$patch((state) => {
//    state.items.push({ name: 'shoes', quantity: 1 })
//    state.hasChanged = true
// })


// $reset() 重置初始值,选项式可以直接调用,组合式store需要自己写代码
store.$reset()

</script>
```

## 两种方式的对比与总结

1. **选项式 store**：
    - 类似vuex,自带$reset()可以重置数据.

2. **组合式 store**：
    - 直接使用 Vue 3 的 `ref`, `reactive`, `computed` 和函数来定义 store，完全遵循组合式 API 的思想。
    - $reset()需要自己定义.

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

    <!-- 直接修改 name -->
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
    
    // 使用 mapWritableState 映射可直接修改的 state（例如 name）, 映射为一个可写的 `computed` 属性
    ...mapWritableState(useCounterStore, ['name']),
  },
  
  methods: {
    // 使用 mapActions 将 actions 映射为方法
    ...mapActions(useCounterStore, ['increment', 'decrement', 'updateState', 'resetStore']),
  },
}
</script>
```
