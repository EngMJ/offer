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
