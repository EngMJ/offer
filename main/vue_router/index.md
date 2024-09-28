# API 文档

## 创建路由

## 路由规则

## 组件 

### RouterLink 

链接组件，被渲染成一个a标签

**基本用法:**

```js
<template>
  <div>
    // 默认
    <RouterLink to="/">Home</RouterLink>
    // to对象参数
    <RouterLink :to="{ name: 'user', params: { userId: 123 }, query: { ref: 'navbar' }, replace:false}">User 123</RouterLink>
    // replace
    <RouterLink to="/" replace>Home</RouterLink>
    // 自定义高亮类名
    <RouterLink to="/about" active-class="active-link">About</RouterLink>
    // 自定义准确路径的高亮类名
    <RouterLink to="/about" exactActiveClass="active-link">About</RouterLink>
    // 路由插槽
    <RouterLink to="/about" v-slot="{ isActive, isExactActive }">
      <span :class="{ active: isActive, exact: isExactActive }">About</span>
    </RouterLink>
    // custom 使RouterLink不渲染成a标签,形成自定义组件
    <RouterLink to="/about" custom v-slot="{ navigate, isActive, isExactActive }">
      <button :class="{ active: isActive, exact: isExactActive }" @click="navigate">
        Go to About
      </button>
    </RouterLink>
    // 无障碍值,可选 ariaCurrentValue: "location" | "time" | "page" | "step" | "date" | "true" | "false"
    <RouterLink to="/about" ariaCurrentValue="location">About</RouterLink>
  </div>
</template>
```
___

### RouterView 

路由视图组件

**基本用法:**

```js
// routes.js
// 具名路由渲染规则
  const routes = [
    { path: '/', component: Home, name: 'home' },
    { path: '/about', component: About, name: 'about' },
  ];

// 组件
<template>
  <div>
    // 默认
    <RouterView />
    // 具名
    <RouterView name="home" />
    // v-slot
    <RouterView v-slot="{ Component, Route }">
      <component :is="Component" />
    </RouterView>
    // 过渡/保持
    <transition name="fade">
      <RouterView />
    </transition>
    <RouterView v-slot="{ Component }">
      <keep-alive>
        <component :is="Component" />
      </keep-alive>
    </RouterView>
  </div>
</template>
```


## 导航守卫

**路由导航流程:**

- 导航被触发。
- 在失活的组件里调用 beforeRouteLeave 守卫。
- 调用全局的 beforeEach 守卫。
- 在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
- 在路由配置里调用 beforeEnter。
- 解析异步路由组件。
- 在被激活的组件里调用 beforeRouteEnter。
- 调用全局的 beforeResolve 守卫(2.5+)。
- 导航被确认。
- 调用全局的 afterEach 钩子。
- 触发 DOM 更新。
- 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

## useRoute

## useRouter

## 组合式API中的模板仍可使用 $router 和 $route

## 检测导航故障

如果导航被阻止，导致用户停留在同一个页面上，由 `router.push` 返回的 `Promise` 的解析值将是 _Navigation Failure_。否则，它将是一个 _falsy_ 值(通常是 `undefined`)。这样我们就可以区分我们导航是否离开了当前位置：

```js
const navigationResult = await router.push('/my-profile')

if (navigationResult) {
  // 导航被阻止
} else {
  // 导航成功 (包括重新导航的情况)
  this.isMenuOpen = false
}
```

_Navigation Failure_ 是带有一些额外属性的 `Error` 实例，这些属性为我们提供了足够的信息，让我们知道哪些导航被阻止了以及为什么被阻止了。要检查导航结果的性质，请使用 `isNavigationFailure` 函数：

```js
import { NavigationFailureType, isNavigationFailure } from 'vue-router'

// 试图离开未保存的编辑文本界面
const failure = await router.push('/articles/2')

if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
  // 给用户显示一个小通知
  showToast('You have unsaved changes, discard and leave anyway?')
}
```

- tip 如果你忽略第二个参数： `isNavigationFailure(failure)`，那么就只会检查这个 `failure` 是不是一个 _Navigation Failure_。

全局导航错误处理

```js

router.afterEach((to, from, failure) => {
  if (failure) {
    sendToAnalytics(to, from, failure)
  }
})

```

## 检测重定向

当在导航守卫中返回一个新的位置时，我们会触发一个新的导航，覆盖正在进行的导航。与其他返回值不同的是，重定向不会阻止导航，**而是创建一个新的导航**。因此，通过读取路由地址中的 `redirectedFrom` 属性，对其进行不同的检查：

```js
await router.push('/my-profile')
if (router.currentRoute.value.redirectedFrom) {
  // redirectedFrom 是解析出的路由地址，就像导航守卫中的 to和 from
}
```

### START\_LOCATION 

路由器的初始路由位置。可以在导航守卫中使用来区分初始导航。

**属性:**

