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

### routes 路由规则

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

- 1.导航被触发。
- 2.在失活的组件里调用 beforeRouteLeave 守卫。
```js
// 选项式
<script>
export default {
  beforeRouteLeave(to, from, next) {
    // 在导航离开渲染该组件的对应路由时调用
    // 与 `beforeRouteUpdate` 一样，它可以访问组件实例 `this`
    // 调用next,或返回路由信息或false,false 则停止导航
    return false;
  },
}
</script>
```

```js
// 组合式
<script setup>
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'
import { ref } from 'vue'

// 与 beforeRouteLeave 相同，无法访问 `this`
onBeforeRouteLeave((to, from, next) => {
  const answer = window.confirm(
    'Do you really want to leave? you have unsaved changes!'
  )
  // 调用next,或返回路由信息或false,false 则停止导航
  // 取消导航并停留在同一页面上
  if (!answer) return false
})
</script>

```

- 3.调用全局的 beforeEach 守卫。
```js
// 当一个导航触发时，全局前置守卫按照创建顺序调用
router.beforeEach((to, from, next) => {
  // 调用next,或返回路由信息或false,false 则停止导航
  next()
})

```

- 4.在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
```js
// 选项式
<script>
export default {
    beforeRouteUpdate(to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 `/users/:id`，在 `/users/1` 和 `/users/2` 之间跳转的时候，
    // 由于会渲染同样的 `UserDetails` 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 因为在这种情况发生的时候，组件已经挂载好了，导航守卫可以访问组件实例 `this`
    // 调用next,或返回路由信息或false,false 则停止导航
    next()
    },
}
</script>
```

```js
// 组合式
<script setup>
    import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'
    import { ref } from 'vue'
    
    const userData = ref()

    // 与 beforeRouteUpdate 相同，无法访问 `this`
    onBeforeRouteUpdate(async (to, from, next) => {
        //仅当 id 更改时才获取用户，例如仅 query 或 hash 值已更改
        if (to.params.id !== from.params.id) {
            userData.value = await fetchUser(to.params.id)
        }
        // 调用next,或返回路由信息或false,false 则停止导航
        next()
    })
</script>

```

- 5.在路由配置里调用 beforeEnter。 守卫只在进入路由时触发,不会在 params、query 或 hash 改变时触发.
```js
// 当beforeEnter返回值为false时停止导航,否则需要返回对应路由信息
function removeQueryParams(to) {
    if (Object.keys(to.query).length)
        return { path: to.path, query: {}, hash: to.hash }
}

function removeHash(to) {
    if (to.hash) return { path: to.path, query: to.query, hash: '' }
}

const routes = [
    {
        path: '/users/:id',
        component: UserDetails,
        beforeEnter: [removeQueryParams, removeHash], // 数组方式,按顺序调用
        // 函数方式
        // beforeEnter: (to, from, next) => {
        //     // 调用next,或返回路由信息或false,false 则停止导航
        //     return false
        // },
    }
]

```

- 6.解析异步路由组件。
- 7.在被激活的组件里调用 beforeRouteEnter。 (组合式api中不存在)
```js
// 选项式
<script>
export default {
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被验证前调用
    // 不能获取组件实例 `this` ！
    // 因为当守卫执行时，组件实例还没被创建！
    // 调用next,或返回路由信息或false,false 则停止导航
    next()
  },
}
</script>
```
- 8.调用全局的 beforeResolve 守卫(2.5+)。 解析守卫刚好会在导航被确认之前、所有组件内守卫和异步路由组件被解析之后调用
```js
// router.beforeResolve 是获取数据或执行任何其他操作（如果用户无法进入页面时你希望避免执行的操作）的理想位置
router.beforeResolve(async (to, from, nenxt) => {
  if (to.meta.requiresCamera) {
    try {
      await askForCameraPermission()
    } catch (error) {
      if (error instanceof NotAllowedError) {
        // ... 处理错误，然后取消导航
        return false
      } else {
        // 意料之外的错误，取消导航并把错误传给全局处理器
        throw error
      }
    }
  }
})
```

- 9.导航被确认。
- 10.调用全局的 afterEach 钩子。导航完成后触发,不会接受 next 函数也不会改变导航本身
```js
// 用于分析、更改页面标题、错误处理等
router.afterEach((to, from, failure) => {
  if (!failure) sendToAnalytics(to.fullPath)
})
```

- 11.触发 DOM 更新。
- 12.调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

## useRoute

**示例:**

