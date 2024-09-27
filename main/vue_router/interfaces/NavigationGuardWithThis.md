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
