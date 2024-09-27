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
