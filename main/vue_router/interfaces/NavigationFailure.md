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