+ fullPath 包括 search 和 hash 在内的完整地址。该字符串是经过百分号编码的。
+ path 经过百分号编码的 URL 中的 pathname 段。
+ hash 当前地址的 hash。如果存在则以 # 开头。
+ name 匹配的路由名称。
+ meta 从所有匹配的路由记录中合并的 meta 属性。
+ params 从 path 中提取出来并解码后的参数对象。
+ query 代表当前地址的 search 属性的对象
+ matched RouteLocationMatched 当前路径的路由信息数组
+ redirectedFrom 包含在重定向到当前地址之前，我们最初想访问的地址。

**示例**

```js
import { START_LOCATION } from 'vue-router'
// 可以在导航守卫中使用来区分初始导航
router.beforeEach((to, from) => {
  if (from === START_LOCATION) {
    // 初始导航
  }
})
```








## 函数 

### createMemoryHistory 

▸ **createMemoryHistory**(`base?`): [`RouterHistory`](interfaces/RouterHistory.md)

创建一个基于内存的历史。该历史的主要目的是为了处理服务端渲染。它从一个不存在的特殊位置开始。用户可以通过调用 `router.push` 或 `router.replace` 将该位置替换成起始位置。

#### 参数 

| 名称 | 类型 | 默认值 | 描述 |
| :------ | :------ | :------ | :------ |
| `base` | `string` | `''` | 所有 URL 的基础位置，默认为 '/' |

#### 返回值 

[`RouterHistory`](interfaces/RouterHistory.md)

一个历史对象，可以传递给路由器构造函数。

___

### createRouter 

▸ **createRouter**(`options`): [`Router`](interfaces/Router.md)

创建一个可以被 Vue 应用使用的 Router 实例。

#### 参数 

[RouterOptions](interfaces/RouterOptions.md) |

以下是关于 `createRouter` 可用选项的所有示例：

### 1. `history`

指定路由的历史模式，可以是 `createWebHistory`、`createWebHashHistory` 或 `createMemoryHistory`。

```javascript
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(), // 使用 HTML5 History 模式
  routes,
});
```

### 2. `routes`

定义路由数组，每个路由包含路径和组件。

```javascript
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});
```

### 3. `scrollBehavior`

定义路由导航时的滚动行为。可以返回一个位置对象或 `null`。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    return savedPosition || { top: 0 };
  },
});
```

### 4. `strict`

设置为 `true` 时，启用严格模式以检测未定义的路由。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  strict: true, // 启用严格模式
});
```

### 5. `hash`

如果使用 `createWebHashHistory`，可通过此选项设置哈希选项。

```javascript
import { createRouter, createWebHashHistory } from 'vue-router';

const router = createRouter({
  history: createWebHashHistory(),
  routes,
  hash: true, // 适用于哈希模式
});
```

### 6. `base`

为路由设置基础路径，如果应用部署在子路径下非常有用。

```javascript
const router = createRouter({
  history: createWebHistory('/my-app/'), // 基础路径
  routes,
});
```

### 7. `linkActiveClass`

自定义激活链接的 CSS 类名，默认为 `router-link-active`。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  linkActiveClass: 'active-link', // 自定义激活链接类名
});
```

### 8. `linkExactActiveClass`

自定义精确激活链接的 CSS 类名，默认为 `router-link-exact-active`。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  linkExactActiveClass: 'exact-active-link', // 自定义精确激活链接类名
});
```

### 9. `parseQuery`

自定义解析查询参数的函数。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  parseQuery: (query) => {
    // 自定义解析逻辑
    return query;
  },
});
```

### 10. `stringifyQuery`

自定义将查询参数序列化为字符串的函数。

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  stringifyQuery: (query) => {
    // 自定义序列化逻辑
    return query;
  },
});
```

### 总结

以上是 `createRouter` 的所有选项示例，包括历史模式、路由配置、滚动行为等。如果还有其他主题或具体功能想要了解，请告诉我！

___

### createWebHashHistory 

▸ **createWebHashHistory**(`base?`): [`RouterHistory`](interfaces/RouterHistory.md)

创建一个 hash 模式的历史。在没有主机的 web 应用 (如 `file://`) 或无法通过配置服务器来处理任意 URL 的时候非常有用。

#### 参数 

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `base?` | `string` | 可选提供的基础位置。默认为 `location.pathname + location.search`。如果在 `head` 中有一个 `<base>` 标签，它的值会因此被忽略，**但注意它会影响所有 history.pushState() 的调用**，这意味着如果你使用一个 `<base>` 标签，它的 `href` 值**必须与这个参数匹配** (忽略 `#` 后的任何东西)。 |

#### 返回值 

[`RouterHistory`](interfaces/RouterHistory.md)

**示例**