```vue

<template>
  <div>
    <h1>User Profile</h1>
    <p>User ID: {{ userId }}</p> <!-- 显示用户 ID -->
    <p>Search Query: {{ searchQuery }}</p> <!-- 显示查询参数 -->
    <p>Requires Auth: {{ requiresAuth }}</p> <!-- 显示元信息 -->
    <button @click="goToSettings">Go to Settings</button> <!-- 导航到设置 -->
  </div>
</template>

<script>
import { ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

export default {
  setup() {
    const route = useRoute(); // 获取当前路由
    const router = useRouter(); // 获取路由实例

    console.log('Path:', route.path); // 当前路径
    console.log('Name:', route.name); // 当前路由名称
    console.log('Params:', route.params); // 动态参数
    console.log('Query:', route.query); // 查询参数
    console.log('Hash:', route.hash); // 哈希
    console.log('Meta:', route.meta); // 元信息
    console.log('Matched:', route.matched); // 匹配记录
    console.log('Redirected From:', route.redirectedFrom); // 重定向来源
    console.log('Full Path:', route.fullPath); // 完整路径

    // 监听用户 ID 路由变化
    watch(
      () => route.params.id,
      (newId) => {
        userId.value = newId; // 更新用户 ID
      }
    );

    // 监听查询参数变化
    watch(
      () => route.query.q,
      (newQuery) => {
        searchQuery.value = newQuery; // 更新查询参数
      }
    );

    // 导航到设置页面
    const goToSettings = () => {
      router.push({ path: '/settings' }); // 导航到设置页面
    };

    return {
      userId,
      searchQuery,
      requiresAuth,
      goToSettings,
    };
  },
};
</script>


```

## useRouter

```vue

<template>
  <div>
    <h1>User Profile</h1>
    <button @click="goHome">Go Home</button> <!-- 导航到主页 -->
    <button @click="goToProfile">Go to Profile</button> <!-- 导航到个人资料 -->
    <button @click="replaceToSettings">Replace with Settings</button> <!-- 替换为设置 -->
    <button @click="goBack">Go Back</button> <!-- 后退一页 -->
    <button @click="goForward">Go Forward</button> <!-- 前进一步 -->
    <h2>All Routes</h2>
    <pre>{{ allRoutes }}</pre> <!-- 打印所有路由 -->
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';

export default {
  setup() {
    const router = useRouter(); // 获取路由实例
    const allRoutes = ref(router.getRoutes()); // 获取所有已定义的路由

    // 导航到主页
    const goHome = () => {
      router.push('/'); // 导航到主页
    };

    // 导航到用户个人资料
    const goToProfile = () => {
      router.push({ path: '/user/123', query:{}, params:{}, replace: false }); // 导航到用户 ID 为 123 的个人资料
    };

    // 替换当前路由为设置页面
    const replaceToSettings = () => {
      router.replace('/settings'); // 替换当前路由
    };

    // 后退一页
    const goBack = () => {
      // router.back()  效果同下
      router.go(-1); // 返回上一页
    };

    // 前进一步
    const goForward = () => {
      // router.forward()  效果同下
      router.go(1); // 前进一步
    };
    
    // 添加路由
    router.addRoute({ path: '/about', name: 'about', component: About })
    
    // 通过路由name添加子路由
    router.addRoute('admin', { path: 'settings', component: AdminSettings })
    
    // 删除路由
    router.removeRoute('about')

    // 获取路由规则完整列表
    router.getRoutes()
    
    // 检查路由是否存在,有则返回true,否则false
    router.hasRoute('路由名称')
    
    // isReady 判断初始导航是否准备好
    const navigate = async () => {
      try {
        await router.isReady(); // 等待路由准备好,在服务端渲染中确认服务端和客户端输出一致的时候非常有用
        await router.push({ path: '/user/123' });
      } catch (err) {
          console.log(err)
      }
    };
    // resolve 将输入路由字符串分类转化为对象形式
    const path = '/user/123?q=test'; // 要解析的路径
    const routeInfo = router.resolve(path); // 使用 router.resolve 解析路径
    console.log(info.route); // 输出路由对象
    console.log(info.href); // 输出完整的 URL,根据resolve的第二参数是否输入,没有输出就用当前路由路径拼接
    console.log(info.isResolved); // 输出是否成功解析
    console.log(info.params); // 输出动态路由参数
    console.log(info.query); // 输出查询参数

    // onError 全局错误处理
    onMounted(() => {
      // 在组件挂载时设置全局错误处理器
      // 捕获同步和异步被抛出的错误、在任何导航守卫中返回或传入 next 的错误、解析渲染路由的异步组件时发生的错误
      router.onError((err) => {
        console.log(err)
      });
    });
    
    // 导航守卫
    // 每次导航之前执行。返回一个用来移除该钩子的函数
    const removeBeforeEach = router.beforeEach((to, from, next) => {
      // 可写可不写,不写要返回路由对象进行导航
      next()
    })
    // 每次导航将要被解析之前。返回一个用来移除该钩子的函数
    const removebeforeResolve = router.beforeResolve((to, from, next) => {
      // 可写可不写,不写要返回路由对象进行导航
      next()
    })
    // 每次导航之后被执行。返回一个用来移除该钩子的函数
    const removeAfterEach = router.afterEach((to, from, failure) => {
        // 判断导航失败原因
      if (isNavigationFailure(failure)) {
        console.log('failed navigation', failure)
      }
    })
    
    // currentRoute 当前路由对象,相当于 $route
    console.log(router.currentRoute)
    
    // listening 允许关闭历史事件的监听器。这是一个为微前端提供的底层 API。
    router.listening = false;
    
    // options 返回createRouter的原始创建选项
    console.log(router.options)
    
    return {
      goHome,
      goToProfile,
      replaceToSettings,
      goBack,
      goForward,
      allRoutes,
    };
  },
};
</script>


```

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
