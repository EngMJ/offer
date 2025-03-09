下面提供一个比较全面的流程，涵盖了从组件更新触发、节点更新、属性差异比较到子节点 diff 的全过程。下述流程基于 Vue3 的更新机制，但其中很多基本原理也适用于 Vue2（只是 Vue3 在乱序部分引入了 LIS 优化）。

---

## 1. 触发更新：响应式与组件重新渲染

- **数据变化**  
  当组件内部的响应式数据发生变化时，会触发更新（例如通过 setter 通知订阅者）。

- **组件更新**  
  对应组件会调用其 render 函数生成新的 VNode 树。这个 VNode 树包含了节点（元素或组件）、属性（props、class、style、事件等）和子节点（文本或 VNode 数组）的完整描述。

---

## 2. 进入 Patch 流程：新旧 VNode 对比

- **初步判断**  
  进入 patch 函数后，首先会判断新旧 VNode 是否“相同”（通常通过比较 tag（或组件类型）、key、isComment、数据是否定义等条件）。
    - **如果不同**，则直接销毁旧节点并创建新节点。
    - **如果相同**，则进入进一步的“精细化”比较。

- **组件 vs 元素**
    - 如果 VNode 表示的是一个组件（即构造函数或组件选项），那么 patch 流程会进入组件更新分支：
        - 检查已有的组件实例，如果存在则更新 props 并调用组件内部的 render 方法生成新的子树，然后递归进入 patch。
        - 如果不存在，则创建组件实例，并进行初次挂载。
    - 如果 VNode 表示的是普通 DOM 元素，则走元素的更新逻辑。

---

## 3. 节点更新（对于普通 DOM 元素）

### 3.1 节点替换判断

- **判断节点类型**  
  如果新旧节点的 tag 或 key 不同，认为节点不再相同，则直接调用创建新节点的逻辑并替换整个 DOM 元素。

### 3.2 更新属性（属性 / 组件数据）

- **属性比较**  
  Vue 会对比旧 VNode 的 data（包括 attrs、class、style、事件等）和新 VNode 的 data：
    - 对于相同属性且值未变化的不做处理。
    - 对于发生变化的属性，会调用相应的 DOM API 更新，比如修改元素的 class、style、设置或移除属性。
    - 这部分通常在一个叫 patchProps（或 updateProps）的函数中处理。

### 3.3 更新子节点

- **文本节点 vs 数组**
    - 如果旧节点和新节点均为文本节点，则直接比较文本内容，若不同则更新文本。
    - 如果存在子节点数组，则进入子节点 diff 流程。

---

## 4. 子节点 Diff 流程

假设同一父元素下的子节点分别构成旧 children 数组（c1）和新 children 数组（c2），diff 流程通常分为以下几个阶段：

### 4.1 初始阶段

在 patch 过程中，Vue3 会对某个父节点下的 children 数组进行比对。假设旧 children 数组为 c1，新 children 数组为 c2，我们首先记录以下几个下标和长度变量：

- **i**：新旧两边从前开始遍历的起始下标，初始值为 0
- **e1**：旧 children 数组的末尾下标，即 `c1.length - 1`
- **e2**：新 children 数组的末尾下标，即 `c2.length - 1`

此时的目标是找出两侧连续相同的部分，以便将工作量缩小到中间那部分发生变动的节点。

---

### 4.2 头部同步（Sync from Start）

从数组的开头开始，逐个比较新旧节点。如果两个节点是“相同的”（通常通过 isSameVNodeType 判断，比较标签、key、是否为文本节点等）：
- 调用 patch 更新这部分节点（如果需要的话会递归更新子节点）。
- 将指针 i 自增 1。

伪代码示例：

```js
while (i <= e1 && i <= e2) {
  const prevChild = c1[i]
  const nextChild = c2[i]
  if (!isSameVNodeType(prevChild, nextChild)) {
    break
  }
  patch(prevChild, nextChild, container, parentAnchor, parentComponent)
  i++
}
```

经过这一阶段，前面相同的那部分节点都已经处理完毕。

---

### 4.3 尾部同步（Sync from End）

类似地，从数组的末尾开始，倒序逐个比较新旧节点：
- 如果旧尾和新尾节点是相同的，则调用 patch 更新，并将旧尾和新尾指针分别自减 1。

示例伪代码：

```js
while (i <= e1 && i <= e2) {
  const prevChild = c1[e1]
  const nextChild = c2[e2]
  if (!isSameVNodeType(prevChild, nextChild)) {
    break
  }
  patch(prevChild, nextChild, container, parentAnchor, parentComponent)
  e1--
  e2--
}
```

这样，前后两端连续相同的部分就被“剥离”出来，剩下的部分就是需要进一步处理的中间区间。

---

### 4.4 中间部分处理

经过头尾同步后，中间可能存在三种情况：
- **情况 A**：新 children 数组还有剩余节点，而旧 children 数组已经用完 → 需要新增节点。
- **情况 B**：旧 children 数组还有剩余节点，而新 children 数组用完 → 需要删除旧节点。
- **情况 C**：两边都还有剩余，即中间部分发生了乱序、移动或局部更新的情况。

这里我们重点讨论情况 C 的处理流程。

1. 建立新节点映射表

首先，为剩余的新节点（即从下标 s2 = i 到 e2 的部分）建立一个 key 到新节点索引的映射表，这样可以在后续快速查找新节点对应的位置。

```js
const keyToNewIndexMap = new Map()
for (let j = i; j <= e2; j++) {
  const nextChild = c2[j]
  keyToNewIndexMap.set(nextChild.key, j)
}
```

2. 构造 newIndexToOldIndexMap 数组

然后，遍历旧 children 数组中中间未匹配的部分（下标范围 s1 = i 到 e1），对于每个旧节点：
- 如果它的 key 在新映射表中存在，则记录其在新数组中的位置；
- 如果不存在，则说明该旧节点需要被删除。

同时，我们构造一个数组 newIndexToOldIndexMap，其长度为 `toBePatched = e2 - i + 1`，初始值全部为 0。对于旧节点能匹配到的情况，我们把对应的索引记录进去（通常记录的是旧节点在 c1 中的下标 + 1，以避免 0 与“未找到”混淆）。

3. 计算最长递增子序列（LIS）

[参考:计算最长递增子序列](./algorithm.md#最长递增子序列)

在 newIndexToOldIndexMap 数组中，那些已经处于正确顺序的旧节点不需要移动。为了找出这部分节点，我们对这个数组应用最长递增子序列（LIS）算法。
- 得到的最长子序列（实际上是新数组中的索引序列）表示那些节点保持原位即可。
- 其他节点则需要通过 DOM 移动来达到正确的位置。

伪代码示例（省略具体细节）：

```js
const increasingNewIndexSequence = moved
  ? getSequence(newIndexToOldIndexMap)
  : []
```

其中 `getSequence` 函数就是用“贪心 + 二分”来计算最长递增子序列，其时间复杂度通常为 O(n log n)。

4. 移动与插入操作

接下来倒序遍历剩余的新节点（从最后一个到第一个），对每个新节点：
- 如果 newIndexToOldIndexMap 中对应值为 0，则表示该新节点在旧数组中不存在，需要调用 patch(null, newVNode, …) 来创建该节点。
- 如果存在，但其位置不在最长递增子序列内，则需要将其移动到正确的位置（通常使用 hostInsert 或 insertBefore）。
- 如果在 LIS 内，则无需移动，直接跳过即可。

同时，对未匹配到的旧节点（在遍历时被标记为 null 或未被使用的），在最后进行删除。

---


