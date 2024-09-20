# API参考

## 解析 DOM 模板时的注意事项

有些 HTML 元素，诸如 `<ul>`、`<ol>`、`<table>` 和 `<select>`，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 `<li>`、`<tr>` 和 `<option>`，只能出现在其它某些特定的元素内部。

这会导致我们使用这些有约束条件的元素时遇到一些问题。例如：

``` html
<table>
  <blog-post-row></blog-post-row>
</table>
```

这个自定义组件 `<blog-post-row>` 会被作为无效的内容提升到外部，并导致最终渲染结果出错。幸好这个特殊的 `is` attribute 给了我们一个变通的办法：

``` html
<table>
  <tr is="blog-post-row"></tr>
</table>
```

需要注意的是**如果我们从以下来源使用模板的话，这条限制是*不存在*的**：

- 字符串 (例如：`template: '...'`)
- [单文件组件 (`.vue`)](single-file-components.html)
- [`<script type="text/x-template">`](components-edge-cases.html#X-Templates)

到这里，你需要了解的解析 DOM 模板时的注意事项——实际上也是 Vue 的全部*必要内容*，大概就是这些了。恭喜你！接下来还有很多东西要去学习，不过首先，我们推荐你先休息一下，试用一下 Vue，自己随意做些好玩的东西。

如果你感觉已经掌握了这些知识，我们推荐你再回来把完整的组件指南，包括侧边栏中组件深入章节的所有页面读完。

### 传入一个对象的所有 property

如果你想要将一个对象的所有 property 都作为 prop 传入，你可以使用不带参数的 `v-bind` (取代 `v-bind:prop-name`)。例如，对于一个给定的对象 `post`：

``` js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

下面的模板：

``` html
<blog-post v-bind="post"></blog-post>
```

等价于：

``` html
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

> 2.3.0+ 新增

这里的异步组件工厂函数也可以返回一个如下格式的对象：

``` js
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})
```

### inheritAttrs
`inheritAttrs` 用于控制子组件根元素是否自动继承不被显式声明的特性属性（HTML 特性或 `v-bind` 绑定）。
默认情况下，子组件会继承父组件传递的所有属性，并应用到根元素上，而通过 `inheritAttrs: false` 可以阻止这种行为。
当你不希望父组件传递的属性直接绑定到子组件的根元素时，可以使用 `inheritAttrs: false`，然后通过 `$attrs` 手动将这些属性应用到某些具体元素上。
**示例:**

```vue
// 不希望父组件传递的所有属性都应用到根元素，而是通过$attrs手动控制属性的分配
<template>
  <div>
    <label>{{ label }}</label>
    // 在这个例子中，`<input>` 元素手动使用 `$attrs`，这样父组件传递的属性只会绑定到 `<input>` 上，而不会绑定到根元素 `<div>`。
    <input v-bind="$attrs" />
  </div>
</template>

<script>
export default {
  props: ['label'],
  inheritAttrs: false, // 禁止自动继承属性
};
</script>
```
**作用:**
1. **防止不必要的属性传递**：当不希望父组件的所有属性自动传递给子组件的根元素时，使用 `inheritAttrs: false`，避免属性污染。
2. **多根元素组件**：如果一个组件有多个根元素，需要手动分配属性时会非常有用。
3. 注意 inheritAttrs: false 选项**不会影响 style 和 class 的绑定**。
