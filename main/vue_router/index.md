# API 文档

## 创建路由

### 浏览历史模式

+ createWebHistory 浏览器环境
+ createWebHashHistory 浏览器环境 hash模式
+ createMemoryHistory 服务端渲染环境

**参数:**

+ base 字符串基础路径

**示例:**

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

### createRouter

创建 Router 实例

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
    routes,
    scrollBehavior(to, from, savedPosition) { // 路由切换浏览器滚动控制
        return savedPosition || { top: 0 };
    },
    strict: true, // 启用严格模式禁止匹配尾部斜线。例如，路由 /users 将匹配 /users、/users/、甚至 /Users/
    sensitive: true, // 区分大小写
    end: true, // 路由完全匹配
    linkActiveClass: 'active-link', // 自定义全局激活链接类名
    linkExactActiveClass: 'exact-active-link', // 自定义全局精确激活链接类名
    parseQuery: (query) => { // 自定义全局解析查询参数的函数
        return query;
    },
    stringifyQuery: (query) => { // 自定义全局查询参数序列化为字符串的函数
        return query;
    },
});
```

## routes 路由规则

**示例:**

```js

import { createRouter, createWebHistory } from 'vue-router';
import Home from '../views/Home.vue';
import About from '../views/About.vue';
import UserProfile from '../views/UserProfile.vue';
import Parent from '../views/Parent.vue';
import Child from '../views/Child.vue';
import GrandChild from '../views/GrandChild.vue';
import Dashboard from '../views/Dashboard.vue';
import NotFound from '../views/NotFound.vue';
import Settings from '../views/Settings.vue';
import Search from '../views/Search.vue';

const routes = [
    // 1. 基本路由配置
    { path: '/', component: Home },
    
    // 2. 动态路由参数 匹配 /user/任意
    { path: '/user/:id', component: UserProfile },
    // /:orderId -> 仅匹配数字
    { path: '/:orderId(\\d+)', component: UserProfile },

    // 3. 嵌套路由
    {
        path: '/parent',
        component: Parent,
        children: [
            { path: 'child', component: Child },
            {
                path: 'child/grandchild',
                component: GrandChild, // 嵌套的孙子路由
            },
        ],
    },

    // 4. 路由重定向
    { path: '/home', redirect: '/' },
    /*
    // 命名的路由重定向
    { path: '/home', redirect: { name: 'homepage' } }
    // 函数重定向
    {
        path: '/home',
        redirect: to => {
          // 该函数接收目标路由作为参数
          // 相对位置 不以`/`开头或 { path: 'profile'}
          return 'profile'
          // return { path: '/search', query: { q: to.params.searchText } }
        },
    }
    * */

    // 5. 命名路由
    { path: '/about', component: About, name: 'about' },
    /*
    // 使用 name 替代 path 进行导航
    <router-link :to="{ name: 'profile', params: { username: 'erina' } }">
        User profile
    </router-link>
    */

    // 6. 路由守卫
    {
        path: '/dashboard',
        component: Dashboard,
        meta: { requiresAuth: true }, // 需要身份验证
    },

    // 7. 使用懒加载
    {
        path: '/settings',
        component: () => import(/* webpackChunkName: "settings" */ '../views/Settings.vue'),
    },

    // 8. 404 页面处理
    { path: '/:catchAll(.*)', component: NotFound }, // 捕获所有未匹配的路由

    // 9. 区分大小写与禁止尾部/匹配
    {
        path: '/search',
        component: Search,
        sensitive: true, // 区分大小写
        strict: true, // 禁止尾部/匹配
    },
    // 10. 命名视图 渲染对应的具名routerView
    {
        path: '/',
        components: {
            default: Home,
            // LeftSidebar: LeftSidebar 的缩写
            LeftSidebar,
            // 它们与 `<router-view>` 上的 `name` 属性匹配
            RightSidebar,
        },
    },
    /*
    * <router-view class="view left-sidebar" name="LeftSidebar" />
      <router-view class="view main-content" />
      <router-view class="view right-sidebar" name="RightSidebar" />
    * */
    // 11. 别名
    {
        path: '/users',
        component: UsersLayout,
        children: [
            // 为这 3 个 URL 呈现 UserList
            // - /users
            // - /users/list
            // - /people
            { path: '', component: UserList, alias: ['/people', 'list'] },
        ],
    },
    // 12. props
    // 将:id作为props传递给组件
    { path: '/user/:id', component: User, props: true },
    // 控制具名routerView是否接受路由props
    {
        path: '/user/:id',
        components: { default: User, sidebar: Sidebar },
        props: { default: true, sidebar: false }
    },
    // 函数动态控制传递的路由props
    {
        path: '/search',
        component: SearchUser,
        props: route => ({ query: route.query.q })
    }
];

// 创建路由实例
const router = createRouter({
    history: createWebHistory(), // 使用 HTML5 History 模式
    routes, // 路由配置
});

// 示例：全局路由守卫
router.beforeEach((to, from, next) => {
    if (to.meta.requiresAuth && !isAuthenticated) {
        next({ name: 'login' }); // 重定向到登录页面
    } else {
        next(); // 继续导航
    }
});

// 导出路由实例
export default router;

```

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

## START\_LOCATION 

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
