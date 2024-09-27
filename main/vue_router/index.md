
# 特殊注意

## 组合式API中的模板仍可使用 $router 和 $route

## 路由导航流程

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


## 过渡动效/保持活性 使用router-view 插槽

```vue

<router-view v-slot="{ Component }">
  <transition>
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </transition>
</router-view>

```

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

# API 文档

## TS 枚举 

- [NavigationFailureType](enums/NavigationFailureType.md)

# 枚举：NavigationFailureType

为导航失败枚举所有可能的类型。可以传入 [isNavigationFailure](../index.md#isnavigationfailure) 以检查特定的失败情况。

## 枚举成员

### aborted

• **aborted** = ``4``

中断的导航是因为导航守卫返回 `false` 会调用了 `next(false)` 而导致失败的导航。

___

### cancelled

• **cancelled** = ``8``

取消的导航是因为另一个更近的导航已经开始 (不需要完成) 而导致失败的导航。

___

### duplicated

• **duplicated** = ``16``

重复的导航是因为其开始的时候已经处在相同的路径而导致失败的导航。


## TS 接口 

- [HistoryState](interfaces/HistoryState.md)
---
editLink: false
---

[API 参考](../index.md) / HistoryState

# 接口：HistoryState

允许的 HTML history.state

## 可索引成员

▪ [x: `number`]: `HistoryStateValue`

- [NavigationFailure](interfaces/NavigationFailure.md)

---
editLink: false
---

[API 参考](../index.md) / NavigationFailure

# 接口：NavigationFailure

Error 类型的扩展，包含导航失败的额外信息。

## 继承关系

- `Error`

  ↳ **`NavigationFailure`**

## 属性

### cause

• `可选` **cause**: `unknown`

#### 继承自

Error.cause

___

### from

• **from**: [`RouteLocationNormalized`](RouteLocationNormalized.md)

上一个路由位置

___

### message

• **message**: `string`

#### 继承自

Error.message

___

### name

• **name**: `string`

#### 继承自

Error.name

___

### stack

• `可选` **stack**: `string`

#### 继承自

Error.stack

___

### to

• **to**: [`RouteLocationNormalized`](RouteLocationNormalized.md)

要导航至的下一个路由位置

___

### type

• **type**: `NAVIGATION_ABORTED` \| `NAVIGATION_CANCELLED` \| `NAVIGATION_DUPLICATED`

导航类型。属于 [NavigationFailureType](../enums/NavigationFailureType.md) 的一种。


- [NavigationGuard](interfaces/NavigationGuard.md)

---
editLink: false
---

[API 参考](../index.md) / NavigationGuard

# 接口：NavigationGuard

导航守卫。详情可查阅[导航守卫](/zh/guide/advanced/navigation-guards.md)。

## 可调用函数

### NavigationGuard

▸ **NavigationGuard**(`to`, `from`, `next`): `NavigationGuardReturn` \| `Promise`\<`NavigationGuardReturn`\>

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `to` | [`RouteLocationNormalized`](RouteLocationNormalized.md) |
| `from` | [`RouteLocationNormalized`](RouteLocationNormalized.md) |
| `next` | [`NavigationGuardNext`](NavigationGuardNext.md) |

#### 返回值

`NavigationGuardReturn` \| `Promise`\<`NavigationGuardReturn`\>


- [NavigationGuardNext](interfaces/NavigationGuardNext.md)

---
editLink: false
---

[API 参考](../index.md) / NavigationGuardNext

# 接口：NavigationGuardNext

## 可调用函数

### NavigationGuardNext

▸ **NavigationGuardNext**(): `void`

#### 返回值

`void`

### NavigationGuardNext

▸ **NavigationGuardNext**(`error`): `void`

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `error` | `Error` |

#### 返回值

`void`

### NavigationGuardNext

▸ **NavigationGuardNext**(`location`): `void`

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `location` | [`RouteLocationRaw`](../index.md#routelocationraw) |

#### 返回值

`void`

### NavigationGuardNext

▸ **NavigationGuardNext**(`valid`): `void`

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `valid` | `undefined` \| `boolean` |

#### 返回值

`void`

### NavigationGuardNext

▸ **NavigationGuardNext**(`cb`): `void`

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `cb` | `NavigationGuardNextCallback` |

#### 返回值

`void`


- [NavigationGuardWithThis](interfaces/NavigationGuardWithThis.md)

---
editLink: false
---

[API 参考](../index.md) / NavigationGuardWithThis

# 接口：NavigationGuardWithThis\<T\>

导航守卫。详情可查阅[导航守卫](/zh/guide/advanced/navigation-guards.md)。

## 类型参数

| Name |
| :------ |
| `T` |

## 可调用函数

### NavigationGuardWithThis

▸ **NavigationGuardWithThis**(`this`, `to`, `from`, `next`): `NavigationGuardReturn` \| `Promise`\<`NavigationGuardReturn`\>

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `this` | `T` |
| `to` | [`RouteLocationNormalized`](RouteLocationNormalized.md) |
| `from` | [`RouteLocationNormalized`](RouteLocationNormalized.md) |
| `next` | [`NavigationGuardNext`](NavigationGuardNext.md) |

#### 返回值

`NavigationGuardReturn` \| `Promise`\<`NavigationGuardReturn`\>


- [NavigationHookAfter](interfaces/NavigationHookAfter.md)

---
editLink: false
---

[API 参考](../index.md) / NavigationHookAfter

# 接口：NavigationHookAfter

## 可调用函数

### NavigationHookAfter

▸ **NavigationHookAfter**(`to`, `from`, `failure?`): `any`

#### 参数

| 名称 | 类型 |
| :------ | :------ |
| `to` | [`RouteLocationNormalized`](RouteLocationNormalized.md) |
| `from` | [`RouteLocationNormalized`](RouteLocationNormalized.md) |
| `failure?` | `void` \| [`NavigationFailure`](NavigationFailure.md) |

#### 返回值

`any`


- [RouteLocation](interfaces/RouteLocation.md)

---
editLink: false
---

[API 参考](../index.md) / RouteLocation

# 接口：RouteLocation

通过匹配解析出来的 [RouteLocationRaw](../index.md#routelocationraw)

## 继承关系

- `_RouteLocationBase`

  ↳ **`RouteLocation`**

## 属性

### fullPath

• **fullPath**: `string`

包括 `search` 和 `hash` 在内的完整地址。该字符串是经过百分号编码的。

#### 继承自

\_RouteLocationBase.fullPath

___

### hash

• **hash**: `string`

当前地址的 hash。如果存在则以 `#` 开头。

#### 继承自

\_RouteLocationBase.hash

___

### matched

• **matched**: [`RouteRecordNormalized`](RouteRecordNormalized.md)[]

包含添加记录时被传入的 [RouteRecord](../index.md#routerecord) 的数组。它也可以包含重定向记录。不能直接使用。

___

### meta

• **meta**: [`RouteMeta`](RouteMeta.md)

从所有匹配的路由记录中合并的 `meta` 属性。

#### 继承自

\_RouteLocationBase.meta

___

### name

• **name**: `undefined` \| ``null`` \| [`RouteRecordName`](../index.md#routerecordname)

匹配的路由名称。

#### 继承自

\_RouteLocationBase.name

___

### params

• **params**: [`RouteParams`](../index.md#routeparams)

从 `path` 中提取出来并解码后的参数对象。

#### 继承自

\_RouteLocationBase.params

___

### path

• **path**: `string`

经过百分号编码的 URL 中的 pathname 段。

#### 继承自

\_RouteLocationBase.path

___

### query

• **query**: [`LocationQuery`](../index.md#locationquery)

代表当前地址的 `search` 属性的对象

#### 继承自

\_RouteLocationBase.query

___

### redirectedFrom

• **redirectedFrom**: `undefined` \| [`RouteLocation`](RouteLocation.md)

包含在重定向到当前地址之前，我们最初想访问的地址。

#### 继承自

\_RouteLocationBase.redirectedFrom


- [RouteLocationMatched](interfaces/RouteLocationMatched.md)

---
editLink: false
---

[API 参考](../index.md) / RouteLocationMatched

# 接口：RouteLocationMatched

一条[路由记录](../index.md#routerecord)的规范化版本。

## 继承关系

- [`RouteRecordNormalized`](RouteRecordNormalized.md)

  ↳ **`RouteLocationMatched`**

## 属性

### aliasOf

• **aliasOf**: `undefined` \| [`RouteRecordNormalized`](RouteRecordNormalized.md)

定义了是否这条记录是另一条的别名。如果记录是原始记录，则该属性为 `undefined`。

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[aliasOf](RouteRecordNormalized.md#aliasof)

___

### beforeEnter

• **beforeEnter**: `undefined` \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

被注册的 beforeEnter 守卫

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[beforeEnter](RouteRecordNormalized.md#beforeenter)

___

### children

• **children**: [`RouteRecordRaw`](../index.md#routerecordraw)[]

嵌套的路由记录。

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[children](RouteRecordNormalized.md#children)

___

### components

• **components**: `undefined` \| ``null`` \| `Record`\<`string`, [`RouteComponent`](../index.md#routecomponent)\>

{@inheritDoc RouteRecordMultipleViews.components}

#### Override

[RouteRecordNormalized](RouteRecordNormalized.md).[components](RouteRecordNormalized.md#components)

___

### instances

• **instances**: `Record`\<`string`, `undefined` \| ``null`` \| `ComponentPublicInstance`\>

<!-- TODO: translation -->

Mounted route component instance。 Having the instances on the record mean beforeRouteUpdate and beforeRouteLeave guards can only be invoked with the latest mounted app instance if there are multiple application instances rendering the same view, basically duplicating the content on the page, which shouldn't happen in practice. It will work if multiple apps are rendering different named views.

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[instances](RouteRecordNormalized.md#instances)

___

### meta

• **meta**: [`RouteMeta`](RouteMeta.md)

<!-- TODO: translation -->

Arbitrary data attached to the record.

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[meta](RouteRecordNormalized.md#meta)

___

### name

• **name**: `undefined` \| [`RouteRecordName`](../index.md#routerecordname)

<!-- TODO: translation -->

Name for the route record. Must be unique.

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[name](RouteRecordNormalized.md#name)

___

### path

• **path**: `string`

<!-- TODO: translation -->

Path of the record. Should start with `/` unless the record is the child of another record.

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[path](RouteRecordNormalized.md#path)

___

### props

• **props**: `Record`\<`string`, `_RouteRecordProps`\>

<!-- TODO: translation -->

Allow passing down params as props to the component rendered by `router-view`. Should be an object with the same keys as `components` or a boolean to be applied to every component.

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[props](RouteRecordNormalized.md#props)

___

### redirect

• **redirect**: `undefined` \| `RouteRecordRedirectOption`

<!-- TODO: translation -->

Where to redirect if the route is directly matched. The redirection happens before any navigation guard and triggers a new navigation with the new target location.

#### 继承自

[RouteRecordNormalized](RouteRecordNormalized.md).[redirect](RouteRecordNormalized.md#redirect)


- [RouteLocationNormalized](interfaces/RouteLocationNormalized.md)

---
editLink: false
---

[API 参考](../index.md) / RouteLocationNormalized

# 接口：RouteLocationNormalized

和 [RouteLocation](RouteLocation.md) 类似但是其 [RouteLocationNormalized.matched](RouteLocationNormalized.md#matched) 无法包含重定向的记录

## 继承关系

- `_RouteLocationBase`

  ↳ **`RouteLocationNormalized`**

## 属性

### fullPath

• **fullPath**: `string`

包括 `search` 和 `hash` 在内的完整地址。该字符串是经过百分号编码的。

#### 继承自

\_RouteLocationBase.fullPath

___

### hash

• **hash**: `string`

当前地址的 hash。如果存在则以 `#` 开头。

#### 继承自

\_RouteLocationBase.hash

___

### matched

• **matched**: [`RouteRecordNormalized`](RouteRecordNormalized.md)[]

[RouteRecordNormalized](RouteRecordNormalized.md) 数组。

___

### meta

• **meta**: [`RouteMeta`](RouteMeta.md)

从所有匹配的路由记录中合并的 `meta` 属性。

#### 继承自

\_RouteLocationBase.meta

___

### name

• **name**: `undefined` \| ``null`` \| [`RouteRecordName`](../index.md#routerecordname)

匹配的路由名称。

#### 继承自

\_RouteLocationBase.name

___

### params

• **params**: [`RouteParams`](../index.md#routeparams)

从 `path` 中提取出来并解码后的参数对象。

#### 继承自

\_RouteLocationBase.params

___

### path

• **path**: `string`

经过百分号编码的 URL 中的 pathname 段。

#### 继承自

\_RouteLocationBase.path

___

### query

• **query**: [`LocationQuery`](../index.md#locationquery)

代表当前地址的 `search` 属性的对象

#### 继承自

\_RouteLocationBase.query

___

### redirectedFrom

• **redirectedFrom**: `undefined` \| [`RouteLocation`](RouteLocation.md)

包含在重定向到当前地址之前，我们最初想访问的地址。

#### 继承自

\_RouteLocationBase.redirectedFrom


- [RouteLocationNormalizedLoaded](interfaces/RouteLocationNormalizedLoaded.md)

---
editLink: false
---

[API 参考](../index.md) / RouteLocationNormalizedLoaded

# 接口：RouteLocationNormalizedLoaded

<!-- TODO: translation -->

[RouteLocationRaw](../index.md#routelocationraw) with

## 继承关系

- `_RouteLocationBase`

  ↳ **`RouteLocationNormalizedLoaded`**

## 属性

### fullPath

• **fullPath**: `string`

包括 `search` 和 `hash` 在内的完整地址。该字符串是经过百分号编码的。

#### 继承自

\_RouteLocationBase.fullPath

___

### hash

• **hash**: `string`

当前地址的 hash。如果存在则以 `#` 开头。

#### 继承自

\_RouteLocationBase.hash

___

### matched

• **matched**: [`RouteLocationMatched`](RouteLocationMatched.md)[]

[RouteLocationMatched](RouteLocationMatched.md) 数组，只包含直接的组件 (任何已被加载并在 `components` 对象内被替换掉的懒加载组件)。所以它可以被直接用于展示路由。同样它不包含重定向的记录。

___

### meta

• **meta**: [`RouteMeta`](RouteMeta.md)

从所有匹配的路由记录中合并的 `meta` 属性。

#### 继承自

\_RouteLocationBase.meta

___

### name

• **name**: `undefined` \| ``null`` \| [`RouteRecordName`](../index.md#routerecordname)

匹配的路由名称。

#### 继承自

\_RouteLocationBase.name

___

### params

• **params**: [`RouteParams`](../index.md#routeparams)

从 `path` 中提取出来并解码后的参数对象。

#### 继承自

\_RouteLocationBase.params

___

### path

• **path**: `string`

经过百分号编码的 URL 中的 pathname 段。

#### 继承自

\_RouteLocationBase.path

___

### query

• **query**: [`LocationQuery`](../index.md#locationquery)

代表当前地址的 `search` 属性的对象

#### 继承自

\_RouteLocationBase.query

___

### redirectedFrom

• **redirectedFrom**: `undefined` \| [`RouteLocation`](RouteLocation.md)

包含在重定向到当前地址之前，我们最初想访问的地址。

#### 继承自

\_RouteLocationBase.redirectedFrom


- [RouteLocationOptions](interfaces/RouteLocationOptions.md)

---
editLink: false
---

[API 参考](../index.md) / RouteLocationOptions

# 接口：RouteLocationOptions

对所有导航方法通用的选项。

## 属性

### force

• `可选` **force**: `boolean`

触发导航，即使该地址与当前地址相同。请注意，这也会新添加一条历史记录，除非传入 `replace: true`。

___

### replace

• `可选` **replace**: `boolean`

替换而不是加入一个新的历史记录。

___

### state

• `可选` **state**: [`HistoryState`](HistoryState.md)

使用 History API 保存的状态。它不能包含任何响应性的值，同时一些诸如 Symbol 的基础类型是被禁用的。更多信息见 https://developer.mozilla.org/en-US/docs/Web/API/History/state


- [RouteMeta](interfaces/RouteMeta.md)

---
editLink: false
---

[API 参考](../index.md) / RouteMeta

# 接口：RouteMeta

路由记录中的 `meta` 字段的类型接口。

**`Example`**

```ts
// typings.d.ts 或 router.ts
import 'vue-router';

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
  }
 }
```

## 继承关系

- `Record`\<`string` \| `number` \| `symbol`, `unknown`\>

  ↳ **`RouteMeta`**


- [RouteRecordMultipleViews](interfaces/RouteRecordMultipleViews.md)

---
editLink: false
---

<!-- TODO: translation -->

[API Documentation](../index.md) / RouteRecordMultipleViews

# Interface: RouteRecordMultipleViews

Route Record defining multiple named components with the `components` option.

## Hierarchy

- [`_RouteRecordBase`](RouteRecordBase.md)

  ↳ **`RouteRecordMultipleViews`**

## Properties

### alias

• `Optional` **alias**: `string` \| `string`[]

Aliases for the record. Allows defining extra paths that will behave like a
copy of the record. Allows having paths shorthands like `/users/:id` and
`/u/:id`. All `alias` and `path` values must share the same params.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[alias](RouteRecordBase.md#alias)

___

### beforeEnter

• `Optional` **beforeEnter**: [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

Before Enter guard specific to this record. Note `beforeEnter` has no
effect if the record has a `redirect` property.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[beforeEnter](RouteRecordBase.md#beforeEnter)

___

### children

• `Optional` **children**: `undefined`

Array of nested routes.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[children](RouteRecordBase.md#children)

___

### component

• `Optional` **component**: `undefined`

___

### components

• **components**: `Record`\<`string`, `RawRouteComponent`\>

Components to display when the URL matches this route. Allow using named views.

___

### end

• `Optional` **end**: `boolean`

Should the RegExp match until the end by appending a `$` to it.

**`Default Value`**

`true`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[end](RouteRecordBase.md#end)

___

### meta

• `Optional` **meta**: [`RouteMeta`](RouteMeta.md)

Arbitrary data attached to the record.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[meta](RouteRecordBase.md#meta)

___

### name

• `Optional` **name**: [`RouteRecordName`](../index.md#RouteRecordName)

Name for the route record. Must be unique.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[name](RouteRecordBase.md#name)

___

### path

• **path**: `string`

Path of the record. Should start with `/` unless the record is the child of
another record.

**`Example`**

```ts
`/users/:id` matches `/users/1` as well as `/users/posva`.
```

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[path](RouteRecordBase.md#path)

___

### props

• `Optional` **props**: `boolean` \| `Record`\<`string`, `_RouteRecordProps`\>

Allow passing down params as props to the component rendered by
`router-view`. Should be an object with the same keys as `components` or a
boolean to be applied to every component.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[props](RouteRecordBase.md#props)

___

### redirect

• `Optional` **redirect**: `undefined`

Where to redirect if the route is directly matched. The redirection happens
before any navigation guard and triggers a new navigation with the new
target location.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[redirect](RouteRecordBase.md#redirect)

___

### sensitive

• `Optional` **sensitive**: `boolean`

Makes the RegExp case-sensitive.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[sensitive](RouteRecordBase.md#sensitive)

___

### strict

• `Optional` **strict**: `boolean`

Whether to disallow a trailing slash or not.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[strict](RouteRecordBase.md#strict)


- [RouteRecordMultipleViewsWithChildren](interfaces/RouteRecordMultipleViewsWithChildren.md)

---
editLink: false
---

<!-- TODO: translation -->

[API Documentation](../index.md) / RouteRecordMultipleViewsWithChildren

# Interface: RouteRecordMultipleViewsWithChildren

Route Record defining multiple named components with the `components` option and children.

## Hierarchy

- [`_RouteRecordBase`](RouteRecordBase.md)

  ↳ **`RouteRecordMultipleViewsWithChildren`**

## Properties

### alias

• `Optional` **alias**: `string` \| `string`[]

Aliases for the record. Allows defining extra paths that will behave like a
copy of the record. Allows having paths shorthands like `/users/:id` and
`/u/:id`. All `alias` and `path` values must share the same params.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[alias](RouteRecordBase.md#alias)

___

### beforeEnter

• `Optional` **beforeEnter**: [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

Before Enter guard specific to this record. Note `beforeEnter` has no
effect if the record has a `redirect` property.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[beforeEnter](RouteRecordBase.md#beforeEnter)

___

### children

• **children**: [`RouteRecordRaw`](../index.md#RouteRecordRaw)[]

Array of nested routes.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[children](RouteRecordBase.md#children)

___

### component

• `Optional` **component**: `undefined`

___

### components

• `Optional` **components**: ``null`` \| `Record`\<`string`, `RawRouteComponent`\>

Components to display when the URL matches this route. Allow using named views.

___

### end

• `Optional` **end**: `boolean`

Should the RegExp match until the end by appending a `$` to it.

**`Default Value`**

`true`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[end](RouteRecordBase.md#end)

___

### meta

• `Optional` **meta**: [`RouteMeta`](RouteMeta.md)

Arbitrary data attached to the record.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[meta](RouteRecordBase.md#meta)

___

### name

• `Optional` **name**: [`RouteRecordName`](../index.md#RouteRecordName)

Name for the route record. Must be unique.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[name](RouteRecordBase.md#name)

___

### path

• **path**: `string`

Path of the record. Should start with `/` unless the record is the child of
another record.

**`Example`**

```ts
`/users/:id` matches `/users/1` as well as `/users/posva`.
```

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[path](RouteRecordBase.md#path)

___

### props

• `Optional` **props**: `boolean` \| `Record`\<`string`, `_RouteRecordProps`\>

Allow passing down params as props to the component rendered by
`router-view`. Should be an object with the same keys as `components` or a
boolean to be applied to every component.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[props](RouteRecordBase.md#props)

___

### redirect

• `Optional` **redirect**: `RouteRecordRedirectOption`

Where to redirect if the route is directly matched. The redirection happens
before any navigation guard and triggers a new navigation with the new
target location.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[redirect](RouteRecordBase.md#redirect)

___

### sensitive

• `Optional` **sensitive**: `boolean`

Makes the RegExp case-sensitive.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[sensitive](RouteRecordBase.md#sensitive)

___

### strict

• `Optional` **strict**: `boolean`

Whether to disallow a trailing slash or not.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[strict](RouteRecordBase.md#strict)


- [RouteRecordNormalized](interfaces/RouteRecordNormalized.md)

---
editLink: false
---

[API 参考](../index.md) / RouteRecordNormalized

# 接口：RouteRecordNormalized

一条[路由记录](../index.md#routerecord)的规范化版本。

## 继承关系

- **`RouteRecordNormalized`**

  ↳ [`RouteLocationMatched`](RouteLocationMatched.md)

## 属性

### aliasOf

• **aliasOf**: `undefined` \| [`RouteRecordNormalized`](RouteRecordNormalized.md)

定义了是否这条记录是另一条的别名。如果记录是原始记录，则该属性为 `undefined`。

___

### beforeEnter

• **beforeEnter**: `undefined` \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

被注册的 beforeEnter 守卫

___

### children

• **children**: [`RouteRecordRaw`](../index.md#routerecordraw)[]

嵌套的路由记录。

___

### components

• **components**: `undefined` \| ``null`` \| `Record`\<`string`, `RawRouteComponent`\>

当 URL 匹配到该路由时显示的组件。允许使用命名视图。

___

### instances

• **instances**: `Record`\<`string`, `undefined` \| ``null`` \| `ComponentPublicInstance`\>

挂载的路由组件实例。
在记录上存在实例意味着，当有多个应用实例渲染相同的视图时，beforeRouteUpdate 和 beforeRouteLeave 守卫只能被最后挂载的应用实例调用。这样的渲染基本上只会对页面内容进行复制，在实际情况下并不应该发生。它可以在多个应用渲染不同的命名视图时工作。

___

### meta

• **meta**: [`RouteMeta`](RouteMeta.md)

附加在记录上的任意数据。

___

### name

• **name**: `undefined` \| [`RouteRecordName`](../index.md#routerecordname)

路由记录的名称。必须唯一。

___

### path

• **path**: `string`

记录的路径。应该以 `/` 开头，除非该记录为另一条记录的子记录。

___

### props

• **props**: `Record`\<`string`, `_RouteRecordProps`\>

允许将参数作为 props 传递给由 `router-view` 渲染的组件。应是一个具有与 `components` 相同键的对象，或是一个应用于所有组件的布尔值。

___

### redirect

• **redirect**: `undefined` \| `RouteRecordRedirectOption`

路由直接匹配时重定向的位置。重定向发生在任何导航守卫和带有新目标位置的新导航触发之前。


- [RouteRecordRedirect](interfaces/RouteRecordRedirect.md)

---
editLink: false
---

<!-- TODO: translation -->

[API Documentation](../index.md) / RouteRecordRedirect

# Interface: RouteRecordRedirect

Route Record that defines a redirect. Cannot have `component` or `components`
as it is never rendered.

## Hierarchy

- [`_RouteRecordBase`](RouteRecordBase.md)

  ↳ **`RouteRecordRedirect`**

## Properties

### alias

• `Optional` **alias**: `string` \| `string`[]

Aliases for the record. Allows defining extra paths that will behave like a
copy of the record. Allows having paths shorthands like `/users/:id` and
`/u/:id`. All `alias` and `path` values must share the same params.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[alias](RouteRecordBase.md#alias)

___

### beforeEnter

• `Optional` **beforeEnter**: [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

Before Enter guard specific to this record. Note `beforeEnter` has no
effect if the record has a `redirect` property.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[beforeEnter](RouteRecordBase.md#beforeEnter)

___

### children

• `Optional` **children**: [`RouteRecordRaw`](../index.md#RouteRecordRaw)[]

Array of nested routes.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[children](RouteRecordBase.md#children)

___

### component

• `Optional` **component**: `undefined`

___

### components

• `Optional` **components**: `undefined`

___

### end

• `Optional` **end**: `boolean`

Should the RegExp match until the end by appending a `$` to it.

**`Default Value`**

`true`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[end](RouteRecordBase.md#end)

___

### meta

• `Optional` **meta**: [`RouteMeta`](RouteMeta.md)

Arbitrary data attached to the record.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[meta](RouteRecordBase.md#meta)

___

### name

• `Optional` **name**: [`RouteRecordName`](../index.md#RouteRecordName)

Name for the route record. Must be unique.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[name](RouteRecordBase.md#name)

___

### path

• **path**: `string`

Path of the record. Should start with `/` unless the record is the child of
another record.

**`Example`**

```ts
`/users/:id` matches `/users/1` as well as `/users/posva`.
```

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[path](RouteRecordBase.md#path)

___

### props

• `Optional` **props**: `undefined`

Allow passing down params as props to the component rendered by `router-view`.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[props](RouteRecordBase.md#props)

___

### redirect

• **redirect**: `RouteRecordRedirectOption`

Where to redirect if the route is directly matched. The redirection happens
before any navigation guard and triggers a new navigation with the new
target location.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[redirect](RouteRecordBase.md#redirect)

___

### sensitive

• `Optional` **sensitive**: `boolean`

Makes the RegExp case-sensitive.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[sensitive](RouteRecordBase.md#sensitive)

___

### strict

• `Optional` **strict**: `boolean`

Whether to disallow a trailing slash or not.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[strict](RouteRecordBase.md#strict)


- [RouteRecordSingleView](interfaces/RouteRecordSingleView.md)

---
editLink: false
---

<!-- TODO: translation -->

[API Documentation](../index.md) / RouteRecordSingleView

# Interface: RouteRecordSingleView

Route Record defining one single component with the `component` option.

## Hierarchy

- [`_RouteRecordBase`](RouteRecordBase.md)

  ↳ **`RouteRecordSingleView`**

## Properties

### alias

• `Optional` **alias**: `string` \| `string`[]

Aliases for the record. Allows defining extra paths that will behave like a
copy of the record. Allows having paths shorthands like `/users/:id` and
`/u/:id`. All `alias` and `path` values must share the same params.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[alias](RouteRecordBase.md#alias)

___

### beforeEnter

• `Optional` **beforeEnter**: [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

Before Enter guard specific to this record. Note `beforeEnter` has no
effect if the record has a `redirect` property.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[beforeEnter](RouteRecordBase.md#beforeEnter)

___

### children

• `Optional` **children**: `undefined`

Array of nested routes.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[children](RouteRecordBase.md#children)

___

### component

• **component**: `RawRouteComponent`

Component to display when the URL matches this route.

___

### components

• `Optional` **components**: `undefined`

___

### end

• `Optional` **end**: `boolean`

Should the RegExp match until the end by appending a `$` to it.

**`Default Value`**

`true`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[end](RouteRecordBase.md#end)

___

### meta

• `Optional` **meta**: [`RouteMeta`](RouteMeta.md)

Arbitrary data attached to the record.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[meta](RouteRecordBase.md#meta)

___

### name

• `Optional` **name**: [`RouteRecordName`](../index.md#RouteRecordName)

Name for the route record. Must be unique.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[name](RouteRecordBase.md#name)

___

### path

• **path**: `string`

Path of the record. Should start with `/` unless the record is the child of
another record.

**`Example`**

```ts
`/users/:id` matches `/users/1` as well as `/users/posva`.
```

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[path](RouteRecordBase.md#path)

___

### props

• `Optional` **props**: `_RouteRecordProps`

Allow passing down params as props to the component rendered by `router-view`.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[props](RouteRecordBase.md#props)

___

### redirect

• `Optional` **redirect**: `undefined`

Where to redirect if the route is directly matched. The redirection happens
before any navigation guard and triggers a new navigation with the new
target location.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[redirect](RouteRecordBase.md#redirect)

___

### sensitive

• `Optional` **sensitive**: `boolean`

Makes the RegExp case-sensitive.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[sensitive](RouteRecordBase.md#sensitive)

___

### strict

• `Optional` **strict**: `boolean`

Whether to disallow a trailing slash or not.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[strict](RouteRecordBase.md#strict)


- [RouteRecordSingleViewWithChildren](interfaces/RouteRecordSingleViewWithChildren.md)

---
editLink: false
---

<!-- TODO: translation -->

[API Documentation](../index.md) / RouteRecordSingleViewWithChildren

# Interface: RouteRecordSingleViewWithChildren

Route Record defining one single component with a nested view.

## Hierarchy

- [`_RouteRecordBase`](RouteRecordBase.md)

  ↳ **`RouteRecordSingleViewWithChildren`**

## Properties

### alias

• `Optional` **alias**: `string` \| `string`[]

Aliases for the record. Allows defining extra paths that will behave like a
copy of the record. Allows having paths shorthands like `/users/:id` and
`/u/:id`. All `alias` and `path` values must share the same params.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[alias](RouteRecordBase.md#alias)

___

### beforeEnter

• `Optional` **beforeEnter**: [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

Before Enter guard specific to this record. Note `beforeEnter` has no
effect if the record has a `redirect` property.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[beforeEnter](RouteRecordBase.md#beforeEnter)

___

### children

• **children**: [`RouteRecordRaw`](../index.md#RouteRecordRaw)[]

Array of nested routes.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[children](RouteRecordBase.md#children)

___

### component

• `Optional` **component**: ``null`` \| `RawRouteComponent`

Component to display when the URL matches this route.

___

### components

• `Optional` **components**: `undefined`

___

### end

• `Optional` **end**: `boolean`

Should the RegExp match until the end by appending a `$` to it.

**`Default Value`**

`true`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[end](RouteRecordBase.md#end)

___

### meta

• `Optional` **meta**: [`RouteMeta`](RouteMeta.md)

Arbitrary data attached to the record.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[meta](RouteRecordBase.md#meta)

___

### name

• `Optional` **name**: [`RouteRecordName`](../index.md#RouteRecordName)

Name for the route record. Must be unique.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[name](RouteRecordBase.md#name)

___

### path

• **path**: `string`

Path of the record. Should start with `/` unless the record is the child of
another record.

**`Example`**

```ts
`/users/:id` matches `/users/1` as well as `/users/posva`.
```

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[path](RouteRecordBase.md#path)

___

### props

• `Optional` **props**: `_RouteRecordProps`

Allow passing down params as props to the component rendered by `router-view`.

#### Overrides

[_RouteRecordBase](RouteRecordBase.md).[props](RouteRecordBase.md#props)

___

### redirect

• `Optional` **redirect**: `RouteRecordRedirectOption`

Where to redirect if the route is directly matched. The redirection happens
before any navigation guard and triggers a new navigation with the new
target location.

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[redirect](RouteRecordBase.md#redirect)

___

### sensitive

• `Optional` **sensitive**: `boolean`

Makes the RegExp case-sensitive.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[sensitive](RouteRecordBase.md#sensitive)

___

### strict

• `Optional` **strict**: `boolean`

Whether to disallow a trailing slash or not.

**`Default Value`**

`false`

#### Inherited from

[_RouteRecordBase](RouteRecordBase.md).[strict](RouteRecordBase.md#strict)

- [Router](interfaces/Router.md)

---
editLink: false
---

<!-- TODO: translation -->

[API 参考](../index.md) / Router

# 接口：Router

路由器实例。

## 属性

### currentRoute

• `只读` **currentRoute**: `Ref`\<[`RouteLocationNormalizedLoaded`](RouteLocationNormalizedLoaded.md)\>

当前的 [RouteLocationNormalized](RouteLocationNormalized.md)。

___

### listening

• **listening**: `boolean`

允许关闭历史事件的监听器。这是一个为微前端提供的底层 API。

___

### options

• `只读` **options**: [`RouterOptions`](RouterOptions.md)

创建路由器时的原始选项对象。

## Methods

### addRoute

▸ **addRoute**(`parentName`, `route`): () => `void`

添加一个新的[路由记录](../index.md#routerecordraw)，将其作为一个已有路由的子路由。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `parentName` | [`RouteRecordName`](../index.md#routerecordname) | `route` 应该被加入到的父级路由记录 |
| `route` | [`RouteRecordRaw`](../index.md#routerecordraw) | 要加入的路由记录 |

#### 返回值

`fn`

▸ (): `void`

添加一个新的[路由记录](../index.md#routerecordraw)，将其作为一个已有路由的子路由。

##### 返回值

`void`

▸ **addRoute**(`route`): () => `void`

添加一个新的[路由记录](../index.md#routerecordraw)到该路由器中。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `route` | [`RouteRecordRaw`](../index.md#routerecordraw) | 要加入的路由记录 |

#### 返回值

`fn`

▸ (): `void`

添加一个新的[路由记录](../index.md#routerecordraw)到该路由器中。

##### 返回值

`void`

___

### afterEach

▸ **afterEach**(`guard`): () => `void`

添加一个导航钩子，它会在每次导航之后被执行。返回一个用来移除该钩子的函数。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `guard` | [`NavigationHookAfter`](NavigationHookAfter.md) | 要加入的导航钩子 |

#### 返回值

`fn`

a function that removes the registered hook

▸ (): `void`

添加一个导航钩子，它会在每次导航之后被执行。返回一个用来移除该钩子的函数。

**`Example`**

```js
router.afterEach((to, from, failure) => {
  if (isNavigationFailure(failure)) {
    console.log('failed navigation', failure)
  }
})
```

##### 返回值

`void`

___

### back

▸ **back**(): `void`

通过调用 `history.back()` 在可能的情况下在历史中后退。相当于 `router.go(-1)`。

#### 返回值

`void`

___

### beforeEach

▸ **beforeEach**(`guard`): () => `void`

添加一个导航钩子，它会在每次导航之前被执行。返回一个用来移除该钩子的函数。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `guard` | [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> | 要加入的导航钩子 |

#### 返回值

`fn`

▸ (): `void`

添加一个导航钩子，它会在每次导航之前被执行。返回一个用来移除该钩子的函数。

##### 返回值

`void`

___

### beforeResolve

▸ **beforeResolve**(`guard`): () => `void`

添加一个导航守卫，它会在导航将要被解析之前被执行。此时所有组件都已经获取完毕，且其它导航守卫也都已经完成调用。返回一个用来移除该守卫的函数。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `guard` | [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> | navigation guard to add |

#### 返回值

`fn`

▸ (): `void`

添加一个导航守卫，它会在导航将要被解析之前被执行。此时所有组件都已经获取完毕，且其它导航守卫也都已经完成调用。返回一个用来移除该守卫的函数。

**`Example`**

```js
router.beforeResolve(to => {
  if (to.meta.requiresAuth && !isAuthenticated) return false
})
```

##### 返回值

`void`

___

### forward

▸ **forward**(): `void`

通过调用 `history.forward()` 在可能的情况下在历史中前进。相当于 `router.go(1)`。

#### 返回值

`void`

___

### getRoutes

▸ **getRoutes**(): [`RouteRecordNormalized`](RouteRecordNormalized.md)[]

获得所有[路由记录](../index.md#routerecord)的完整列表。

#### 返回值

[`RouteRecordNormalized`](RouteRecordNormalized.md)[]

___

### go

▸ **go**(`delta`): `void`

允许你在历史中前进或后退。相当于 `router.go()`。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `delta` | `number` | 相对于当前页面你想要移动到的历史中的位置 |

#### 返回值

`void`

___

### hasRoute

▸ **hasRoute**(`name`): `boolean`

检查一个给定名称的路由是否存在。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `name` | [`RouteRecordName`](../index.md#routerecordname) | 要检查的路由名称 |

#### 返回值

`boolean`

___

### isReady

▸ **isReady**(): `Promise`\<`void`\>

返回一个 Promise，它会在路由器完成初始导航之后被解析，也就是说这时所有和初始路由有关联的异步入口钩子和异步组件都已经被解析。如果初始导航已经发生，则该 Promise 会被立刻解析。

这在服务端渲染中确认服务端和客户端输出一致的时候非常有用。注意在服务端你需要手动加入初始地址，而在客户端，路由器会从 URL 中自动获取。

#### 返回值

`Promise`\<`void`\>

___

### onError

▸ **onError**(`handler`): () => `void`

添加一个错误处理器，它会在每次导航遇到未被捕获的错误出现时被调用。其中包括同步和异步被抛出的错误、在任何导航守卫中返回或传入 `next` 的错误、尝试解析一个需要渲染路由的异步组件时发生的错误。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `handler` | `_ErrorListener` | 要注册的错误处理器 |

#### 返回值

`fn`

▸ (): `void`

添加一个错误处理器，它会在每次导航遇到未被捕获的错误出现时被调用。其中包括同步和异步被抛出的错误、在任何导航守卫中返回或传入 `next` 的错误、尝试解析一个需要渲染路由的异步组件时发生的错误。

##### 返回值

`void`

___

### push

▸ **push**(`to`): `Promise`\<`undefined` \| `void` \| [`NavigationFailure`](NavigationFailure.md)\>

程序式地通过将一条记录加入到历史栈中来导航到一个新的 URL。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `to` | [`RouteLocationRaw`](../index.md#routelocationraw) | 要导航到的路由 |

#### 返回值

`Promise`\<`undefined` \| `void` \| [`NavigationFailure`](NavigationFailure.md)\>

___

### removeRoute

▸ **removeRoute**(`name`): `void`

根据其名称移除一个现有的路由。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `name` | [`RouteRecordName`](../index.md#routerecordname) | 要移除的路由名称 |

#### 返回值

`void`

___

### replace

▸ **replace**(`to`): `Promise`\<`undefined` \| `void` \| [`NavigationFailure`](NavigationFailure.md)\>

程序式地通过替换历史栈中的当前记录来导航到一个新的 URL。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `to` | [`RouteLocationRaw`](../index.md#routelocationraw) | 要导航到的路由 |

#### 返回值

`Promise`\<`undefined` \| `void` \| [`NavigationFailure`](NavigationFailure.md)\>

___

### resolve

▸ **resolve**(`to`, `currentLocation?`): [`RouteLocation`](RouteLocation.md) & { `href`: `string`  }

返回一个[路由地址](../index.md#routelocationraw)的[规范化版本](RouteLocation.md)。同时包含一个包含任何现有 `base` 的 `href` 属性。默认情况下，用于 `router.currentRoute` 的 `currentLocation` 应该在特别高阶的用例下才会被覆写。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `to` | [`RouteLocationRaw`](../index.md#routelocationraw) | 要解析的原始路由地址 |
| `currentLocation?` | [`RouteLocationNormalizedLoaded`](RouteLocationNormalizedLoaded.md) | 可选的被解析的当前地址 |

#### 返回值

[`RouteLocation`](RouteLocation.md) & { `href`: `string`  }


- [RouterHistory](interfaces/RouterHistory.md)

---
editLink: false
---

[API 参考](../index.md) / RouterHistory

# 接口：RouterHistory

由 History 实现的接口，可以作为 Router.history 传递给路由器。

## 属性

### base

• `只读` **base**: `string`

基准路径，它被预置到每个 URL 上。这允许在一个域名子文件夹中托管 SPA，例如将 `base` 设置为 `/sub-folder` 使得其托管在 `example.com/sub-folder`。

___

### location

• `只读` **location**: `string`

当前历史的地址

___

### state

• `只读` **state**: [`HistoryState`](HistoryState.md)

当前历史的状态

## Methods

### createHref

▸ **createHref**(`location`): `string`

生成用于链接标签的相应的 href。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `location` | `string` | 应该创建一个 href 的历史的地址 |

#### 返回值

`string`

___

### destroy

▸ **destroy**(): `void`

清除任何通过该历史实现附加的事件监听器。

#### 返回值

`void`

___

### go

▸ **go**(`delta`, `triggerListeners?`): `void`

按指定方向访问历史。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `delta` | `number` | 访问的距离。如果 delta \< 0 则后退相应数量的记录，如果 \> 0 则前进。 |
| `triggerListeners?` | `boolean` | 是否应该触发连接到该历史的监听器 |

#### 返回值

`void`

**`Example`**

```js
myHistory.go(-1) // equivalent to window.history.back()
myHistory.go(1) // equivalent to window.history.forward()
```

___

### listen

▸ **listen**(`callback`): () => `void`

给历史实现附加一个监听器，当导航从外部被触发时 (像浏览器的前进后退按钮) 或者向 RouterHistory.back 和 RouterHistory.forward 传递 `true` 时，监听器就会被触发。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `callback` | `NavigationCallback` | 附加的监听器 |

#### 返回值

`fn`

用来移除该监听器的回调函数。

▸ (): `void`

给历史实现附加一个监听器，当导航从外部被触发时 (像浏览器的前进后退按钮) 或者向 RouterHistory.back 和 RouterHistory.forward 传递 `true` 时，监听器就会被触发。

##### 返回值

`void`

用来移除该监听器的回调函数。

___

### push

▸ **push**(`to`, `data?`): `void`

导航到一个地址。在 HTML5 历史实现下，这将调用 `history.pushState` 来有效改变 URL。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `to` | `string` | location to push |
| `data?` | [`HistoryState`](HistoryState.md) | 可选的 [HistoryState](HistoryState.md) 以关联该导航记录 |

#### 返回值

`void`

___

### replace

▸ **replace**(`to`, `data?`): `void`

和 [RouterHistory.push](RouterHistory.md#push) 相同，只是执行了 `history.replaceState`
以换掉 `history.pushState`。

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `to` | `string` | 要设置的地址 |
| `data?` | [`HistoryState`](HistoryState.md) | 可选的 [HistoryState](HistoryState.md) 以关联该导航记录 |

#### 返回值

`void`


- [RouterLinkProps](interfaces/RouterLinkProps.md)

---
editLink: false
---

[API 参考](../index.md) / RouterLinkProps

# 接口：RouterLinkProps

## 继承关系

- `RouterLinkOptions`

  ↳ **`RouterLinkProps`**

## 属性

### activeClass

• `可选` **activeClass**: `string`

链接在匹配当前路由时被应用到 class。

___

### ariaCurrentValue

• `可选` **ariaCurrentValue**: ``"location"`` \| ``"time"`` \| ``"page"`` \| ``"step"`` \| ``"date"`` \| ``"true"`` \| ``"false"``

链接在匹配当前路由时传入 `aria-current` attribute 的值。

**`Default Value`**

`'page'`

___

### custom

• `可选` **custom**: `boolean`

RouterLink 是否应该将其内容包裹在一个 `a` 标签里。用于通过 `v-slot` 创建自定义 RouterLink。

___

### exactActiveClass

• `可选` **exactActiveClass**: `string`

链接在严格匹配当前路由时被应用到 class。

___

### replace

• `可选` **replace**: `boolean`

调用 `router.replace` 以替换 `router.push`。

#### 继承自

RouterLinkOptions.replace

___

### to

• **to**: [`RouteLocationRaw`](../index.md#routelocationraw)

当点击该链接时应该进入的路由地址。

#### 继承自

RouterLinkOptions.to


- [RouterOptions](interfaces/RouterOptions.md)

---
editLink: false
---

[API 参考](../index.md) / RouterOptions

# 接口：RouterOptions

用来初始化一个 [Router](Router.md) 实例的选项。

## 继承关系

- [`PathParserOptions`](../index.md#pathparseroptions)

  ↳ **`RouterOptions`**

## 属性

### end

• `可选` **end**: `boolean`

其 RegExp 是否应该在末尾加一个 `$` 以匹配到末尾。

**`默认值`**

`true`

#### 继承自

PathParserOptions.end

___

### history

• **history**: [`RouterHistory`](RouterHistory.md)

路由器使用的历史记录模式。大多数应用应该使用 `createWebHistory`，但这需要正确配置服务器。你也可以使用 `createWebHashHistory` 来实现基于 *hash* 的历史记录，无需配置服务器。但这种方式不会被搜索引擎处理，SEO 的效果较差。

**`示例`**

```js
createRouter({
  history: createWebHistory(),
  // 其它选项...
})
```

___

### linkActiveClass

• `可选` **linkActiveClass**: `string`

匹配当前路由的 [RouterLink](../index.md#routerlink) 默认的 CSS class。如果没有提供，则会使用 `router-link-active`。

___

### linkExactActiveClass

• `可选` **linkExactActiveClass**: `string`

严格匹配当前路由的 [RouterLink](../index.md#routerlink) 默认的 CSS class。如果没有提供，则会使用 `router-link-exact-active`。

___

### parseQuery

• `可选` **parseQuery**: (`search`: `string`) => [`LocationQuery`](../index.md#locationquery)

#### 类型声明

▸ (`search`): [`LocationQuery`](../index.md#locationquery)

解析查询的自定义实现。请查阅其相关内容 [RouterOptions.stringifyQuery](RouterOptions.md#stringifyquery)。

##### 参数

| 名称 | 类型 |
| :------ | :------ |
| `search` | `string` |

##### 返回值

[`LocationQuery`](../index.md#locationquery)

**`示例`**

假设你想使用 [qs 包](https://github.com/ljharb/qs) 来解析查询，那么你可以同时提供 `parseQuery` 和 `stringifyQuery`：

```js
import qs from 'qs'

createRouter({
  // 其它选项...
  parseQuery: qs.parse,
  stringifyQuery: qs.stringify,
})
```

___

### routes

• **routes**: 只读 [`RouteRecordRaw`](../index.md#routerecordraw)[]

应该添加到路由器的初始路由列表。

___

### scrollBehavior

• `可选` **scrollBehavior**: [`RouterScrollBehavior`](RouterScrollBehavior.md)

当在页面之间导航时控制滚动的功能。可以返回一个 Promise 来延迟滚动。相关内容请查阅 ScrollBehavior。

**`示例`**

```js
function scrollBehavior(to, from, savedPosition) {
  // `to` 和 `from` 都是路由路径
  // `savedPosition` 如果不存在可以为 null
}
```

___

### sensitive

• `可选` **sensitive**: `boolean`

使该 RegExp 区分大小写。

**`默认值`**

`false`

#### 继承自

PathParserOptions.sensitive

___

### strict

• `可选` **strict**: `boolean`

是否禁止尾部斜线。

**`默认值`**

`false`

#### 继承自

PathParserOptions.strict

___

### stringifyQuery

• `可选` **stringifyQuery**: (`query`: [`LocationQueryRaw`](../index.md#locationqueryraw)) => `string`

#### 类型声明

▸ (`query`): `string`

对查询对象进行字符串化的自定义实现。该实现不应该前置 `?`。[parseQuery](RouterOptions.md#parsequery) 对应处理查询解析。

##### Parameters

| 名称 | 类型 |
| :------ | :------ |
| `query` | [`LocationQueryRaw`](../index.md#locationqueryraw) |

##### 返回值

`string`


- [RouterScrollBehavior](interfaces/RouterScrollBehavior.md)

---
editLink: false
---

[API 参考](../index.md) / RouterScrollBehavior

# 接口：RouterScrollBehavior

可以被传递给 `createRouter` 的 `scrollBehavior` 选项的类型。

## 可调用函数

### RouterScrollBehavior

▸ **RouterScrollBehavior**(`to`, `from`, `savedPosition`): `Awaitable`\<``false`` \| `void` \| `ScrollPosition`\>

#### 参数

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `to` | [`RouteLocationNormalized`](RouteLocationNormalized.md) | 我们要导航到的路由地址 |
| `from` | [`RouteLocationNormalizedLoaded`](RouteLocationNormalizedLoaded.md) | 我们要离开的路由地址 |
| `savedPosition` | ``null`` \| `_ScrollPositionNormalized` | 要保存的页面位置，如果不存在则是 `null` |

#### 返回值

`Awaitable`\<``false`` \| `void` \| `ScrollPosition`\>


- [RouterViewProps](interfaces/RouterViewProps.md)

---
editLink: false
---

[API 参考](../index.md) / RouterViewProps

# 接口：RouterViewProps

## 属性

### name

• `可选` **name**: `string`

___

### route

• `可选` **route**: [`RouteLocationNormalized`](RouteLocationNormalized.md)


- [\_RouteRecordBase](interfaces/RouteRecordBase.md)

---
editLink: false
---

<!-- TODO: translation -->

[API Documentation](../index.md) / \_RouteRecordBase

# Interface: \_RouteRecordBase

Internal type for common properties among all kind of [RouteRecordRaw](../index.md#RouteRecordRaw).

## Hierarchy

- [`PathParserOptions`](../index.md#PathParserOptions)

  ↳ **`_RouteRecordBase`**

  ↳↳ [`RouteRecordSingleView`](RouteRecordSingleView.md)

  ↳↳ [`RouteRecordSingleViewWithChildren`](RouteRecordSingleViewWithChildren.md)

  ↳↳ [`RouteRecordMultipleViews`](RouteRecordMultipleViews.md)

  ↳↳ [`RouteRecordMultipleViewsWithChildren`](RouteRecordMultipleViewsWithChildren.md)

  ↳↳ [`RouteRecordRedirect`](RouteRecordRedirect.md)

## Properties

### alias

• `Optional` **alias**: `string` \| `string`[]

Aliases for the record. Allows defining extra paths that will behave like a
copy of the record. Allows having paths shorthands like `/users/:id` and
`/u/:id`. All `alias` and `path` values must share the same params.

___

### beforeEnter

• `Optional` **beforeEnter**: [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\> \| [`NavigationGuardWithThis`](NavigationGuardWithThis.md)\<`undefined`\>[]

Before Enter guard specific to this record. Note `beforeEnter` has no
effect if the record has a `redirect` property.

___

### children

• `Optional` **children**: [`RouteRecordRaw`](../index.md#RouteRecordRaw)[]

Array of nested routes.

___

### end

• `Optional` **end**: `boolean`

Should the RegExp match until the end by appending a `$` to it.

**`Default Value`**

`true`

#### Inherited from

PathParserOptions.end

___

### meta

• `Optional` **meta**: [`RouteMeta`](RouteMeta.md)

Arbitrary data attached to the record.

___

### name

• `Optional` **name**: [`RouteRecordName`](../index.md#RouteRecordName)

Name for the route record. Must be unique.

___

### path

• **path**: `string`

Path of the record. Should start with `/` unless the record is the child of
another record.

**`Example`**

```ts
`/users/:id` matches `/users/1` as well as `/users/posva`.
```

___

### props

• `Optional` **props**: `_RouteRecordProps` \| `Record`\<`string`, `_RouteRecordProps`\>

Allow passing down params as props to the component rendered by `router-view`.

___

### redirect

• `Optional` **redirect**: `RouteRecordRedirectOption`

Where to redirect if the route is directly matched. The redirection happens
before any navigation guard and triggers a new navigation with the new
target location.

___

### sensitive

• `Optional` **sensitive**: `boolean`

Makes the RegExp case-sensitive.

**`Default Value`**

`false`

#### Inherited from

PathParserOptions.sensitive

___

### strict

• `Optional` **strict**: `boolean`

Whether to disallow a trailing slash or not.

**`Default Value`**

`false`

#### Inherited from

PathParserOptions.strict



## TS 类型别名 

### LocationQuery 

Ƭ **LocationQuery**: `Record`\<`string`, `LocationQueryValue` \| `LocationQueryValue`[]\>

出现在 [RouteLocationNormalized](interfaces/RouteLocationNormalized.md) 中的规范化查询对象。

___

### LocationQueryRaw 

Ƭ **LocationQueryRaw**: `Record`\<`string` \| `number`, `LocationQueryValueRaw` \| `LocationQueryValueRaw`[]\>

松散的 [LocationQuery](index.md#locationquery) 对象，可以被传递给诸如
[Router.push](interfaces/Router.md#push)、[Router.replace](interfaces/Router.md#replace) 或任何创建
[RouteLocationRaw](index.md#routelocationraw) 的函数。

___

### PathParserOptions 

Ƭ **PathParserOptions**: `Pick`\<`_PathParserOptions`, ``"end"`` \| ``"sensitive"`` \| ``"strict"``\>

___

### RouteComponent 

Ƭ **RouteComponent**: `Component` \| `DefineComponent`

在 [RouteLocationMatched](interfaces/RouteLocationMatched.md) 中允许的组件。

___

### RouteLocationRaw 

Ƭ **RouteLocationRaw**: `string` \| `RouteLocationPathRaw` \| `RouteLocationNamedRaw`

用户级别的路由位置。

___

### RouteParams 

Ƭ **RouteParams**: `Record`\<`string`, `RouteParamValue` \| `RouteParamValue`[]\>

___

### RouteParamsRaw 

Ƭ **RouteParamsRaw**: `Record`\<`string`, `RouteParamValueRaw` \| `Exclude`\<`RouteParamValueRaw`, ``null`` \| `undefined`\>[]\>

___

### RouteRecord 

Ƭ **RouteRecord**: [`RouteRecordNormalized`](interfaces/RouteRecordNormalized.md)

一个[路由记录](index.md#routerecord)的规范化版本。

___

### RouteRecordName 

Ƭ **RouteRecordName**: `string` \| `symbol`

用户定义的路由记录的可能的名称。

___

### RouteRecordRaw 

Ƭ **RouteRecordRaw**: [`RouteRecordSingleView`](interfaces/RouteRecordSingleView.md) \| [`RouteRecordSingleViewWithChildren`](interfaces/RouteRecordSingleViewWithChildren.md) \| [`RouteRecordMultipleViews`](interfaces/RouteRecordMultipleViews.md) \| [`RouteRecordMultipleViewsWithChildren`](interfaces/RouteRecordMultipleViewsWithChildren.md) \| [`RouteRecordRedirect`](interfaces/RouteRecordRedirect.md)

___

### UseLinkOptions 

Ƭ **UseLinkOptions**: `VueUseOptions`\<`RouterLinkOptions`\>

## 变量 

### RouterLink 

• `Const` **RouterLink**: `_RouterLinkI`

用来渲染一个链接的组件，该链接在被点击时会触发导航。

___

### RouterView 

• `Const` **RouterView**: () => \{ `$props`: `AllowedComponentProps` & `ComponentCustomProps` & `VNodeProps` & [`RouterViewProps`](interfaces/RouterViewProps.md) ; `$slots`: \{ `default?`: (`__namedParameters`: \{ `Component`: `VNode`\<`RendererNode`, `RendererElement`, \{ `[key: string]`: `any`;  }\> ; `route`: [`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)  }) => `VNode`\<`RendererNode`, `RendererElement`, \{ `[key: string]`: `any`;  }\>[]  }  }

#### 类型声明 

• **new RouterView**(): `Object`

用于显示用户当前所处路由的组件。

##### 返回值 

`Object`

| 名称 | 类型 |
| :------ | :------ |
| `$props` | `AllowedComponentProps` & `ComponentCustomProps` & `VNodeProps` & [`RouterViewProps`](interfaces/RouterViewProps.md) |
| `$slots` | \{ `default?`: (`__namedParameters`: \{ `Component`: `VNode`\<`RendererNode`, `RendererElement`, \{ `[key: string]`: `any`;  }\> ; `route`: [`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)  }) => `VNode`\<`RendererNode`, `RendererElement`, \{ `[key: string]`: `any`;  }\>[]  } |
| `$slots.default?` | (`__namedParameters`: \{ `Component`: `VNode`\<`RendererNode`, `RendererElement`, \{ `[key: string]`: `any`;  }\> ; `route`: [`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)  }) => `VNode`\<`RendererNode`, `RendererElement`, \{ `[key: string]`: `any`;  }\>[] |

___

### START\_LOCATION 

• `Const` **START\_LOCATION**: [`RouteLocationNormalizedLoaded`](interfaces/RouteLocationNormalizedLoaded.md)

路由器的初始路由位置。可以在导航守卫中使用来区分初始导航。

**示例**

```js
import { START_LOCATION } from 'vue-router'

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

| 名称 | 类型 | 描述 |
| :------ | :------ | :------ |
| `options` | [`RouterOptions`](interfaces/RouterOptions.md) | [RouterOptions](interfaces/RouterOptions.md) |

#### 返回值 

[`Router`](interfaces/Router.md)

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
