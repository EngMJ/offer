### **1. 实现核心功能**

#### `simple-pinia.js`
这是我们实现的简单 Pinia 类。

```javascript
import { reactive, computed } from 'vue';

// Pinia 核心实现
class Store {
  constructor(options, plugins = []) {
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

    // 插件扩展
    plugins.forEach((plugin) => plugin(this));
  }
}

// 动态注册器
const storeRegistry = {};

export function createStore(options, plugins = []) {
  return new Store(options, plugins);
}

export function useStore(name, storeDefinition) {
  if (!storeRegistry[name]) {
    storeRegistry[name] = storeDefinition();
  }
  return storeRegistry[name];
}
```

---

### **2. 定义 Stores**

#### `stores/counter.js`
这是一个计数器 Store 模块。

```javascript
import { createStore } from '../simple-pinia';

export const useCounterStore = createStore({
  state: () => ({
    count: 0,
  }),
  getters: {
    doubleCount(state) {
      return state.count * 2;
    },
  },
  actions: {
    increment() {
      this.$state.count++;
    },
    asyncIncrement() {
      setTimeout(() => {
        this.$state.count++;
      }, 1000);
    },
  },
});
```

#### `stores/user.js`
这是一个用户 Store 模块。

```javascript
import { createStore } from '../simple-pinia';

export const useUserStore = createStore({
  state: () => ({
    name: '',
    loggedIn: false,
  }),
  actions: {
    login(name) {
      this.$state.name = name;
      this.$state.loggedIn = true;
    },
    logout() {
      this.$state.name = '';
      this.$state.loggedIn = false;
    },
  },
});
```

---

### **3. 添加插件**

#### `plugins/logger.js`
一个简单的日志插件，用于记录状态变化。

```javascript
export const loggerPlugin = (store) => {
  store.commit = (mutation, payload) => {
    console.log(`[Pinia Logger] Mutation: ${mutation}`, payload);
  };
};
```

---

### **4. 在 Vue 应用中使用**

#### `App.vue`
主应用组件，结合多个 Store。

```vue
<template>
  <div>
    <h1>Counter Store</h1>
    <p>Count: {{ counter.$state.count }}</p>
    <p>Double Count: {{ counter.getters.doubleCount }}</p>
    <button @click="counter.actions.increment">Increment</button>
    <button @click="counter.actions.asyncIncrement">Async Increment</button>

    <h1>User Store</h1>
    <p>User: {{ user.$state.name || 'Guest' }}</p>
    <p>Logged In: {{ user.$state.loggedIn ? 'Yes' : 'No' }}</p>
    <button @click="user.actions.login('John Doe')">Login</button>
    <button @click="user.actions.logout">Logout</button>
  </div>
</template>

<script>
import { useCounterStore } from './stores/counter';
import { useUserStore } from './stores/user';

export default {
  setup() {
    const counter = useCounterStore;
    const user = useUserStore;

    return { counter, user };
  },
};
</script>
```

---

#### `main.js`
将插件和 Store 注册到 Vue 应用。

```javascript
import { createApp } from 'vue';
import App from './App.vue';
import { useStore } from './simple-pinia';
import { loggerPlugin } from './plugins/logger';

// 动态注册 Store 示例
const dynamicStore = () =>
  createStore(
    {
      state: () => ({ dynamicValue: 'I am dynamic!' }),
      actions: {
        updateValue(newValue) {
          this.$state.dynamicValue = newValue;
        },
      },
    },
    [loggerPlugin] // 添加日志插件
  );

const app = createApp(App);

// 动态注册 Store
const myStore = useStore('dynamicStore', dynamicStore);
console.log(myStore.$state.dynamicValue); // 输出: I am dynamic!

app.mount('#app');
```

---
