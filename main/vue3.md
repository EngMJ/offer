# Vue3.5.11 API 参考

### 插值表达式内受限的全局访问
- 模板中的表达式访问到有限的全局对象。会暴露常用内置全局对象，如 Math 和 Date。

- 没有显式包含在列表中的全局对象将不能在模板内表达式中访问，可以在 app.config.globalProperties 全局显式添加。

### script setup
组件中`<script setup>` 中的顶层的导入、声明的变量和函数可在模板中直接使用。

### reactive

声明响应式状态使对象本身具有响应性, 只接受对象类型. reactive() 返回的是一个原始对象的 Proxy，它和原始对象是不相等的.
使用shallowReactive() API 可以选择退出深层响应性.

```js
import { reactive } from 'vue'


const state = reactive({ count: 0 })
console.log(state) // {count: 0}

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

### ref
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