```js
// 基于 https://example.com/folder
createWebHashHistory() // 给出一个 `https://example.com/folder#` 的 URL
createWebHashHistory('/folder/') // 给出一个 `https://example.com/folder/#` 的 URL
// 如果其基础位置提供了 `#`，则不会被 `createWebHashHistory` 添加
createWebHashHistory('/folder/#/app/') // 给出一个 `https://example.com/folder/#/app/` 的 URL
// 你应该避免这样做，因为它改变了原始的 URL 且破坏了复制 URL 的工作
createWebHashHistory('/other-folder/') // 给出一个 `https://example.com/other-folder/#` 的 URL

// 基于 file:///usr/etc/folder/index.html
// 对于没有 `host` 的位置，该 base 会被忽略
createWebHashHistory('/iAmIgnored') // 给出一个 `file:///usr/etc/folder/index.html#` 的 URL
```

___

### createWebHistory 

▸ **createWebHistory**(`base?`): [`RouterHistory`](interfaces/RouterHistory.md)

创建一个 HTML5 历史。对于单页应用来说这是最常见的历史。

#### 参数 

| 名称 | 类型 |
| :------ | :------ |
| `base?` | `string` |

#### 返回值 

[`RouterHistory`](interfaces/RouterHistory.md)

___

### isNavigationFailure 

▸ **isNavigationFailure**(`error`, `type?`): error is NavigationRedirectError

检查一个对象是否是 [NavigationFailure](interfaces/NavigationFailure.md)。

#### 参数 

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `error` | `any` | 可能的 [NavigationFailure](interfaces/NavigationFailure.md) |
| `type?` | `NAVIGATION_GUARD_REDIRECT` | 可选的待检查类型 |

#### 返回值 

error is NavigationRedirectError

**示例**

```js
import { isNavigationFailure, NavigationFailureType } from 'vue-router'

router.afterEach((to, from, failure) => {
  // 任何类型的导航失败
  if (isNavigationFailure(failure)) {
    // ...
  }
  // 重复的导航
  if (isNavigationFailure(failure, NavigationFailureType.duplicated)) {
    // ...
  }
  // 中止或取消的导航
  if (isNavigationFailure(failure, NavigationFailureType.aborted | NavigationFailureType.canceled)) {
    // ...
  }
})
```

▸ **isNavigationFailure**(`error`, `type?`): error is NavigationFailure

#### 参数 

| 名称 | 类型 |
| :------ | :------ |
| `error` | `any` |
| `type?` | `ErrorTypes` \| [`NavigationFailureType`](enums/NavigationFailureType.md) |

#### 返回值 

error is NavigationFailure

___

### loadRouteLocation 

▸ **loadRouteLocation**(`route`): `Promise`\<[`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)\>

确保路由被加载，所以它可以作为一个 prop 传递给 `<RouterView>`。

#### 参数 

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `route` | [`RouteLocationNormalized`](interfaces/RouteLocationNormalized.md) | 解析要加载的路由 |

#### 返回值 

`Promise`\<[`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)\>

___

### onBeforeRouteLeave 

▸ **onBeforeRouteLeave**(`leaveGuard`): `void`

添加一个导航守卫，不论当前位置的组件何时离开都会触发。类似于 beforeRouteLeave，但可以在任意组件中使用。当组件被卸载时，该守卫会被移除。

#### 参数 

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `leaveGuard` | [`NavigationGuard`](interfaces/NavigationGuard.md) | [NavigationGuard](interfaces/NavigationGuard.md) |

#### 返回值 

`void`

___

### onBeforeRouteUpdate 

▸ **onBeforeRouteUpdate**(`updateGuard`): `void`

添加一个导航守卫，不论当前位置何时被更新都会触发。类似于 beforeRouteUpdate，但可以在任何组件中使用。当组件被卸载时，该守卫会被移除。

#### 参数 

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `updateGuard` | [`NavigationGuard`](interfaces/NavigationGuard.md) | [NavigationGuard](interfaces/NavigationGuard.md) |

#### 返回值 

`void`

___

### useLink 

▸ **useLink**(`props`): `Object`

#### 参数 

| 名称 | 类型 |
| :------ | :------ |
| `props` | `VueUseOptions`\<`RouterLinkOptions`\> |

#### 返回值 

`Object`

| 名称 | 类型 |
| :------ | :------ |
| `href` | `ComputedRef<string\>` |
| `isActive` | `ComputedRef`\<`boolean`\> |
| `isExactActive` | `ComputedRef`\<`boolean`\> |
| `navigate` | (`e`: `MouseEvent`) => `Promise`\<`void` \| [`NavigationFailure`](interfaces/NavigationFailure.md)\> |
| `route` | `ComputedRef`\<[`RouteLocation`](interfaces/RouteLocation.md) & { `href`: `string`  }\> |

___

### useRoute 

▸ **useRoute**(): [`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)

返回当前的路由地址。相当于在模板中使用 `$route`。

#### 返回值 

[`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)

___

### useRouter 

▸ **useRouter**(): [`Router`](interfaces/Router.md)

返回路由器实例。相当于在模板中使用 `$router`。

#### 返回值 

[`Router`](interfaces/Router.md)
