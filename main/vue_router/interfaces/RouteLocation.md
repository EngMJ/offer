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
