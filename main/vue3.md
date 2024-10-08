# Vue3.5.11 API 参考

## 插值表达式内受限的全局访问
- 模板中的表达式访问到有限的全局对象。会暴露常用内置全局对象，如 Math 和 Date。

- 没有显式包含在列表中的全局对象将不能在模板内表达式中访问，可以在 app.config.globalProperties 全局显式添加。

## script setup
组件中`<script setup>` 中的顶层的导入、声明的变量和函数可在模板中直接使用。

## 响应式值改变触发监听

以下改变可触发监听:
1. 数组 下标修改,push/pop/shift/unshift/splice/sort/reverse,重新赋值
2. 对象 key修改, 重新赋值
3. 原始类型 重新赋值

## reactive

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

## ref
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

## ref解包

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


## computed 计算属性

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

## Class 与 Style 绑定
1. :class 和 :style 都可以接受字符串/对象/数组 值
2. :style 会自动添加浏览器特殊css前缀

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
   <div v-for="item in 10" :key="item">
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

## 生命周期

**流程图:**

![](https://cn.vuejs.org/assets/lifecycle_zh-CN.W0MNXI0C.png)

```vue
<script setup>
import { onBeforeMount,onMounted,onBeforeUpdate,onUpdated,onBeforeUnmount,onUnmounted,onActivated,onDeactivated,onErrorCaptured,onRenderTracked,onRenderTriggered,onServerPrefetch,onServerPrefetch } from 'vue'
// 在组件被挂载之前被调用, DOM节点还未创建
onBeforeMount(()=>{
    // ...
})

// 在组件挂载完成后执行,完成以下两步
// 1. 所有同步子组件都已经被挂载 (不包含异步组件或 <Suspense> 树内的组件)
// 2. 自身的 DOM 树已经创建完成并插入了父容器中
onMounted(() => {
    // 通常用于访问/操作DOM 
})

// 在组件响应式状态变更而更新其 DOM 树之前调用
onBeforeUpdate(()=>{
    // 用来在 Vue 更新 DOM 之前访问 DOM 状态
})

// 在组件的响应式状态变更而更新 DOM 树之后调用
onUpdated(() => {
   // 访问更新后的 DOM
})

// 在组件实例被卸载之前调用,组件实例依然还保有全部的功能
onBeforeUnmount(()=>{
    // ...
})

// 组件实例被卸载之后调用
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

// 仅在开发模式下可用，且在服务器端渲染期间不会被调用
// 当响应式依赖的变更触发了组件渲染时调用
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
