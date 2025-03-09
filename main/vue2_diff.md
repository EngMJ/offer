## 1. 组件更新与 VNode 生成

- **响应式触发**  
  当组件内的数据变化时，响应式系统会触发该组件的重新渲染。
- **生成新 VNode 树**  
  组件的 render 函数被调用，生成一颗新的虚拟 DOM（VNode）树。
- **旧 VNode 树**  
  此前渲染好的虚拟 DOM 树保存在内存中，作为 diff 的对比基准。

---

## 2. 进入 Patch 流程

- **patch 函数**  
  Vue2 的更新入口就是 patch(oldVnode, vnode)：
    - **判断是否为真实 DOM**：如果 oldVnode 是一个真实 DOM 元素（首次渲染时），则根据 vnode 创建新的 DOM，并替换旧 DOM。
    - **同一节点比较**：如果 oldVnode 和 vnode 被认为是“相同”的（通常需要 tag 相同、key 相同），则调用 patchVnode 进行“精细化”更新。
    - **不相同直接替换**：如果判断不相同，则直接用新 vnode 创建对应 DOM，并替换旧节点。

---

## 3. 节点级别更新（patchVnode）

进入 patchVnode 后，主要工作包括：

### 3.1 复用旧 DOM 节点

- 将 vnode.el 指向旧节点的 DOM（复用已有 DOM），以便在后续操作中直接修改。

### 3.2 文本节点更新

- 如果旧 vnode 和新 vnode 均为文本节点，则直接比较文本内容：
    - 若不同，调用 DOM API（例如 setTextContent）更新文本内容。

### 3.3 属性更新（patchProps）

- 对比旧 vnode.data 与新 vnode.data 中的各个属性（如 class、style、事件等）：
    - 对于值有变化的属性，调用相应 API 更新真实 DOM；
    - 对于新增加的属性，添加到 DOM 上；
    - 对于删除的属性，从 DOM 上移除。

### 3.4 子节点更新

- 如果 vnode 既有 children 数组又有文本节点，则优先考虑 children 数组。
- 当新旧 vnode 均有 children 数组时，进入**子节点 diff 流程**（调用 updateChildren 函数）。
- 如果只有一个有 children，则执行对应的挂载或卸载操作。

---

## 4. 子节点 Diff 流程（updateChildren）

假设旧 children 数组为 c1，新 children 数组为 c2，diff 过程如下：

### 4.1 初始化指针

- 定义四个指针：
    - **oldStartIdx** = 0，**oldEndIdx** = c1.length - 1
    - **newStartIdx** = 0，**newEndIdx** = c2.length - 1
- 分别取出对应的节点：
    - **oldStartVnode** = c1[oldStartIdx]，**oldEndVnode** = c1[oldEndIdx]
    - **newStartVnode** = c2[newStartIdx]，**newEndVnode** = c2[newEndIdx]

### 4.2 双端对比（四种情形）

在 while 循环中，当 oldStartIdx ≤ oldEndIdx 且 newStartIdx ≤ newEndIdx 时，依次尝试以下对比策略：

1. **头对头比较**
    - 如果 sameVnode(oldStartVnode, newStartVnode) 为 true，则调用 patchVnode 更新该节点；
    - 然后将 oldStartIdx 和 newStartIdx 分别自增 1，并更新对应的节点。

2. **尾对尾比较**
    - 如果 sameVnode(oldEndVnode, newEndVnode) 为 true，则调用 patchVnode 更新；
    - 将 oldEndIdx 与 newEndIdx 分别自减 1，并更新对应节点。

3. **头对尾比较**
    - 如果 sameVnode(oldStartVnode, newEndVnode) 为 true，则表示旧头节点移动到了尾部；
    - 调用 patchVnode 更新后，将旧头节点通过 DOM 插入（例如 insertBefore(oldStartVnode.el, oldEndVnode.el.nextSibling)）到旧尾节点之后；
    - 然后 oldStartIdx 自增、新EndIdx 自减。

4. **尾对头比较**
    - 如果 sameVnode(oldEndVnode, newStartVnode) 为 true，则说明旧尾节点移动到了头部；
    - 调用 patchVnode 更新后，通过 DOM 插入将旧尾节点移动到旧头节点之前；
    - 然后 oldEndIdx 自减、新StartIdx 自增。

### 4.3 乱序部分处理

当上述四种比较均不匹配时：

- **建立映射表**
    - 为旧 children 数组中未处理部分（从 oldStartIdx 到 oldEndIdx）建立一个 key→索引 的哈希表。

- **查找匹配节点**
    - 对于新 children 中当前的新节点 newStartVnode，根据其 key 到映射表中查找对应的旧节点索引。
    - 如果找到了并且节点满足 sameVnode，则调用 patchVnode 更新，并将对应旧节点标记为已处理（例如设置为 null）；
    - 同时将该旧节点移动到当前 oldStartVnode 的前面。

- **处理未匹配的情况**
    - 如果没找到，则说明当前新节点是新增的，调用 createElm 创建对应的 DOM，并插入到 oldStartVnode 的前面。
    - 新指针 newStartIdx 自增。

### 4.4 循环结束后处理剩余节点

- **新增节点**
    - 如果 newStartIdx ≤ newEndIdx（即新 children 中还有剩余），则遍历剩余的新节点，调用 createElm 创建 DOM，并插入到合适位置（一般插入到 newEndIdx 后面的锚点处）。

- **删除节点**
    - 如果 oldStartIdx ≤ oldEndIdx（即旧 children 中还有剩余未处理的节点），则遍历删除这些节点对应的 DOM 元素。

