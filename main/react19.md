# 高级前端 React 高频面试题列表

---

## 一、React 核心原理

### 1. React 的设计哲学是什么？为什么选择声明式而不是命令式？

**答案：**

React 的核心设计哲学是 **UI = f(state)**，即界面是状态的函数。开发者只需描述"UI 应该是什么样子"，而不需要关心"如何一步步操作 DOM 达到这个状态"。

**声明式 vs 命令式的本质区别：**

```javascript
// 命令式：关注"怎么做"
const list = document.getElementById('list');
list.innerHTML = '';
data.forEach(item => {
  const li = document.createElement('li');
  li.textContent = item.name;
  list.appendChild(li);
});

// 声明式：关注"是什么"
return <ul>{data.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
```

**选择声明式的原因：**

1. **可预测性**：相同的 state 永远产生相同的 UI，便于调试和测试
2. **抽象复杂度**：将 DOM 操作的复杂性封装在框架内部，开发者只需关注业务逻辑
3. **优化空间**：框架可以在内部做批量更新、Diff 优化等，开发者无需手动优化
4. **组件化基础**：声明式天然适合组件拆分和组合

**技术要点：**
- 声明式并不意味着性能更好，而是提供了更好的开发体验和可维护性
- React 内部依然执行命令式的 DOM 操作，只是对开发者隐藏了这些细节

---

### 2. Virtual DOM 的本质是什么？它真的比直接操作 DOM 快吗？

**答案：**

Virtual DOM 的本质是 **用 JavaScript 对象描述 UI 结构的轻量级抽象层**。它不是为了"更快"，而是为了提供一种 **可预测、跨平台、便于优化的渲染模型**。

**Virtual DOM 的结构：**

```javascript
// JSX
<div className="container">
  <span>Hello</span>
</div>

// 转换为 Virtual DOM 对象
{
  type: 'div',
  props: {
    className: 'container',
    children: {
      type: 'span',
      props: { children: 'Hello' }
    }
  }
}
```

**性能真相：**

Virtual DOM **不比直接操作 DOM 快**，准确地说：

1. **最优手写 DOM 操作** > Virtual DOM > **无脑全量替换 innerHTML**
2. Virtual DOM 的价值是在 **"不需要开发者手动优化"** 的前提下，提供 **"足够好"** 的性能

**Virtual DOM 的真实价值：**

1. **批量更新**：收集多次状态变更，一次性计算最小 DOM 操作
2. **跨平台**：同一套 Virtual DOM 可以渲染到 DOM、Native、Canvas 等
3. **可预测的更新**：通过 Diff 算法精确计算需要更新的节点
4. **函数式编程友好**：每次渲染生成新树，便于实现时间旅行等功能

**常见误区：**
- 误以为 Virtual DOM 是 React 的核心创新（实际上声明式编程模型才是）
- 误以为所有场景都应该用 React（简单页面直接操作 DOM 可能更高效）

---

### 3. React Diff 算法的三个假设/策略是什么？为什么 key 如此重要？

**答案：**

React Diff 算法基于三个核心假设，将 O(n³) 的树比对复杂度降到 O(n)：

**三个假设/策略：**

1. **Tree Diff（层级策略）**：只比较同一层级的节点，跨层级移动视为删除+创建
2. **Component Diff（组件策略）**：不同类型的组件生成不同的树，直接替换整个子树
3. **Element Diff（元素策略）**：同一层级的子节点通过 key 标识，实现高效复用

**key 的重要性：**

```javascript
// 没有 key 或使用 index 作为 key
// React 无法识别哪个元素被移动/删除，只能按顺序逐个对比

// 列表从 [A, B, C] 变为 [C, A, B]
// 无 key：认为 A->C, B->A, C->B，三次更新
// 有 key：识别为移动操作，DOM 节点复用

<ul>
  {items.map(item => <li key={item.id}>{item.name}</li>)}
</ul>
```

**key 的作用机制：**

1. key 帮助 React 在 Diff 时建立 **旧节点到新节点的映射关系**
2. 相同 key 的节点被认为是"同一个元素"，可以复用 DOM 和内部状态
3. key 变化时，React 会销毁旧组件并创建新组件（可用于强制重置状态）

**key 的最佳实践：**
- 使用稳定、唯一的业务 ID 作为 key
- 避免使用 index（除非列表是静态且不会重排序）
- 避免使用随机数（每次渲染都会强制重建）

**key 使用 index 会带来哪些具体问题？**

**答案：**

**问题 1：列表项状态错乱**

```javascript
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React' },
    { id: 2, text: 'Build App' },
  ]);
  
  const addFirst = () => {
    setTodos([{ id: 3, text: 'New Todo' }, ...todos]);
  };
  
  return (
    <>
      <button onClick={addFirst}>Add to top</button>
      {todos.map((todo, index) => (
        // 使用 index 作为 key
        <TodoItem key={index} todo={todo} />
      ))}
    </>
  );
}

function TodoItem({ todo }) {
  const [isEditing, setIsEditing] = useState(false);
  
  // 问题：点击 "Add to top" 后
  // 原来 index=0 的 "Learn React" 变成了 index=1
  // 但 React 认为 key=0 的还是同一个组件
  // 导致新添加的 todo 继承了原来第一项的编辑状态！
  
  return <div>{isEditing ? <input /> : todo.text}</div>;
}
```

**问题 2：输入框内容错乱**

```javascript
function List({ items }) {
  return items.map((item, index) => (
    <div key={index}>
      {item.name}
      <input defaultValue={item.value} />
    </div>
  ));
}

// 当列表重排序时：
// 原来: [{name: 'A', value: '1'}, {name: 'B', value: '2'}]
// 删除 A: [{name: 'B', value: '2'}]
// 
// React 看到 key=0 还在，认为是同一个元素
// 但数据已经是 B 了，而 input 还保持着 A 的值 '1'
```

**问题 3：动画异常**

```javascript
// 使用 index 作为 key 时
// 删除中间项会导致后面所有项的 key 变化
// 动画库可能无法正确识别哪个元素被删除

// 原来: [0, 1, 2] -> 删除 index=1 -> [0, 1]
// React 认为 key=2 被删除了，而不是原来 key=1 的元素
```

**问题 4：性能下降**

```javascript
// 在列表开头插入元素时
// 所有元素的 index 都变了，React 认为所有元素都"改变"了
// 导致整个列表重新渲染，无法复用任何 DOM 节点

// 使用稳定的 id 作为 key 时
// React 知道只是插入了一个新元素，其他元素可以复用
```

**什么时候可以用 index：**

1. 列表是静态的，不会重排序、增删
2. 列表项没有内部状态（如输入框、展开/折叠）
3. 列表项没有依赖位置的动画

```javascript
// 可以用 index 的场景
const weekdays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri'];
weekdays.map((day, index) => <span key={index}>{day}</span>);
```
---

### 4. React Fiber 是什么？它解决了旧架构的什么根本问题？

**答案：**

Fiber 是 React 16 引入的 **新协调引擎**，本质是将原来基于递归的栈调用改为 **基于链表的可中断循环**。

**旧架构（Stack Reconciler）的问题：**

```javascript
// 旧架构：递归遍历，一旦开始无法中断
function reconcile(element) {
  // 处理当前节点
  element.children.forEach(child => reconcile(child)); // 递归
}
// 如果组件树很深，这个递归会长时间占用主线程
// 导致用户输入、动画等高优先级任务无法响应
```

**Fiber 的核心改进：**

1. **可中断渲染**：将渲染工作拆分成小单元，每个单元执行后检查是否需要让出主线程
2. **优先级调度**：不同更新有不同优先级，高优先级任务可以打断低优先级任务
3. **增量渲染**：渲染工作可以分多个帧完成，不会阻塞浏览器

**Fiber 节点的数据结构：**

```javascript
{
  type: 'div',           // 节点类型
  key: null,             // 唯一标识
  stateNode: domNode,    // 对应的真实 DOM 或组件实例
  child: childFiber,     // 第一个子节点
  sibling: siblingFiber, // 下一个兄弟节点
  return: parentFiber,   // 父节点
  alternate: oldFiber,   // 上一次渲染的 Fiber（双缓冲）
  effectTag: 'UPDATE',   // 副作用标记
  // ...
}
```

**Fiber 解决的根本问题：**
- 长任务阻塞导致的页面卡顿
- 无法区分任务优先级
- 用户交互响应延迟

---

### 5. Fiber 中的「可中断渲染」具体是如何实现的？

**答案：**

Fiber 的可中断渲染通过 **工作循环（Work Loop）+ 时间切片（Time Slicing）** 实现。

**核心实现原理：**

```javascript
// 简化的工作循环
function workLoop(deadline) {
  let shouldYield = false;
  
  while (nextUnitOfWork && !shouldYield) {
    // 执行一个工作单元，返回下一个工作单元
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    // 检查是否还有剩余时间（默认 5ms 一个时间片）
    shouldYield = deadline.timeRemaining() < 1;
  }
  
  // 如果还有工作，请求下一个空闲时段继续
  if (nextUnitOfWork) {
    requestIdleCallback(workLoop);
  }
}

requestIdleCallback(workLoop);
```

**关键技术点：**

1. **链表结构**：通过 child/sibling/return 指针形成链表，可以随时暂停和恢复遍历
2. **时间切片**：使用 `requestIdleCallback`（实际用 MessageChannel 模拟）在浏览器空闲时执行
3. **双缓冲机制**：同时维护 current 树和 workInProgress 树，完成后一次性切换

**工作单元的遍历顺序：**

```
     A
    /|\
   B C D
  /|
 E F

遍历顺序：A → B → E → F → C → D
（深度优先，先 child，无 child 则 sibling，无 sibling 则回溯到 parent 的 sibling）
```

**为什么能中断：**
- 每处理完一个 Fiber 节点，就有机会检查是否需要让出控制权
- 中断时保存当前 Fiber 指针，恢复时从该位置继续
- 所有中间状态都保存在 Fiber 节点上，不依赖调用栈

---

### 6. React 一次更新的完整流程（从 setState 到 DOM 更新）

**答案：**

一次完整的更新流程分为 **触发 → 调度 → 协调 → 提交** 四个阶段：

```
setState() 
    ↓
┌─────────────────┐
│  触发阶段        │  创建 Update 对象，加入更新队列
└─────────────────┘
    ↓
┌─────────────────┐
│  调度阶段        │  Scheduler 根据优先级安排任务
└─────────────────┘
    ↓
┌─────────────────┐
│  协调阶段        │  Reconciler 构建 Fiber 树，标记副作用（可中断）
│  (Render Phase) │
└─────────────────┘
    ↓
┌─────────────────┐
│  提交阶段        │  Renderer 执行 DOM 操作（不可中断）
│  (Commit Phase) │
└─────────────────┘
```

**详细流程：**

1. **触发阶段**
   - 调用 setState/forceUpdate/useState 的 setter
   - 创建 Update 对象，包含新状态和优先级信息
   - 将 Update 加入 Fiber 节点的 updateQueue

2. **调度阶段**
   - Scheduler 接收到更新任务
   - 根据任务优先级（lane）决定何时执行
   - 高优先级任务可以打断低优先级任务

3. **协调阶段（Render Phase）**
   - 从根节点开始，深度优先遍历 Fiber 树
   - 对比新旧 Fiber，计算需要变更的内容
   - 给需要操作的 Fiber 打上 effectTag（Placement/Update/Deletion）
   - 收集所有有副作用的 Fiber 形成 Effect List
   - **此阶段可以被中断**

4. **提交阶段（Commit Phase）**
   - Before Mutation：调用 getSnapshotBeforeUpdate
   - Mutation：执行 DOM 操作
   - Layout：调用 componentDidMount/Update，执行 useLayoutEffect
   - **此阶段同步执行，不可中断**

---

### 7. Reconciliation 阶段和 Commit 阶段分别做什么？为什么要拆分？

**答案：**

**Reconciliation 阶段（协调/渲染阶段）：**

- 构建新的 Fiber 树（workInProgress tree）
- 执行组件的 render 函数
- 对比新旧 Fiber，计算差异
- 标记副作用（effectTag）
- **特点：纯计算，无副作用，可中断可重复执行**

**Commit 阶段（提交阶段）：**

- 根据 effectTag 执行实际的 DOM 操作
- 调用生命周期方法（componentDidMount/Update）
- 执行 useLayoutEffect 和 useEffect
- **特点：有副作用，必须同步完成，不可中断**

**为什么要拆分：**

1. **保证 UI 一致性**：
   - Reconciliation 可以被打断和重试，但用户不会看到中间状态
   - Commit 阶段一次性完成所有 DOM 更新，避免 UI 撕裂

2. **支持并发特性**：
   - 可以同时准备多个版本的 UI（如 Suspense 的 fallback 和 content）
   - 可以丢弃低优先级的渲染结果，切换到高优先级任务

3. **优化性能**：
   - 将 CPU 密集的计算（Reconciliation）分散到多个帧
   - 将 DOM 操作（Commit）集中执行，减少布局抖动

**注意事项：**
- render 函数可能被多次调用（因为可以中断重试），所以必须是纯函数
- 副作用只能放在 Commit 阶段执行的生命周期/Effect 中

---

### 8. React 中「优先级」是如何影响更新调度的？

**答案：**

React 使用 **Lane 模型** 来管理更新优先级，不同优先级的更新会被不同方式调度。

**优先级等级（从高到低）：**

```javascript
// 简化的优先级分类
SyncLane           // 同步优先级：用户输入、点击事件
InputContinuousLane // 连续输入：拖拽、滚动
DefaultLane        // 默认优先级：普通更新
TransitionLane     // 过渡优先级：startTransition 包裹的更新
IdleLane           // 空闲优先级：非紧急更新
```

**优先级调度机制：**

1. **高优先级打断低优先级**：
```javascript
// 用户正在输入时，低优先级的数据更新会被延后
startTransition(() => {
  setSearchResults(data); // 低优先级，可被打断
});
setInputValue(e.target.value); // 高优先级，立即响应
```

2. **同优先级批量处理**：
```javascript
// 同一事件中的多次 setState 会被合并
function handleClick() {
  setA(1); // 不会触发渲染
  setB(2); // 不会触发渲染
  setC(3); // 三次更新合并为一次渲染
}
```

3. **饥饿问题处理**：
   - 低优先级任务如果长时间被打断，会逐渐提升优先级
   - 确保所有更新最终都能被执行

**Lane 模型的优势：**

- 使用位运算，可以高效地合并、拆分、比较优先级
- 支持批量处理相同优先级的更新
- 支持优先级的动态调整

---

### 9. 什么情况下 React 会放弃复用节点，强制重新渲染？

**答案：**

React 在以下情况会放弃复用，强制销毁并重建组件：

**1. 组件类型改变：**
```javascript
// 从 div 变成 span，整个子树重建
{condition ? <div><Child /></div> : <span><Child /></span>}

// 从 ComponentA 变成 ComponentB，即使结构相似也会重建
{condition ? <ComponentA /> : <ComponentB />}
```

**2. key 改变：**
```javascript
// key 变化会强制重建，常用于重置组件状态
<UserProfile key={userId} userId={userId} />

// userId 变化时，组件完全重建，内部状态清空
```

**3. 条件渲染导致的位置变化：**
```javascript
// 不推荐：condition 变化导致 ComponentB 的位置变化
{condition && <ComponentA />}
<ComponentB />

// 推荐：使用固定结构
{condition ? <ComponentA /> : null}
<ComponentB />
```

**4. 列表中元素顺序变化且无稳定 key：**
```javascript
// 使用 index 作为 key，列表重排序会导致错误复用
{items.map((item, index) => <Item key={index} data={item} />)}
```

**5. Fragment 的特殊情况：**
```javascript
// 从单个元素变成 Fragment，或反过来
{condition ? <div /> : <><span /><span /></>}
```

**强制重建的应用场景：**

1. **重置表单状态**：切换编辑对象时，通过改变 key 清空表单
2. **动画触发**：改变 key 触发组件的进入/退出动画
3. **解决状态残留问题**：确保组件从全新状态开始

---

## 二、Hooks 深度原理

### 10. Hooks 为什么**必须**按顺序调用？内部是如何保证状态对应的？

**答案：**

Hooks 必须按顺序调用，因为 React 内部使用 **链表** 来存储每个组件的 Hooks 状态，通过 **调用顺序** 来匹配状态。

**内部存储机制：**

```javascript
// 简化的 Hooks 链表结构
fiber.memoizedState = {
  state: 'first useState value',
  next: {
    state: 'second useState value',
    next: {
      effect: { /* useEffect 信息 */ },
      next: null
    }
  }
}
```

**执行流程：**

```javascript
function MyComponent() {
  // 第一次渲染：创建链表节点
  // 后续渲染：按顺序读取链表节点
  const [a, setA] = useState(1);  // 读取第 1 个节点
  const [b, setB] = useState(2);  // 读取第 2 个节点
  useEffect(() => {});            // 读取第 3 个节点
}
```

**为什么不能条件调用：**

```javascript
function BadComponent({ condition }) {
  const [a, setA] = useState(1);     // 节点 1
  if (condition) {
    const [b, setB] = useState(2);   // 节点 2（条件为 true 时）
  }
  const [c, setC] = useState(3);     // 节点 2 或 3？

  // 当 condition 从 true 变为 false：
  // c 会错误地读取到原来 b 的状态！
}
```

**设计考量：**

- 使用调用顺序而非命名，避免了 key 冲突和命名空间污染
- 链表结构内存效率高，支持动态数量的 Hooks
- 强制顺序调用换来的是简洁的 API 和可预测的行为

**ESLint 规则：**
- `eslint-plugin-react-hooks` 的 `rules-of-hooks` 规则会在编译时检测违规

---

### 11. 常用 Hooks 有哪些？各自的使用场景是什么？

+   `useState`: 用于定义组件状态, 需要注意的是该方法在更新状态时会使用 `Object.is` 进行比较, 如果待更新状态值和当前状态值相同, 则不会进行更新, 不会引起组件的重新渲染

```js
const [state, setState] = useState(0);
setState(0); // 不会引起组件重新渲染

// 注意：对象/数组是引用比较
const [obj, setObj] = useState({ a: 1 });
setObj({ a: 1 }); // 会重新渲染（不同引用）
setObj(obj);      // 不会重新渲染（同一引用）
```

+   `useRef`: 获取 `DOM` 元素对象、记录非状态数据、获取子组件实例对象
+   `useImperativeHandle` 用于控制暴露给父组件的属性
+   `useEffect`: 让函数型组件拥有处理 `副作⽤` 的能⼒, 每次依赖项改变, 都会触发回调函数的执行, 通过它可模拟类似 `类组件` 中的部分⽣命周期, 如 `componentDidMount`、`componentDidUpdate`、`componentWillUnmount`

```js
// 触发时机：组件挂载后（即首次渲染完成后）
// componentDidMount模拟：传递一个空依赖数组 [] 给 useEffect, 这样就只会在组件挂载后执行一次
useEffect(() => {
    console.log('componentDidMount');
    // 这里可以执行一次性的初始化任务
}, []);

// 触发时机：组件更新时（即组件的 state 或 props 改变后）
// componentDidUpdate模拟：在 useEffect 中传递一个依赖数组，只有依赖项变化时，useEffect 才会触发
useEffect(() => {
    console.log('componentDidUpdate');
    // 这里可以执行依赖项变化后的任务
}, [dependency]);  // 只有当 dependency 改变时才会触发

// 触发时机：组件卸载时
// componentWillUnmount模拟：在 useEffect 中返回一个函数，这个函数会在组件卸载时执行
useEffect(() => {
    console.log('componentDidMount');
    // 执行一些操作

    return () => {
        console.log('componentWillUnmount');
        // 这里可以清理副作用，比如取消订阅或清除定时器
    };
}, []);

// useInsertionEffect(()=>{}, dependencies?)
// useLayoutEffect(()=>{}, dependencies?)
```
+   `useLayoutEffect`: 与 `useEffect` 相同, 但它会在所有的 `DOM` 变更之后同步调用
+   `useInsertionEffect`: 在任何 `DOM` 突变之前触发, 主要是解决 `CSS-in-JS` 在渲染中注入样式的性能问题


> **useEffect、useLayoutEffect、useInsertionEffect 之间的区别:**
>
> 三者执行顺序: `useInsertionEffect(DOM 变更前)` → `useLayoutEffect(DOM 变更后, 绘制前, 同步)` → `useEffect(绘制后, 异步)`

+   `useMemo`: 缓存计算结果, 适用于计算量较大的场景, 只有依赖项发生变化时才会重新计算
+   `useCallback`: 缓存函数,在依赖项不变的情况下, 不会重新创建函数, 适用于函数作为 `props` 传递给子组件时, 避免不必要的重新渲染

```js
// useMemo
useMemo(() => '返回缓存的值', [todos, tab])

// useCallback
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

+   `useReducer`: 使用简易版 `Redux`

```js
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
    const [state, dispatch] = useReducer(reducer, {age: 42});
    // ...
}
```

+   `useContext`: 获取 `context` 的值
+   `useDeferredValue`: 用于推迟更新部分 `UI`
+   `useTransition`: 允许在不阻塞 `UI` 的情况下更新状态

```js
// useDeferredValue
function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}

// useTransition
import { useTransition } from 'react';

function TabContainer() {
    const [isPending, startTransition] = useTransition();
    startTransition(()=>{
        // 延迟更新的逻辑
        // ...
    })
}
```

+   `useId`: 生成唯一 `ID`, 是 `hook` 所以只能在组件的顶层或您自己的 `Hook` 中调用它, 您不能在循环或条件内调用它、不应该用于生成列表中的键
+   `useDebugValue`: 可以在 `React DevTools` 中向自定义 `Hook` 添加一个标签, 方便追踪数据
```js
useDebugValue('dev工具显示的value');
```
+   `useSyncExternalStore`: 用于同步外部状态, 适用于 `Redux`、`Mobx` 等状态管理工具

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

**React 19 新增 Hooks**：

+   `useActionState`: 管理表单 Action 状态，自动处理 pending 状态, 用于简单的表单提交等场景
+   `useFormStatus`: 在表单子组件中获取父级 form 的提交状态,
+   `useOptimistic`: 实现乐观更新，在异步操作完成前先显示预期结果, 用于点赞/收藏等场景
+   `use`: 在渲染期间读取 Promise 或 Context（可在条件语句中使用）,可替代 useContext

```js
// useActionState 示例
const [state, formAction, isPending] = useActionState(
  async (prevState, formData) => {
    const result = await submitForm(formData);
    return result;
  },
  { message: '' }
);

// useOptimistic 示例
const [optimisticState, addOptimistic] = useOptimistic(
  state,
  (currentState, optimisticValue) => [...currentState, optimisticValue]
);
```

---

---

### 12. Ref 的原理及使用场景有哪些？

#### 作用

1.  在函数组件中, 当我们希望组件能够 `记住` 或者说 `存储` 某些信息, 但呢又不希望该信息触发新的渲染时, 就可以使用 ref 来存储
2.  用于访问真实 `DOM` 元素
3.  当父组件需要获取子组件实例对象时, 也可通过 `ref` 来实现

#### 获取真实 DOM: 三种创建方式

1.  推荐使用 `API`: `useRef`

```js
const ref = React.useRef();
<div ref={ref}></div>
```

2.  `ref` 回调函数方式

```js

const bindRef = useCallback((ele) => {
  // ele 就是当前的 DOM 元素
}, []);
<div ref={bindRef}></div>
```

3.  字符串(仅限类组件中使用)

```js
// 会自动在 this 上绑定 bodyRef, 等于当前元素
<div ref="bodyRef"></div>
```

#### 获取子组件实例

1.  子组件为类组件, 直接绑定 `ref`, 就能够拿到整个子组件的实例对象

```js
class A extends Component {}

const App = () => {
  const ref = useRef()
  return (<A ref={ref}/>)
}
```

2.  函数组件: `forwardRef` + `useImperativeHandle`

> **React 19 更新**: 函数组件可以直接接收 `ref` 作为 prop，不再需要 `forwardRef`

```js
// React 19 写法（推荐）
import React, { useState, useImperativeHandle, useRef } from 'react';

function MyComponent({ ref }) {
    const [count, setCount] = useState(0);

    useImperativeHandle(ref, () => ({
        increment: () => setCount(c => c + 1),
        reset: () => setCount(0),
    }));

    return <div>{count}</div>;
}

// React 18 及之前写法（仍兼容）
import React, { useState, forwardRef, useImperativeHandle, useRef } from 'react';

const MyComponentLegacy = forwardRef((props, ref) => {
    const [count, setCount] = useState(0);

    useImperativeHandle(ref, () => ({
        increment: () => setCount(c => c + 1),
        reset: () => setCount(0),
    }));

    return <div>{count}</div>;
});

// 使用方式（两种写法相同）
const App = () => {
    const componentRef = useRef();

    return (
        <div>
            <MyComponent ref={componentRef} />
            <button onClick={() => componentRef.current.increment()}>Increment</button>
            <button onClick={() => componentRef.current.reset()}>Reset</button>
        </div>
    );
};

export default App;

```

#### 转发 ref

> **React 19 更新**: 函数组件可以直接接收 `ref` 作为 prop，不再强制需要 `forwardRef`

1.  **React 19 推荐写法**：直接将 ref 作为 prop 接收

```js
// React 19：ref 直接作为 prop
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

// 使用
function Form() {
  const inputRef = useRef(null);
  return <Input ref={inputRef} placeholder="输入..." />;
}
```

2.  **React 18 及之前**：使用 `React.forwardRef` 转发 ref

```js
import React, { Component, forwardRef } from 'react';

// 类组件
class MyClassComponent extends Component {
    focus() {
        this.props.inputRef.focus();
    }

    render() {
        return <input ref={(ref) => (this.props.inputRef = ref)} />;
    }
}

// 使用 forwardRef 来转发 ref
const ForwardedClassComponent = forwardRef((props, ref) => {
    return <MyClassComponent inputRef={ref} />;
});

export default ForwardedClassComponent;
```

3.  使用传入props将 `ref` 进行转发(常见于类组件, 毕竟 `forwardRef` 不能用于类组件)

```js
class Cmp extends Component {
  render () {
    return (
      <div ref={this.props.innerRef}>
        1
      </div>
    );
  }
}

const bodyRef = useRef()

export default <Cmp innerRef={bodyRef} />;
```

---

### 13. useState 的更新是同步还是异步？为什么你"感觉"它是异步的？

**答案：**

useState 的更新 **本身是同步的**，但 React 的 **批处理机制** 让你"感觉"它是异步的。

**核心概念：**

```javascript
function handleClick() {
  setCount(count + 1);
  console.log(count); // 仍然是旧值！

  // 这不是因为 setCount "异步执行"
  // 而是因为 count 是一个闭包中的常量
  // 新值要等到下次渲染才能拿到
}
```

**为什么看起来是"异步"：**

1. **闭包陷阱**：函数组件中的 state 是渲染时的快照，本次渲染中不会变化
2. **批处理**：React 会将同一事件中的多次 setState 合并为一次渲染

**React 18 前后的差异：**

```javascript
// React 17：只在 React 事件处理函数中批处理
setTimeout(() => {
  setA(1); // 触发一次渲染
  setB(2); // 触发另一次渲染（共 2 次）
}, 0);

// React 18+：自动批处理（Automatic Batching）
setTimeout(() => {
  setA(1); // 不触发渲染
  setB(2); // 合并为一次渲染（共 1 次）
}, 0);
```

**获取最新值的方式：**

```javascript
// 方式 1：使用函数式更新
setCount(prev => prev + 1);

// 方式 2：使用 useRef 存储最新值
const countRef = useRef(count);
countRef.current = count;

// 方式 3：使用 useEffect 监听变化
useEffect(() => {
  console.log('count updated:', count);
}, [count]);
```

---

### 14. useEffect、useLayoutEffect 的本质区别和使用边界

**答案：**

两者的核心区别在于 **执行时机**：

```
浏览器渲染流程：
  JS 执行 → Style → Layout → Paint → Composite
                              ↑
                    useLayoutEffect 在这之前
                              ↓
                    useEffect 在这之后（异步）
```

**useEffect：**
- 在浏览器完成绑画 **之后** 异步执行
- 不阻塞浏览器渲染
- 适合大多数副作用：数据请求、订阅、日志等

**useLayoutEffect：**
- 在浏览器绑画 **之前** 同步执行
- 会阻塞浏览器渲染
- 适合需要同步读取/修改 DOM 的场景

**使用边界：**

```javascript
// useEffect：大多数场景
useEffect(() => {
  fetchData();           // 数据请求
  analytics.track();     // 埋点统计
  const sub = subscribe(); // 事件订阅
  return () => sub.unsubscribe();
}, []);

// useLayoutEffect：需要同步操作 DOM
useLayoutEffect(() => {
  // 测量 DOM 尺寸并同步更新
  const { height } = ref.current.getBoundingClientRect();
  setHeight(height);
  // 如果用 useEffect，用户会看到一帧闪烁
}, []);

// useLayoutEffect：防止视觉闪烁
useLayoutEffect(() => {
  // 根据某些条件立即修改样式
  if (shouldHide) {
    ref.current.style.display = 'none';
  }
}, [shouldHide]);
```

**性能考量：**
- 优先使用 useEffect
- 只有在出现视觉闪烁或需要同步测量时才用 useLayoutEffect
- useLayoutEffect 中的长任务会阻塞渲染，导致页面卡顿

---

### 15. useEffect 的依赖数组是如何工作的？漏依赖会带来什么问题？

**答案：**

React 使用 **Object.is** 浅比较依赖数组中的每一项，决定是否重新执行 Effect。

**依赖对比机制：**

```javascript
// 上次依赖：[1, 'hello', obj1]
// 本次依赖：[1, 'hello', obj2]

// 对比过程：
Object.is(1, 1)           // true
Object.is('hello', 'hello') // true
Object.is(obj1, obj2)     // false（引用不同）

// 结论：重新执行 Effect
```

**漏依赖的问题：**

```javascript
// 问题示例：漏掉了 count 依赖
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1); // 永远是 0 + 1 = 1
    }, 1000);
    return () => clearInterval(timer);
  }, []); // 漏掉了 count
}

// 正确做法 1：添加依赖
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(timer);
}, [count]); // 但这样每秒都会重建定时器

// 正确做法 2：使用函数式更新
useEffect(() => {
  const timer = setInterval(() => {
    setCount(prev => prev + 1); // 不依赖外部 count
  }, 1000);
  return () => clearInterval(timer);
}, []); // 空依赖是正确的
```

**常见问题模式：**

1. **函数依赖**：函数每次渲染都是新引用
```javascript
// 问题：fetchData 每次都是新函数
useEffect(() => {
  fetchData();
}, [fetchData]); // Effect 每次都执行

// 解决：用 useCallback 包裹
const fetchData = useCallback(() => { ... }, [userId]);
```

2. **对象依赖**：对象字面量每次都是新引用
```javascript
// 问题：options 每次都是新对象
useEffect(() => {
  fetch(url, options);
}, [options]); // 死循环

// 解决：解构出基本类型，或用 useMemo
useEffect(() => {
  fetch(url, { method, headers });
}, [method, headers]);
```

---

### 16. React 18+ 中 useEffect 在 StrictMode 下为什么会执行两次？

**答案：**

这是 React 18 的 **有意设计**，目的是帮助开发者发现副作用清理不当的问题，为未来的并发特性做准备。

**执行过程：**

```javascript
// StrictMode 下组件的生命周期
挂载 → 卸载 → 重新挂载

// useEffect 执行顺序
effect 执行 → cleanup 执行 → effect 再次执行
```

**设计目的：**

1. **检测副作用清理问题**：
```javascript
// 有问题的代码
useEffect(() => {
  window.addEventListener('resize', handler);
  // 忘记 return cleanup
}, []);

// 执行两次后，会注册两个事件监听器
// 帮助开发者发现遗漏的清理逻辑
```

2. **为 Offscreen API 做准备**：
- 未来 React 可能支持组件"隐藏"而非卸载
- 隐藏时需要清理副作用，显示时需要重建
- 现在的双重调用模拟了这个场景

**如何正确应对：**

```javascript
// 确保 Effect 是幂等的，cleanup 是完整的
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(setData);
  
  return () => controller.abort(); // 正确清理
}, []);

// 避免在 Effect 中做只能执行一次的操作
// 如果必须，可以用 ref 标记
const initialized = useRef(false);
useEffect(() => {
  if (initialized.current) return;
  initialized.current = true;
  // 只执行一次的逻辑
}, []);
```

**注意：**
- 这只在开发环境的 StrictMode 下发生
- 生产环境不会双重调用
- 不要试图"解决"这个问题，而是修复 Effect 的清理逻辑

---

### 17. useMemo 和 useCallback 的真实价值是什么？滥用会发生什么？

**答案：**

**useMemo**：缓存计算结果，避免重复执行昂贵计算
**useCallback**：缓存函数引用，避免子组件不必要的重渲染

**真实价值：**

```javascript
// useMemo：避免昂贵计算
const sortedList = useMemo(() => {
  return items.sort((a, b) => a.price - b.price); // 大数组排序
}, [items]);

// useCallback：配合 React.memo 避免子组件重渲染
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

<MemoizedChild onClick={handleClick} /> // 函数引用稳定，不触发重渲染
```

**滥用的问题：**

```javascript
// 反模式 1：缓存简单计算
const doubled = useMemo(() => count * 2, [count]); // 没必要

// 反模式 2：没有配合 memo 使用
const handler = useCallback(() => {}, []);
<Child onClick={handler} /> // Child 没有 memo，毫无意义

// 反模式 3：依赖项过多，频繁失效
const result = useMemo(() => compute(a, b, c, d, e), [a, b, c, d, e]);
// 任何一个变化都会重新计算，缓存命中率极低
```

**滥用的代价：**

1. **内存开销**：缓存需要额外存储空间
2. **CPU 开销**：每次渲染都要比较依赖数组
3. **代码复杂度**：增加心智负担，依赖项管理容易出错
4. **可能更慢**：对于简单计算，缓存的开销可能超过计算本身

**使用原则：**

- 先不用，发现性能问题再加
- useMemo 用于计算成本 > 比较成本的场景
- useCallback 必须配合 React.memo 才有意义
- React 19 的编译器会自动处理这些优化

---

### 18. 为什么说「useMemo 不是性能银弹」？

**答案：**

useMemo 只解决了"重复计算"这一个问题，但 React 性能问题的根源往往不在这里。

**useMemo 无法解决的问题：**

1. **组件结构问题**：
```javascript
// 状态提升过高导致的大面积重渲染
function App() {
  const [input, setInput] = useState('');
  return (
    <>
      <Input value={input} onChange={setInput} />
      <HeavyComponent /> {/* 每次输入都重渲染 */}
    </>
  );
}

// useMemo 不解决问题，需要组件拆分
function App() {
  return (
    <>
      <InputContainer /> {/* 状态下沉 */}
      <HeavyComponent />
    </>
  );
}
```

2. **Context 导致的重渲染**：
```javascript
// Context 值变化，所有消费者都会重渲染
// useMemo 缓存的值在 Context 变化时也会重新计算
```

3. **props 引用变化**：
```javascript
// 父组件重渲染时创建新对象
<Child config={{ theme: 'dark' }} /> // 每次都是新对象
// 即使 Child 用了 memo，也会重渲染
```

**更有效的优化策略：**

1. **组件拆分**：将频繁变化的状态隔离到小组件
2. **状态下沉**：将状态放到真正需要它的组件中
3. **组合模式**：用 children 传递不变的组件树
4. **Context 拆分**：将频繁变化和不常变化的数据分开

**正确心态：**

```javascript
// 不要这样想：
"我应该在哪里加 useMemo？"

// 应该这样想：
"为什么这个组件会重渲染？能否从结构上避免？"
```

---

### 19. 自定义 Hook 的设计原则是什么？哪些逻辑不适合做成 Hook？

**答案：**

**设计原则：**

1. **单一职责**：一个 Hook 只做一件事
```javascript
// 好：职责清晰
const { data, loading, error } = useFetch('/api/users');
const [value, toggle] = useToggle(false);

// 坏：职责混乱
const { data, isOpen, toggle, loading } = useUserModalWithData();
```

2. **返回值稳定**：返回的函数引用应该是稳定的
```javascript
function useCounter(initial) {
  const [count, setCount] = useState(initial);
  
  // 用 useCallback 保证函数引用稳定
  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  
  return { count, increment, decrement };
}
```

3. **参数设计合理**：必要参数在前，可选配置在后
```javascript
// 好
useFetch(url, { method: 'POST', body });

// 坏
useFetch({ url, method: 'POST', body });
```

4. **处理边界情况**：考虑组件卸载、参数变化等场景
```javascript
function useFetch(url) {
  useEffect(() => {
    const controller = new AbortController();
    fetch(url, { signal: controller.signal });
    return () => controller.abort(); // 组件卸载时取消请求
  }, [url]);
}
```

**不适合做成 Hook 的逻辑：**

1. **纯计算逻辑**：没有状态或副作用，应该是普通函数
```javascript
// 不需要 Hook
function formatPrice(price) {
  return `$${price.toFixed(2)}`;
}
```

2. **不依赖组件生命周期的逻辑**：
```javascript
// 不需要 Hook，普通工具函数即可
function debounce(fn, delay) { ... }
```

3. **一次性初始化逻辑**：
```javascript
// 不需要 Hook，在模块顶层执行
const config = loadConfig();
```

4. **只返回静态值的"Hook"**：
```javascript
// 这不是 Hook，只是普通函数
function useConstants() {
  return { MAX_SIZE: 100 }; // 没有用任何 Hook
}
```

---

### 20. 如何在 Hook 中正确处理「可取消的异步请求」？

**答案：**

处理异步请求需要解决两个核心问题：**组件卸载时取消** 和 **竞态条件**。

**方案 1：AbortController（推荐）**

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    
    setLoading(true);
    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err);
          setLoading(false);
        }
      });

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```

**方案 2：标志位（处理竞态）**

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetch(url)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) {
          setData(data); // 只有最新的请求才更新状态
        }
      });

    return () => {
      cancelled = true;
    };
  }, [url]);

  return data;
}
```

**方案 3：使用 React 19 的 use() + Suspense**

```javascript
// 配合数据获取库（如 React Query、SWR）
function UserProfile({ userId }) {
  const { data } = useSuspenseQuery(['user', userId], () => 
    fetchUser(userId)
  );
  return <div>{data.name}</div>;
}

// 父组件处理 loading 和 error
<Suspense fallback={<Loading />}>
  <ErrorBoundary fallback={<Error />}>
    <UserProfile userId={id} />
  </ErrorBoundary>
</Suspense>
```

**竞态条件示例：**

```javascript
// 用户快速切换 userId：1 → 2 → 3
// 请求发出顺序：1, 2, 3
// 响应返回顺序可能是：2, 1, 3
// 如果不处理，最终显示的是请求 3 的数据（正确）
// 但中间会闪烁显示请求 1 的数据（错误）
```

---

### 21. Hooks 闭包陷阱产生的原因？如何系统性规避？

**答案：**

**闭包陷阱的根本原因：**

函数组件每次渲染都会创建新的函数作用域，Hooks 回调中捕获的是 **创建时的变量快照**，而非最新值。

```javascript
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // 永远是 0
      setCount(count + 1); // 永远是 0 + 1
    }, 1000);
    return () => clearInterval(timer);
  }, []); // 空依赖，Effect 只执行一次
}
```

**系统性规避方案：**

**1. 函数式更新**
```javascript
setCount(prev => prev + 1); // 不依赖外部变量
```

**2. useRef 存储最新值**
```javascript
function useLatest(value) {
  const ref = useRef(value);
  ref.current = value;
  return ref;
}

function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useLatest(count);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(countRef.current); // 总是最新值
    }, 1000);
    return () => clearInterval(timer);
  }, []);
}
```

**3. 正确设置依赖**
```javascript
useEffect(() => {
  console.log(count);
}, [count]); // count 变化时重新执行
```

**4. 使用 useReducer 解耦状态逻辑**
```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  useEffect(() => {
    const timer = setInterval(() => {
      dispatch({ type: 'increment' }); // dispatch 是稳定的
    }, 1000);
    return () => clearInterval(timer);
  }, []); // 空依赖没问题
}
```

**5. 使用 useEffectEvent（React 实验性 API）**
```javascript
// 未来可能的解决方案
const onTick = useEffectEvent(() => {
  console.log(count); // 总是最新值
});

useEffect(() => {
  const timer = setInterval(onTick, 1000);
  return () => clearInterval(timer);
}, []);
```

**检测工具：**
- `eslint-plugin-react-hooks` 的 `exhaustive-deps` 规则
- 大多数闭包陷阱都会被 ESLint 检测出来

---


## 三、Concurrent Rendering & React 18+ / 19 新特性

### 22. 什么是 Concurrent Rendering？它和多线程有什么本质区别？

**答案：**

Concurrent Rendering（并发渲染）是 React 18 引入的核心能力，允许 React **同时准备多个版本的 UI**，并且渲染过程 **可以被中断、暂停、恢复或放弃**。

**与多线程的本质区别：**

| 特性 | 多线程 | Concurrent Rendering |
|------|--------|---------------------|
| 执行环境 | 多个 CPU 核心并行 | 单线程（主线程） |
| 并发方式 | 真正的并行执行 | 时间切片，交替执行 |
| 共享状态 | 需要锁机制 | 不需要，天然安全 |
| 复杂度 | 高（死锁、竞态） | 低（框架封装） |

**Concurrent Rendering 的工作原理：**

```javascript
// 概念模型：不是并行，而是可中断的协作式调度
主线程：|--渲染A--|--用户输入--|--继续渲染A--|--提交--|

// 高优先级任务可以打断低优先级任务
主线程：|--渲染列表--|-用户点击-|--渲染列表(继续)--|
                    ↑ 中断低优先级，处理用户输入
```

**并发特性的体现：**

1. **可中断渲染**：长时间渲染不会阻塞用户交互
2. **优先级调度**：紧急更新优先处理
3. **多版本 UI**：可以同时准备 Suspense 的 loading 和 content

**为什么不用 Web Worker：**
- DOM 操作只能在主线程
- 组件状态和 DOM 的同步会非常复杂
- 通信开销可能超过收益

---

### 23. startTransition / useTransition 的真实使用场景

**答案：**

startTransition 将状态更新标记为 **"过渡更新"**（非紧急），告诉 React 这个更新可以被更紧急的更新打断。

**核心概念：**

```javascript
import { startTransition, useTransition } from 'react';

// startTransition：简单用法
startTransition(() => {
  setSearchResults(filterLargeList(query)); // 非紧急更新
});

// useTransition：获取 pending 状态
const [isPending, startTransition] = useTransition();
startTransition(() => {
  setSearchResults(data);
});
// isPending 为 true 时可以显示 loading 指示器
```

**真实使用场景：**

**1. 搜索输入框 + 大列表过滤**
```javascript
function SearchableList({ items }) {
  const [query, setQuery] = useState('');
  const [filteredItems, setFilteredItems] = useState(items);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    setQuery(e.target.value); // 紧急：立即更新输入框

    startTransition(() => {
      // 非紧急：可以延迟的大列表过滤
      setFilteredItems(items.filter(item => 
        item.name.includes(e.target.value)
      ));
    });
  };

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <List items={filteredItems} />
    </>
  );
}
```

**2. Tab 切换时保持旧内容**
```javascript
function TabContainer() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const handleTabChange = (newTab) => {
    startTransition(() => {
      setTab(newTab); // 切换时保持旧 tab 内容，直到新内容准备好
    });
  };

  return (
    <>
      <TabButtons onClick={handleTabChange} />
      <div style={{ opacity: isPending ? 0.7 : 1 }}>
        <TabContent tab={tab} />
      </div>
    </>
  );
}
```

**不适合使用的场景：**
- 需要立即反馈的操作（如表单验证提示）
- 简单的状态更新（没有性能问题）
- 受控输入框的 value 更新

---

### 24. useDeferredValue 解决的是什么问题？和 debounce 有何不同？

**答案：**

useDeferredValue 返回一个 **"延迟版本"** 的值，当原值更新时，延迟值会在后台更新，不阻塞紧急渲染。

**核心区别：**

| 特性 | debounce | useDeferredValue |
|------|----------|------------------|
| 延迟机制 | 固定时间延迟 | 基于优先级调度 |
| 执行时机 | 固定等待后执行 | 有空闲时立即执行 |
| 适应性 | 固定延迟，不考虑设备性能 | 自动适应设备性能 |
| 中间状态 | 丢弃中间值 | 可以显示过渡状态 |

**使用示例：**

```javascript
function SearchResults({ query }) {
  // query 立即更新，deferredQuery 延迟更新
  const deferredQuery = useDeferredValue(query);
  
  // 当两者不同时，说明正在"过渡"
  const isStale = query !== deferredQuery;
  
  // 用 deferredQuery 计算昂贵的结果
  const results = useMemo(() => 
    filterHugeList(deferredQuery), 
    [deferredQuery]
  );
  
  return (
    <div style={{ opacity: isStale ? 0.7 : 1 }}>
      <ResultsList results={results} />
    </div>
  );
}
```

**vs debounce 的行为差异：**

```javascript
// debounce：用户停止输入 300ms 后才搜索
// 用户输入 "react"：r...e...a...c...t...(等 300ms)...搜索
// 快速设备和慢速设备体验相同

// useDeferredValue：有空闲立即处理
// 快速设备：几乎实时响应
// 慢速设备：自动延迟更多时间，但不卡顿输入
```

**适用场景：**
- 搜索结果列表的渲染
- 复杂图表的重绘
- 任何"输入框 + 昂贵计算"的场景

---

### 25. Suspense 的工作原理是什么？它解决的不是"loading"问题？

**答案：**

Suspense 的核心原理是 **捕获子组件抛出的 Promise**，在 Promise resolve 之前展示 fallback，之后展示真实内容。

**工作原理：**

```javascript
// 子组件"抛出" Promise 告诉 React "我还没准备好"
function DataComponent() {
  if (!data) {
    throw fetchPromise; // 不是 Error，是 Promise！
  }
  return <div>{data}</div>;
}

// Suspense 捕获这个 Promise
<Suspense fallback={<Loading />}>
  <DataComponent />
</Suspense>
```

**Suspense 解决的真正问题：**

不是简单的 "loading 状态管理"，而是 **声明式的异步边界** 和 **协调多个异步操作**。

```javascript
// 传统方式：命令式管理多个 loading 状态
function Page() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);
  const [loadingUser, setLoadingUser] = useState(true);
  const [loadingPosts, setLoadingPosts] = useState(true);
  
  // 大量的 if/else 判断...
}

// Suspense 方式：声明式
function Page() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <UserProfile /> {/* 内部自己获取数据 */}
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts /> {/* 内部自己获取数据 */}
      </Suspense>
    </Suspense>
  );
}
```

**Suspense 的价值：**

1. **组件职责清晰**：数据获取逻辑封装在组件内部
2. **loading 状态提升**：在任意层级统一处理 loading
3. **避免瀑布流**：配合并发数据获取，避免串行请求
4. **流式渲染支持**：SSR 中支持 streaming

**注意:** 未提供 `fallback` 时渲染为空（不再向上传递）。

---

### 26. Suspense 在数据请求和代码拆分中的差异用法

**答案：**

**代码拆分（Code Splitting）：**

```javascript
// React.lazy 配合 Suspense
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent /> {/* 按需加载 */}
    </Suspense>
  );
}

// 路由级别的代码拆分
const routes = [
  {
    path: '/dashboard',
    element: (
      <Suspense fallback={<PageLoader />}>
        <Dashboard />
      </Suspense>
    )
  }
];
```

**数据请求（Data Fetching）：**

```javascript
// 需要配合支持 Suspense 的数据获取库
// 如 React Query、SWR、Relay

// React Query 示例
import { useSuspenseQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  // 这个 hook 会"挂起"组件直到数据加载完成
  const { data } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId)
  });
  
  return <div>{data.name}</div>; // 渲染时 data 一定存在
}

// 使用
<Suspense fallback={<ProfileSkeleton />}>
  <UserProfile userId={1} />
</Suspense>
```

**关键差异：**

| 方面 | 代码拆分 | 数据请求 |
|------|----------|----------|
| 原生支持 | React.lazy 原生支持 | 需要第三方库支持 |
| 触发时机 | 组件首次渲染时 | 每次数据依赖变化时 |
| 缓存 | 浏览器自动缓存模块 | 需要库实现缓存策略 |
| 使用复杂度 | 简单 | 需要理解数据获取库 |

**组合使用：**

```javascript
// 同时拆分代码和获取数据
const Dashboard = React.lazy(() => import('./Dashboard'));

<Suspense fallback={<DashboardSkeleton />}>
  <Dashboard /> {/* 加载代码 + 内部获取数据 */}
</Suspense>
```

---

### 27. React 18 自动批处理（Automatic Batching）改变了什么？

**答案：**

React 18 将批处理扩展到 **所有场景**，不再局限于 React 事件处理函数。

**React 17 vs React 18：**

```javascript
// React 17：只在 React 事件中批处理
function handleClick() {
  setCount(c => c + 1); // 不渲染
  setFlag(f => !f);     // 不渲染
  // 事件结束后，一次性渲染（1 次）
}

// 但在 setTimeout/Promise/原生事件中不批处理
setTimeout(() => {
  setCount(c => c + 1); // 渲染！
  setFlag(f => !f);     // 又渲染！（共 2 次）
}, 0);

// React 18：所有场景都自动批处理
setTimeout(() => {
  setCount(c => c + 1); // 不渲染
  setFlag(f => !f);     // 不渲染
  // 一次性渲染（1 次）
}, 0);

// fetch 回调中也批处理
fetch('/api').then(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // 一次性渲染
});
```

**退出批处理（需要时）：**

```javascript
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCount(c => c + 1); // 立即渲染
  });
  // DOM 已更新
  
  flushSync(() => {
    setFlag(f => !f); // 再次立即渲染
  });
}
```

**实际影响：**

1. **性能提升**：减少不必要的渲染次数
2. **行为一致**：不再需要考虑"在哪里调用 setState"
3. **迁移注意**：依赖"每次 setState 都渲染"的代码可能需要调整

---

### 28. React 19 中 `use()` 是做什么的？适合用在哪？

**答案：**

`use()` 是 React 19 引入的新 Hook，可以在 **渲染期间读取 Promise 或 Context 的值**，且不受 Hook 规则限制（可以在条件语句中使用）。

**基本用法：**

```javascript
import { use } from 'react';

// 读取 Promise
function UserProfile({ userPromise }) {
  const user = use(userPromise); // 会"挂起"直到 Promise resolve
  return <div>{user.name}</div>;
}

// 读取 Context（新方式）
function ThemeButton() {
  const theme = use(ThemeContext); // 等同于 useContext(ThemeContext)
  return <button className={theme}>Click</button>;
}
```

**use() 的特殊能力：**

```javascript
// 可以在条件语句中使用（普通 Hook 不行）
function Component({ shouldFetch, dataPromise }) {
  if (shouldFetch) {
    const data = use(dataPromise); // 合法！
    return <DataView data={data} />;
  }
  return <Placeholder />;
}

// 可以在循环中使用
function CommentList({ commentPromises }) {
  return commentPromises.map((promise, i) => {
    const comment = use(promise); // 合法！
    return <Comment key={i} data={comment} />;
  });
}
```

**适合的场景：**

1. **配合 Server Components 消费数据**
```javascript
// Server Component 传递 Promise 给 Client Component
async function Page() {
  const dataPromise = fetchData(); // 不 await
  return <ClientComponent dataPromise={dataPromise} />;
}

// Client Component 用 use() 消费
function ClientComponent({ dataPromise }) {
  const data = use(dataPromise);
  return <div>{data}</div>;
}
```

2. **条件性数据获取**
3. **替代 useContext 的新写法**

**注意事项：**
- use() 读取的 Promise 必须来自稳定来源（不能在渲染中创建新 Promise）
- 需要配合 Suspense 使用

---

### 29. React 19 的 Actions / useActionState 解决了什么长期痛点？

**答案：**

Actions 解决了表单提交场景中 **手动管理 pending/error 状态** 的繁琐问题。

**传统方式的痛点：**

```javascript
// 痛点：大量样板代码
function OldForm() {
  const [name, setName] = useState('');
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsPending(true);
    setError(null);
    try {
      await submitForm({ name });
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

**React 19 Actions 方式：**

```javascript
import { useActionState } from 'react';

function NewForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState, formData) => {
      const name = formData.get('name');
      const result = await submitForm({ name });
      return result; // 返回值成为新的 state
    },
    null // 初始 state
  );

  return (
    <form action={formAction}>
      <input name="name" />
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Submit'}
      </button>
      {state?.error && <p>{state.error}</p>}
    </form>
  );
}
```

**Actions 自动处理的事情：**

1. **pending 状态**：自动跟踪请求是否进行中
2. **错误处理**：配合 Error Boundary 自动捕获错误
3. **乐观更新**：配合 useOptimistic 实现乐观 UI
4. **表单重置**：提交成功后自动重置表单
5. **渐进增强**：即使 JS 未加载也能提交表单

---

### 30. React 19 表单处理相比旧方案（Formik / 手写）的优势

**答案：**

**vs 手写表单：**

| 方面 | 手写 | React 19 Actions |
|------|------|------------------|
| pending 状态 | 手动管理 | 自动提供 |
| 错误处理 | try/catch + 状态 | 自动 + Error Boundary |
| 表单数据 | 受控组件 or FormData | 原生 FormData |
| 代码量 | 大量样板代码 | 极简 |

**vs Formik / React Hook Form：**

| 方面 | Formik 等 | React 19 Actions |
|------|----------|------------------|
| 依赖 | 第三方库 | React 内置 |
| 包体积 | 增加 ~10KB+ | 0 |
| 学习成本 | 需要学习库 API | 使用标准 HTML |
| 验证 | 内置丰富验证 | 需要自己实现 |
| 复杂表单 | 更擅长 | 简单场景更合适 |

**React 19 表单的优势：**

```javascript
// 1. 渐进增强：JS 禁用时表单仍可提交
<form action={formAction}>
  <input name="email" type="email" required />
  <button>Subscribe</button>
</form>

// 2. 乐观更新
function LikeButton() {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(likes);
  
  async function handleLike(formData) {
    addOptimisticLike(prev => prev + 1); // 立即更新 UI
    await likePost(postId); // 请求可能较慢
  }
  
  return (
    <form action={handleLike}>
      <button>Like ({optimisticLikes})</button>
    </form>
  );
}

// 3. Server Actions 无缝集成
async function createPost(formData) {
  'use server';
  await db.posts.create({ title: formData.get('title') });
  revalidatePath('/posts');
}

<form action={createPost}>
  <input name="title" />
  <button>Create</button>
</form>
```

**适用边界：**
- 简单表单：React 19 Actions 优先
- 复杂验证/动态字段/向导表单：考虑 React Hook Form

---

### 31. React 19 中 optimistic UI 的实现思路

**答案：**

Optimistic UI（乐观更新）是指 **在服务器确认之前，先假设操作会成功并更新 UI**，提升用户感知性能。

**React 19 的 useOptimistic：**

```javascript
import { useOptimistic } from 'react';

function MessageThread({ messages, sendMessage }) {
  // optimisticMessages 是乐观状态
  // addOptimisticMessage 是更新函数
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    // 更新函数：接收当前状态和新数据，返回新状态
    (currentMessages, newMessage) => [
      ...currentMessages,
      { ...newMessage, sending: true } // 标记为发送中
    ]
  );

  async function handleSend(formData) {
    const text = formData.get('text');
    const newMessage = { id: Date.now(), text };
    
    // 1. 立即更新 UI（乐观）
    addOptimisticMessage(newMessage);
    
    // 2. 发送请求
    await sendMessage(text);
    // 3. 成功后，messages prop 更新，optimisticMessages 自动同步
    // 4. 失败时需要自己处理回滚
  }

  return (
    <>
      {optimisticMessages.map(msg => (
        <Message 
          key={msg.id} 
          data={msg}
          isPending={msg.sending}
        />
      ))}
      <form action={handleSend}>
        <input name="text" />
        <button>Send</button>
      </form>
    </>
  );
}
```

**实现思路的关键点：**

1. **双状态维护**：
   - 真实状态（来自 props/服务器）
   - 乐观状态（本地临时状态）

2. **自动回滚机制**：
   - 当真实状态更新时，乐观状态自动与之同步
   - 不需要手动清理乐观更新

3. **错误处理**：
```javascript
async function handleAction() {
  addOptimistic(optimisticValue);
  try {
    await serverAction();
  } catch (error) {
    // 请求失败，需要显示错误
    // 真实状态没变，乐观状态会自动回滚
    showErrorToast(error.message);
  }
}
```

**适用场景：**
- 点赞/收藏
- 发送消息
- 添加/删除列表项
- 任何希望"即时反馈"的交互

---

## 四、组件设计 & 架构能力

### 32. 如何设计一个「高可复用、低耦合」的 React 组件？

**答案：**

设计高质量组件需要遵循以下原则：

**1. 单一职责原则**

```javascript
// 差：一个组件做太多事
function UserCard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  useEffect(() => { fetchUser(userId) }, []);
  useEffect(() => { fetchPosts(userId) }, []);
  
  return (
    <div>
      <img src={user?.avatar} />
      <h2>{user?.name}</h2>
      <PostList posts={posts} />
      <FollowButton userId={userId} />
    </div>
  );
}

// 好：职责分离
function UserCard({ user, children }) {
  return (
    <div className="user-card">
      <Avatar src={user.avatar} />
      <UserName name={user.name} />
      {children}
    </div>
  );
}

// 组合使用
<UserCard user={user}>
  <PostList posts={posts} />
  <FollowButton userId={user.id} />
</UserCard>
```

**2. Props 设计原则**

```javascript
// 好的 Props 设计
interface ButtonProps {
  // 必需的核心属性
  children: React.ReactNode;
  
  // 可选的行为属性
  onClick?: () => void;
  disabled?: boolean;
  
  // 可选的样式属性
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  
  // 允许扩展
  className?: string;
}

// 支持组合而非配置爆炸
<Button variant="primary" size="lg">
  <Icon name="save" />
  Save Changes
</Button>
```

**3. 控制反转**

```javascript
// 差：组件内部决定渲染逻辑
function List({ items }) {
  return items.map(item => <div key={item.id}>{item.name}</div>);
}

// 好：让调用者决定如何渲染
function List({ items, renderItem }) {
  return items.map(item => renderItem(item));
}

// 使用
<List 
  items={users} 
  renderItem={user => <UserCard key={user.id} user={user} />}
/>
```

**4. 状态提升的判断**

```javascript
// 状态应该放在哪里？
// 问自己：谁需要这个状态？

// 只有当前组件需要 → 组件内部
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// 多个组件需要 → 提升到共同父组件
function SearchPage() {
  const [query, setQuery] = useState('');
  return (
    <>
      <SearchInput value={query} onChange={setQuery} />
      <SearchResults query={query} />
    </>
  );
}
```

---

### 33. 组合（Composition）相比继承的优势体现在哪里？

**答案：**

React 官方推荐组合而非继承，原因如下：

**1. 灵活性更高**

```javascript
// 继承方式（不推荐）
class PrimaryButton extends Button {
  render() {
    return <button className="primary">{this.props.children}</button>;
  }
}

// 组合方式（推荐）
function Button({ variant, children, ...props }) {
  return (
    <button className={`btn btn-${variant}`} {...props}>
      {children}
    </button>
  );
}

// 使用组合可以轻松扩展
<Button variant="primary">Save</Button>
<Button variant="primary" size="large" icon={<SaveIcon />}>Save</Button>
```

**2. 避免层级爆炸**

```javascript
// 继承会导致类层级越来越深
// BaseButton → PrimaryButton → LargePrimaryButton → IconLargePrimaryButton

// 组合是扁平的
<Button variant="primary" size="large">
  <Icon name="save" />
  Save
</Button>
```

**3. children 和 slots 模式**

```javascript
// 通过 children 实现灵活的内容插入
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

// 使用
<Card>
  <CardHeader>
    <h2>Title</h2>
    <CloseButton />
  </CardHeader>
  <CardBody>Content here</CardBody>
</Card>

// 具名 slots 模式
function Layout({ header, sidebar, children }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}

<Layout
  header={<Navigation />}
  sidebar={<Menu />}
>
  <PageContent />
</Layout>
```

**4. 特化组件**

```javascript
// 通过组合创建特化组件
function Dialog({ title, children, onClose }) {
  return (
    <Modal onClose={onClose}>
      <ModalHeader>{title}</ModalHeader>
      <ModalBody>{children}</ModalBody>
    </Modal>
  );
}

function ConfirmDialog({ title, message, onConfirm, onCancel }) {
  return (
    <Dialog title={title} onClose={onCancel}>
      <p>{message}</p>
      <Button onClick={onCancel}>Cancel</Button>
      <Button variant="primary" onClick={onConfirm}>Confirm</Button>
    </Dialog>
  );
}
```

---

### 34. HOC、Render Props、Hooks 各自的适用边界

**答案：**

**三种模式的对比：**

| 特性 | HOC | Render Props | Hooks |
|------|-----|--------------|-------|
| 代码位置 | 组件外部包装 | JSX 内部 | 组件顶部 |
| 类型推导 | 困难 | 中等 | 优秀 |
| 嵌套问题 | wrapper hell | callback hell | 无 |
| 静态分析 | 困难 | 困难 | 容易 |
| 适用场景 | 跨切面关注点 | 动态渲染逻辑 | 大多数场景 |

**HOC 适用场景：**

```javascript
// 适合：跨切面关注点（认证、日志、错误边界）
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { user, loading } = useAuth();
    
    if (loading) return <Loading />;
    if (!user) return <Redirect to="/login" />;
    
    return <Component {...props} user={user} />;
  };
}

// 适合：需要包装组件的场景
const ProtectedPage = withAuth(DashboardPage);
```

**Render Props 适用场景：**

```javascript
// 适合：需要根据状态动态决定渲染内容
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handler = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
  }, []);
  
  return render(position);
}

// 使用：调用者完全控制渲染
<MouseTracker
  render={({ x, y }) => (
    <div>
      Mouse at: {x}, {y}
      <Cursor style={{ left: x, top: y }} />
    </div>
  )}
/>
```

**Hooks 适用场景（推荐首选）：**

```javascript
// 适合：几乎所有状态逻辑复用
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handler = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
  }, []);
  
  return position;
}

// 使用：清晰、可组合
function MyComponent() {
  const { x, y } = useMousePosition();
  const theme = useTheme();
  const { user } = useAuth();
  
  return <div>...</div>;
}
```

**选择建议：**
- 默认选择 Hooks
- 需要包装组件时考虑 HOC
- 需要动态渲染逻辑时考虑 Render Props

---

### 35. Context API 的工作原理？为什么容易引发性能问题？

**答案：**

**工作原理：**

```javascript
// 1. 创建 Context
const ThemeContext = React.createContext('light');

// 2. Provider 提供值
function App() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Page />
    </ThemeContext.Provider>
  );
}

// 3. Consumer 消费值
function Button() {
  const { theme } = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

**性能问题的根源：**

Context 的值变化时，**所有消费该 Context 的组件都会重新渲染**，无论它们是否使用了变化的那部分数据。

```javascript
// 问题示例
const AppContext = React.createContext();

function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  // 每次任何状态变化，value 都是新对象
  const value = { user, theme, notifications, setUser, setTheme };
  
  return (
    <AppContext.Provider value={value}>
      <Page />
    </AppContext.Provider>
  );
}

function ThemeButton() {
  // 只用了 theme，但 user 或 notifications 变化时也会重渲染！
  const { theme } = useContext(AppContext);
  return <button className={theme}>Click</button>;
}
```

**为什么会这样：**

1. Provider 的 value 使用 `Object.is` 比较
2. 每次渲染创建新对象 → 引用变化 → 所有 consumer 重渲染
3. Context 没有选择器机制，无法只订阅部分数据

---

### 36. 如何设计 Context 才不会导致大面积重渲染？

**答案：**

**1. 拆分 Context**

```javascript
// 差：一个大 Context
const AppContext = createContext({ user, theme, cart, notifications });

// 好：按职责拆分
const UserContext = createContext(null);
const ThemeContext = createContext('light');
const CartContext = createContext({ items: [] });

// 只有相关的 consumer 会重渲染
```

**2. 分离状态和更新函数**

```javascript
// 差：状态和 setter 放在一起
const CountContext = createContext({ count: 0, setCount: () => {} });

// 好：分离状态和 dispatch
const CountStateContext = createContext(0);
const CountDispatchContext = createContext(() => {});

function CountProvider({ children }) {
  const [count, setCount] = useState(0);
  
  return (
    <CountStateContext.Provider value={count}>
      <CountDispatchContext.Provider value={setCount}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}

// 只读取 count 的组件不会因为 setCount 变化而重渲染
function Display() {
  const count = useContext(CountStateContext);
  return <span>{count}</span>;
}

// 只需要更新的组件不会因为 count 变化而重渲染
function Controls() {
  const setCount = useContext(CountDispatchContext);
  return <button onClick={() => setCount(c => c + 1)}>+</button>;
}
```

**3. useMemo 稳定 value**

```javascript
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  // 使用 useMemo 避免每次渲染创建新对象
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

**4. 使用状态管理库**

```javascript
// Zustand 等库有内置的选择器机制
import { create } from 'zustand';

const useStore = create((set) => ({
  user: null,
  theme: 'light',
  setTheme: (theme) => set({ theme }),
}));

// 只订阅 theme，user 变化不会触发重渲染
function ThemeButton() {
  const theme = useStore((state) => state.theme);
  return <button className={theme}>Click</button>;
}
```

---

### 37. 受控组件 vs 非受控组件的取舍原则

**答案：**

**概念对比：**

```javascript
// 受控组件：React 控制表单值
function ControlledInput() {
  const [value, setValue] = useState('');
  return (
    <input 
      value={value} 
      onChange={e => setValue(e.target.value)} 
    />
  );
}

// 非受控组件：DOM 控制表单值
function UncontrolledInput() {
  const inputRef = useRef();
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return <input ref={inputRef} defaultValue="" />;
}
```

**选择受控组件的场景：**

```javascript
// 1. 即时验证
function EmailInput() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  
  const handleChange = (e) => {
    const value = e.target.value;
    setEmail(value);
    setError(validateEmail(value) ? '' : 'Invalid email');
  };
  
  return (
    <>
      <input value={email} onChange={handleChange} />
      {error && <span className="error">{error}</span>}
    </>
  );
}

// 2. 条件性禁用提交
function Form() {
  const [values, setValues] = useState({ name: '', email: '' });
  const isValid = values.name && values.email;
  
  return (
    <form>
      <input value={values.name} onChange={...} />
      <input value={values.email} onChange={...} />
      <button disabled={!isValid}>Submit</button>
    </form>
  );
}

// 3. 格式化输入
function PhoneInput() {
  const [phone, setPhone] = useState('');
  
  const handleChange = (e) => {
    // 自动格式化为 xxx-xxx-xxxx
    const formatted = formatPhone(e.target.value);
    setPhone(formatted);
  };
  
  return <input value={phone} onChange={handleChange} />;
}
```

**选择非受控组件的场景：**

```javascript
// 1. 文件上传
function FileUpload() {
  const fileRef = useRef();
  
  const handleSubmit = () => {
    const file = fileRef.current.files[0];
    uploadFile(file);
  };
  
  return <input type="file" ref={fileRef} />;
}

// 2. 集成非 React 代码
function RichTextEditor() {
  const editorRef = useRef();
  
  useEffect(() => {
    // 初始化第三方富文本编辑器
    new Quill(editorRef.current);
  }, []);
  
  return <div ref={editorRef} />;
}

// 3. 简单表单，无需实时验证
function SimpleSearch() {
  const inputRef = useRef();
  
  return (
    <form onSubmit={() => search(inputRef.current.value)}>
      <input ref={inputRef} defaultValue="" />
      <button>Search</button>
    </form>
  );
}
```

**取舍原则：**

| 需求 | 推荐方案 |
|------|----------|
| 即时验证 | 受控 |
| 动态表单（字段联动） | 受控 |
| 简单表单提交 | 非受控 |
| 文件上传 | 非受控 |
| 第三方库集成 | 非受控 |
| 性能敏感（大量输入框） | 非受控 |

---

## 五、性能优化

### 38. React 中常见的性能瓶颈有哪些？

**答案：**

**1. 不必要的重渲染**

```javascript
// 问题：父组件更新导致所有子组件重渲染
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <ExpensiveChild /> {/* 每次点击都重渲染 */}
    </div>
  );
}

// 解决：React.memo 或组件拆分
const ExpensiveChild = React.memo(() => {
  // 昂贵的渲染
});
```

**2. 大列表渲染**

```javascript
// 问题：渲染 10000 个列表项
function List({ items }) {
  return items.map(item => <Item key={item.id} data={item} />);
  // 初次渲染和更新都很慢
}

// 解决：虚拟列表
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  return (
    <FixedSizeList height={400} itemCount={items.length} itemSize={50}>
      {({ index, style }) => (
        <Item style={style} data={items[index]} />
      )}
    </FixedSizeList>
  );
}
```

**3. 昂贵的计算**

```javascript
// 问题：每次渲染都重新计算
function Component({ items }) {
  const sorted = items.sort((a, b) => a.price - b.price); // 每次都排序
  const filtered = sorted.filter(item => item.active);
}

// 解决：useMemo 缓存
function Component({ items }) {
  const processed = useMemo(() => {
    return items
      .filter(item => item.active)
      .sort((a, b) => a.price - b.price);
  }, [items]);
}
```

**4. 频繁的状态更新**

```javascript
// 问题：高频事件触发大量更新
function Component() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handler = (e) => {
      setPosition({ x: e.clientX, y: e.clientY }); // 每次鼠标移动都更新
    };
    window.addEventListener('mousemove', handler);
  }, []);
}

// 解决：节流或使用 ref
function Component() {
  const positionRef = useRef({ x: 0, y: 0 });
  
  useEffect(() => {
    const handler = throttle((e) => {
      positionRef.current = { x: e.clientX, y: e.clientY };
      // 只在需要时触发渲染
    }, 16);
    window.addEventListener('mousemove', handler);
  }, []);
}
```

**5. Context 导致的大范围更新**

```javascript
// 问题：Context 值变化导致所有消费者重渲染
// 见前文 Context 性能问题的讨论
```

**6. 内存泄漏**

```javascript
// 问题：未清理的订阅和定时器
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  const subscription = eventEmitter.subscribe();
  
  // 忘记 return cleanup
}, []);
```

---

### 39. 为什么 props 改变会触发重新渲染？如何阻断？

**答案：**

**原因：**

React 的渲染机制是：**当父组件渲染时，默认会渲染所有子组件**，无论 props 是否变化。

```javascript
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      {/* Child 的 props 没变，但父组件渲染了，Child 也会渲染 */}
      <Child name="fixed" />
    </div>
  );
}
```

**阻断方法：**

**1. React.memo**

```javascript
// 浅比较 props，相同则跳过渲染
const Child = React.memo(function Child({ name }) {
  console.log('Child rendered');
  return <div>{name}</div>;
});

// 自定义比较函数
const Child = React.memo(
  function Child({ user }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // 返回 true 则不重渲染
    return prevProps.user.id === nextProps.user.id;
  }
);
```

**2. 稳定的 props 引用**

```javascript
function Parent() {
  const [count, setCount] = useState(0);
  
  // 差：每次渲染创建新函数
  // <Child onClick={() => console.log('click')} />
  
  // 好：useCallback 保持稳定
  const handleClick = useCallback(() => {
    console.log('click');
  }, []);
  
  // 差：每次渲染创建新对象
  // <Child style={{ color: 'red' }} />
  
  // 好：useMemo 保持稳定
  const style = useMemo(() => ({ color: 'red' }), []);
  
  return <MemoizedChild onClick={handleClick} style={style} />;
}
```

**3. 组件拆分**

```javascript
// 差：状态提升过高
function Page() {
  const [input, setInput] = useState('');
  return (
    <>
      <Input value={input} onChange={setInput} />
      <ExpensiveList /> {/* 每次输入都重渲染 */}
    </>
  );
}

// 好：状态下沉
function Page() {
  return (
    <>
      <SearchSection /> {/* 状态在这里管理 */}
      <ExpensiveList />
    </>
  );
}

function SearchSection() {
  const [input, setInput] = useState('');
  return <Input value={input} onChange={setInput} />;
}
```

**4. children 模式**

```javascript
// 好：children 不会因为父组件更新而重建
function Parent({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      {children} {/* 引用不变，不会重渲染 */}
    </div>
  );
}

// 使用
<Parent>
  <ExpensiveChild />
</Parent>
```

---

### 40. React.memo 的对比逻辑是什么？什么时候会失效？

**答案：**

+   `React.memo` 这个 `HOC(高阶组件)`, 专门为函数组件设计, 用于性能优化

> React.memo 使用说明
>
> +   默认浅比较: 会对组件 `props` 进行 `浅比较`, 只有 `props` 变更才会触发 `render`
> +   允许传入第二参数, 该参数是个函数, 该函数接收 `2` 个参数, 两个参数分别是新旧 `props`,
> +   `arePropsEqual` 返回 `true` 时, 不会触发 `render`, 如果返回 `false` 则会, 和 `shouldComponentUpdate` 刚好与其相反

```js
// 组件
function MyComponent(props) {}

// 比较方法
function areEqual(prevProps, nextProps) {
  if (prevProps !== nextProps) {
    // 会进行渲染
    return false
  }

  // 不会进行渲染
  return true
}

export default React.memo(MyComponent, areEqual);
```

**失效场景：**

**1. 每次传入新的对象/数组**

```javascript
function Parent() {
  // 每次渲染都是新对象，memo 失效
  return <MemoChild config={{ theme: 'dark' }} items={[1, 2, 3]} />;
}

// 解决：useMemo
function Parent() {
  const config = useMemo(() => ({ theme: 'dark' }), []);
  const items = useMemo(() => [1, 2, 3], []);
  return <MemoChild config={config} items={items} />;
}
```

**2. 每次传入新的函数**

```javascript
function Parent() {
  // 每次渲染都是新函数，memo 失效
  return <MemoChild onClick={() => console.log('click')} />;
}

// 解决：useCallback
function Parent() {
  const handleClick = useCallback(() => console.log('click'), []);
  return <MemoChild onClick={handleClick} />;
}
```

**3. children 每次都是新的**

```javascript
function Parent() {
  // children 是新的 React 元素，memo 失效
  return (
    <MemoChild>
      <span>Hello</span>
    </MemoChild>
  );
}

// 解决：提升 children 或使用 useMemo
const content = <span>Hello</span>;
function Parent() {
  return <MemoChild>{content}</MemoChild>;
}
```

**4. Context 变化**

```javascript
const MemoChild = React.memo(function Child() {
  const theme = useContext(ThemeContext);
  // Context 变化时，即使 props 没变也会重渲染
  // memo 只阻止 props 导致的重渲染，不阻止 context
  return <div className={theme}>...</div>;
});
```

---

### 41. 大列表渲染的优化方案有哪些？各自的 trade-off？

**答案：**

**1. 虚拟列表（Virtualization）**

```javascript
// react-window
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  return (
    <FixedSizeList
      height={500}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}
```

| 优点 | 缺点 |
|------|------|
| 只渲染可见区域，性能极佳 | 需要预知或计算每项高度 |
| 支持超大数据量（10万+） | 搜索引擎无法索引隐藏内容 |
| 内存占用低 | 滚动太快可能出现白屏 |
| | 无法使用 Ctrl+F 搜索全部内容 |

**2. 分页（Pagination）**

```javascript
function PaginatedList({ items, pageSize = 20 }) {
  const [page, setPage] = useState(1);
  const pagedItems = items.slice((page - 1) * pageSize, page * pageSize);
  
  return (
    <>
      {pagedItems.map(item => <Item key={item.id} data={item} />)}
      <Pagination 
        current={page} 
        total={items.length} 
        onChange={setPage}
      />
    </>
  );
}
```

| 优点 | 缺点 |
|------|------|
| 实现简单 | 需要用户手动翻页 |
| SEO 友好 | 无法一眼看到全部数据 |
| 可配合服务端分页 | 跨页搜索困难 |

**3. 无限滚动（Infinite Scroll）**

```javascript
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  
  const loadMore = useCallback(() => {
    fetchItems(page).then(newItems => {
      setItems(prev => [...prev, ...newItems]);
      setPage(p => p + 1);
    });
  }, [page]);
  
  // 使用 Intersection Observer 检测滚动到底部
  const observerRef = useIntersectionObserver(loadMore);
  
  return (
    <>
      {items.map(item => <Item key={item.id} data={item} />)}
      <div ref={observerRef}>Loading...</div>
    </>
  );
}
```

| 优点 | 缺点 |
|------|------|
| 用户体验流畅 | 内存持续增长 |
| 适合移动端 | 难以直接跳转到特定位置 |
| | 浏览器后退位置难以保持 |

**4. 时间切片（Time Slicing）**

```javascript
function TimeSlicedList({ items }) {
  const [rendered, setRendered] = useState([]);
  
  useEffect(() => {
    let index = 0;
    const batchSize = 50;
    
    function renderBatch() {
      const batch = items.slice(index, index + batchSize);
      setRendered(prev => [...prev, ...batch]);
      index += batchSize;
      
      if (index < items.length) {
        requestIdleCallback(renderBatch);
      }
    }
    
    renderBatch();
  }, [items]);
  
  return rendered.map(item => <Item key={item.id} data={item} />);
}
```

| 优点 | 缺点 |
|------|------|
| 不阻塞用户交互 | 所有 DOM 最终都会创建 |
| 渐进式显示 | 不适合超大数据量 |

**选择建议：**

| 场景 | 推荐方案 |
|------|----------|
| 数据量 > 1000 | 虚拟列表 |
| 需要 SEO | 分页 / SSR + 分页 |
| 移动端 feed 流 | 无限滚动 |
| 中等数据量，需要搜索 | 分页 |

---

### 42. 如何避免「无意义的重渲染」？

**答案：**

**1. 识别重渲染来源**

```javascript
// 使用 React DevTools Profiler
// 或添加日志
function MyComponent(props) {
  console.log('MyComponent rendered', props);
  // ...
}

// 使用 why-did-you-render 库
import whyDidYouRender from '@welldone-software/why-did-you-render';
whyDidYouRender(React);

MyComponent.whyDidYouRender = true;
```

**2. 组件结构优化**

```javascript
// 问题：状态提升过高
function App() {
  const [searchTerm, setSearchTerm] = useState('');
  
  return (
    <>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <Header />      {/* 每次输入都重渲染 */}
      <Sidebar />     {/* 每次输入都重渲染 */}
      <MainContent /> {/* 每次输入都重渲染 */}
    </>
  );
}

// 解决：状态下沉
function App() {
  return (
    <>
      <SearchSection /> {/* 状态在这里 */}
      <Header />
      <Sidebar />
      <MainContent />
    </>
  );
}
```

**3. 合理使用 memo**

```javascript
// 只对「渲染成本高」且「props 经常不变」的组件使用 memo
const ExpensiveChart = React.memo(function Chart({ data }) {
  // 复杂的图表渲染
});

// 不要过度使用
// 简单组件 memo 的开销可能大于重渲染
```

**4. 稳定化 props**

```javascript
function Parent() {
  // 稳定化对象
  const config = useMemo(() => ({
    theme: 'dark',
    locale: 'en',
  }), []);
  
  // 稳定化函数
  const handleClick = useCallback((id) => {
    doSomething(id);
  }, []);
  
  // 稳定化数组
  const columns = useMemo(() => [
    { key: 'name', title: 'Name' },
    { key: 'age', title: 'Age' },
  ], []);
  
  return <Table config={config} columns={columns} onRowClick={handleClick} />;
}
```

**5. 避免在渲染中创建对象**

```javascript
// 差
<Component style={{ marginTop: 10 }} />
<Component options={{ a: 1, b: 2 }} />

// 好
const style = { marginTop: 10 }; // 组件外
<Component style={style} />

// 或使用 useMemo
const options = useMemo(() => ({ a: 1, b: 2 }), []);
```

**6. 拆分 Context**

```javascript
// 差：一个大 Context
const AppContext = createContext({ user, theme, settings, notifications });

// 好：按更新频率拆分
const UserContext = createContext(null);     // 很少变
const ThemeContext = createContext('light'); // 很少变
const NotificationContext = createContext([]); // 经常变
```

---

### 43. React Profiler 的使用思路和分析流程

**答案：**

**使用流程：**

**1. 开启 Profiler**

```javascript
// React DevTools 的 Profiler 面板
// 或代码中使用
<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>

function onRenderCallback(
  id,           // 组件树的 id
  phase,        // "mount" 或 "update"
  actualDuration, // 本次渲染耗时
  baseDuration,   // 不使用 memo 时的预估耗时
  startTime,      // 开始渲染的时间
  commitTime,     // 提交的时间
) {
  console.log({ id, phase, actualDuration, baseDuration });
}
```

**2. 分析思路**

```
步骤 1: 录制性能问题场景
        ↓
步骤 2: 找到耗时长的 commit
        ↓
步骤 3: 查看哪些组件被渲染
        ↓
步骤 4: 分析"为什么渲染"
        ↓
步骤 5: 针对性优化
```

**3. 关键指标分析**

```javascript
// Flame Graph 分析
// - 宽度 = 渲染时间
// - 颜色 = 渲染时间（黄色/红色需要关注）
// - 灰色 = 本次未渲染

// Ranked Chart 分析
// - 按渲染时间排序
// - 快速找到最耗时的组件

// Component Details
// - Why did this render?
// - Rendered at: 渲染时间点
// - Duration: 持续时间
```

**4. 常见问题诊断**

```javascript
// 问题：组件频繁渲染
// 检查：父组件是否频繁更新？props 是否稳定？

// 问题：单次渲染耗时长
// 检查：渲染逻辑是否复杂？是否有昂贵计算？

// 问题：大量组件同时渲染
// 检查：是否 Context 变化导致？是否可以拆分？
```

**5. 优化验证**

```javascript
// 优化前后对比
// 记录关键指标：
// - 渲染次数
// - 平均渲染时间
// - 最大渲染时间
// - 参与渲染的组件数

// 确保优化有效且没有引入新问题
```

---

### 44. CPU 卡顿 vs 内存泄漏在 React 中的典型表现

**答案：**

**CPU 卡顿的表现：**

```javascript
// 症状
// - 页面卡顿、掉帧
// - 用户操作响应延迟
// - 动画不流畅
// - DevTools Performance 显示长任务

// 典型原因 1：大量 DOM 操作
function BadList({ items }) {
  return items.map(item => (
    <div key={item.id}>
      {/* 10000 个复杂的列表项 */}
      <ComplexItem data={item} />
    </div>
  ));
}

// 典型原因 2：同步的昂贵计算
function ExpensiveComponent({ data }) {
  // 每次渲染都执行昂贵计算
  const processed = heavyComputation(data);
  return <div>{processed}</div>;
}

// 典型原因 3：频繁的状态更新
function RapidUpdater() {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    // 每帧都更新状态
    const handler = (e) => setPos({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
  }, []);
}
```

**内存泄漏的表现：**

```javascript
// 症状
// - 页面运行时间越长越卡
// - 标签页内存持续增长
// - 最终可能崩溃
// - DevTools Memory 显示内存不释放

// 典型原因 1：未清理的订阅/定时器
function LeakyComponent() {
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('tick');
    }, 1000);
    
    // 忘记清理！组件卸载后定时器还在运行
    // return () => clearInterval(timer);
  }, []);
}

// 典型原因 2：未清理的事件监听
function LeakyListener() {
  useEffect(() => {
    const handler = () => console.log('resize');
    window.addEventListener('resize', handler);
    
    // 忘记清理！
    // return () => window.removeEventListener('resize', handler);
  }, []);
}

// 典型原因 3：闭包持有大对象引用
function LeakyClosure() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchLargeData().then(largeData => {
      // 即使组件卸载，如果 promise 还未 resolve
      // largeData 会被闭包持有
      setData(largeData);
    });
  }, []);
}

// 典型原因 4：缓存无限增长
const cache = new Map();
function CacheComponent({ id }) {
  useEffect(() => {
    const data = fetchData(id);
    cache.set(id, data); // 缓存只增不减
  }, [id]);
}
```

**诊断方法：**

```javascript
// CPU 卡顿诊断
// 1. Chrome DevTools → Performance → Record
// 2. 查看 Main 线程的长任务（红色标记）
// 3. 展开查看具体的调用栈

// 内存泄漏诊断
// 1. Chrome DevTools → Memory → Take heap snapshot
// 2. 执行可能泄漏的操作（如多次切换路由）
// 3. 再次 Take heap snapshot
// 4. 比较两个快照，查看增长的对象
// 5. 查看 Retainers 找到持有引用的代码
```

**解决方案模板：**

```javascript
// 正确的 Effect 清理模式
useEffect(() => {
  const controller = new AbortController();
  const timer = setInterval(() => {}, 1000);
  const handler = () => {};
  
  window.addEventListener('resize', handler);
  
  fetchData({ signal: controller.signal });
  
  // 完整的清理
  return () => {
    controller.abort();
    clearInterval(timer);
    window.removeEventListener('resize', handler);
  };
}, []);
```

---

## 六、状态管理

### 45. Redux 的核心思想是什么？它解决的根本问题是什么？

**答案：**

**核心思想：**

Redux 的核心是 **可预测的状态管理**，基于三个原则：

1. **单一数据源（Single Source of Truth）**：整个应用的状态存储在一个 store 中
2. **状态只读（State is Read-Only）**：只能通过 dispatch action 来修改状态
3. **纯函数修改（Changes with Pure Functions）**：reducer 是纯函数，接收旧 state 和 action，返回新 state

```javascript
// Redux 数据流
View → dispatch(action) → Reducer → New State → View 更新

// 示例
const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// 使用
store.dispatch({ type: 'INCREMENT' });
```

**解决的根本问题：**

1. **状态共享问题**：
```javascript
// 问题：多个组件需要同一份数据
// A 和 B 组件都需要 user 数据，如何共享？

// 传统方式：props 逐层传递（props drilling）
<App>
  <Layout user={user}>
    <Sidebar user={user}>
      <UserInfo user={user} />  // 传了 3 层
    </Sidebar>
  </Layout>
</App>

// Redux：任何组件直接订阅
function UserInfo() {
  const user = useSelector(state => state.user);
}
```

2. **状态变更可追踪**：
```javascript
// 传统方式：状态可能被任何地方修改，难以追踪
user.name = 'New Name'; // 谁改的？什么时候改的？

// Redux：所有变更必须通过 action
dispatch({ type: 'UPDATE_USER_NAME', payload: 'New Name' });
// Redux DevTools 可以看到完整的 action 历史
```

3. **时间旅行调试**：因为每次状态变更都有记录，可以回放/撤销状态

---

### 46. Redux 为什么强调不可变数据？

**答案：**

**不可变数据的含义：**

```javascript
// 可变（错误）
function reducer(state, action) {
  state.count = state.count + 1; // 直接修改原对象
  return state;
}

// 不可变（正确）
function reducer(state, action) {
  return { ...state, count: state.count + 1 }; // 返回新对象
}
```

**为什么必须不可变：**

**1. 变更检测效率**

```javascript
// Redux 使用引用比较（===）检测变化
// 如果直接修改对象，引用不变，检测不到变化

const oldState = { count: 1 };
oldState.count = 2; // 引用没变
oldState === oldState; // true，React 认为没变化，不会重渲染

const newState = { ...oldState, count: 2 };
oldState === newState; // false，触发更新
```

**2. 可预测性**

```javascript
// 不可变数据确保 reducer 是纯函数
// 相同输入永远产生相同输出

function pureReducer(state, action) {
  // 不修改输入，不依赖外部状态
  return { ...state, value: action.payload };
}

// 便于测试
expect(pureReducer({ value: 1 }, { payload: 2 }))
  .toEqual({ value: 2 });
```

**3. 时间旅行调试**

```javascript
// 每次变更产生新的状态快照
// 可以保存历史状态，实现撤销/重做

const history = [
  { count: 0 },  // 初始
  { count: 1 },  // INCREMENT
  { count: 2 },  // INCREMENT
  { count: 1 },  // DECREMENT
];

// 时间旅行：直接使用历史中的某个状态
```

**4. 并发安全**

```javascript
// 如果状态可变，多个地方同时读写会出问题
// 不可变保证读取时状态不会被其他代码修改
```

**实践建议：**

```javascript
// 使用 Immer（Redux Toolkit 内置）简化不可变更新
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0, nested: { deep: 1 } },
  reducers: {
    increment: (state) => {
      // 看起来是可变写法，实际 Immer 会转换为不可变
      state.value += 1;
      state.nested.deep += 1;
    }
  }
});
```

---

### 47. Redux Toolkit 相比传统 Redux 改进了什么？

**答案：**

**传统 Redux 的痛点：**

```javascript
// 1. 样板代码太多
// actions.js
const INCREMENT = 'counter/INCREMENT';
const DECREMENT = 'counter/DECREMENT';

export const increment = () => ({ type: INCREMENT });
export const decrement = () => ({ type: DECREMENT });

// reducer.js
const initialState = { value: 0 };
export function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { ...state, value: state.value + 1 };
    case DECREMENT:
      return { ...state, value: state.value - 1 };
    default:
      return state;
  }
}

// store.js
import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

const store = createStore(
  combineReducers({ counter: counterReducer }),
  applyMiddleware(thunk)
);
```

**Redux Toolkit 的改进：**

```javascript
// 1. createSlice：自动生成 action creators 和 action types
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1; // Immer 支持"可变"写法
    },
    decrement: (state) => {
      state.value -= 1;
    },
    addAmount: (state, action) => {
      state.value += action.payload;
    }
  }
});

// 自动生成的 action creators
export const { increment, decrement, addAmount } = counterSlice.actions;

// 2. configureStore：自动配置中间件和 DevTools
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
  // 自动添加 redux-thunk
  // 自动配置 Redux DevTools
  // 自动添加不可变检测中间件（开发环境）
});

// 3. createAsyncThunk：简化异步操作
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, loading: false, error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});
```

**改进总结：**

| 方面 | 传统 Redux | Redux Toolkit |
|------|-----------|---------------|
| 代码量 | 大量样板代码 | 减少 50%+ |
| 不可变更新 | 手动展开操作符 | Immer 自动处理 |
| 异步操作 | 需要手写 thunk | createAsyncThunk |
| 类型支持 | 需要手动定义 | 自动类型推导 |
| DevTools | 需要手动配置 | 自动配置 |

---

### 48. Context + Hooks 能否完全替代 Redux？为什么？

**答案：**

**不能完全替代**，两者适用场景不同。

**Context 的局限性：**

**1. 性能问题**

```javascript
// Context 任何值变化，所有消费者都重渲染
const AppContext = createContext({ user: null, theme: 'light', cart: [] });

function ThemeButton() {
  const { theme } = useContext(AppContext);
  // 即使只用 theme，cart 变化时也会重渲染
}

// Redux 有选择器机制
function ThemeButton() {
  const theme = useSelector(state => state.theme);
  // 只有 theme 变化才重渲染
}
```

**2. 缺乏中间件机制**

```javascript
// Redux 中间件可以拦截 action
// 实现日志、异步操作、错误处理等

const loggerMiddleware = store => next => action => {
  console.log('dispatching', action);
  return next(action);
};

// Context 没有这种能力
```

**3. 缺乏开发者工具**

```javascript
// Redux DevTools 提供：
// - 完整的 action 历史
// - 时间旅行调试
// - 状态快照导出/导入
// - 性能分析

// Context 没有这些能力
```

**4. 状态更新的可预测性**

```javascript
// Redux：所有更新通过 action，可追踪
dispatch({ type: 'ADD_TO_CART', payload: item });

// Context：直接调用 setState，分散在各处
setCart([...cart, item]);
```

**Context 适合的场景：**

```javascript
// 1. 低频更新的全局配置
<ThemeContext.Provider value={theme}>
<LocaleContext.Provider value={locale}>
<AuthContext.Provider value={user}>

// 2. 组件树局部的状态共享
<FormContext.Provider value={formState}>
```

**选择建议：**

| 场景 | 推荐方案 |
|------|----------|
| 主题/语言配置 | Context |
| 用户认证状态 | Context 或 Redux |
| 复杂业务状态 | Redux/Zustand |
| 服务端状态（API 数据） | React Query/SWR |
| 表单状态 | React Hook Form |
| 需要时间旅行调试 | Redux |

---

### 49. Zustand / Jotai / Recoil 的设计思路对比

**答案：**

**Zustand：简化的 Redux 思想**

```javascript
import { create } from 'zustand';

// 创建 store（单一 store，类似 Redux）
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// 使用（内置选择器，性能优化）
function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  return <button onClick={increment}>{count}</button>;
}

// 特点：
// - 极简 API，学习成本低
// - 不需要 Provider
// - 内置选择器优化
// - 4KB 大小
```

**Jotai：原子化状态**

```javascript
import { atom, useAtom } from 'jotai';

// 定义原子（最小粒度的状态单元）
const countAtom = atom(0);
const doubleCountAtom = atom((get) => get(countAtom) * 2); // 派生原子

// 使用
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubleCount = useAtom(doubleCountAtom)[0];
  
  return (
    <div>
      <span>{count} * 2 = {doubleCount}</span>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}

// 特点：
// - 自底向上的原子设计
// - 自动依赖追踪
// - 支持异步原子
// - 类似 useState 的 API
```

**Recoil：Facebook 出品，面向大型应用**

```javascript
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil';

// Atom
const todoListState = atom({
  key: 'todoListState', // 必须唯一
  default: [],
});

// Selector（派生状态）
const filteredTodoListState = selector({
  key: 'filteredTodoListState',
  get: ({ get }) => {
    const list = get(todoListState);
    const filter = get(todoFilterState);
    return list.filter(item => item.status === filter);
  },
});

// 使用
function TodoList() {
  const [todoList, setTodoList] = useRecoilState(todoListState);
  const filteredList = useRecoilValue(filteredTodoListState);
}

// 特点：
// - 需要 RecoilRoot Provider
// - 支持复杂的依赖图
// - 支持异步 selector
// - 开发者工具支持
```

**对比总结：**

| 特性 | Zustand | Jotai | Recoil |
|------|---------|-------|--------|
| 心智模型 | 单一 store | 原子 | 原子 |
| Provider | 不需要 | 可选 | 需要 |
| 学习曲线 | 低 | 低 | 中 |
| 包大小 | ~4KB | ~8KB | ~20KB |
| TypeScript | 优秀 | 优秀 | 良好 |
| 适合场景 | 中小型应用 | 细粒度状态 | 大型应用 |
| 维护状态 | 活跃 | 活跃 | 一般 |

---

### 50. 如何避免「状态爆炸」？

**答案：**

**状态爆炸的表现：**

```javascript
// 问题：状态太多，难以管理
function ComplexComponent() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [phone, setPhone] = useState('');
  const [address, setAddress] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [selectedTab, setSelectedTab] = useState(0);
  // ... 20 more states
}
```

**解决方案：**

**1. 合并相关状态**

```javascript
// 差：分散的状态
const [name, setName] = useState('');
const [email, setEmail] = useState('');
const [phone, setPhone] = useState('');

// 好：合并为对象
const [formData, setFormData] = useState({
  name: '',
  email: '',
  phone: ''
});

// 更新
setFormData(prev => ({ ...prev, name: 'John' }));
```

**2. 使用 useReducer**

```javascript
// 复杂状态逻辑用 reducer 管理
const initialState = {
  data: null,
  loading: false,
  error: null,
};

function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, data: action.payload };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}

function Component() {
  const [state, dispatch] = useReducer(reducer, initialState);
}
```

**3. 提取自定义 Hook**

```javascript
// 封装相关的状态逻辑
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    // fetch 逻辑
  }, [url]);
  
  return { data, loading, error };
}

// 使用
function Component() {
  const { data, loading, error } = useFetch('/api/data');
}
```

**4. 使用表单库**

```javascript
// 表单状态交给专门的库管理
import { useForm } from 'react-hook-form';

function Form() {
  const { register, handleSubmit, errors } = useForm();
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name', { required: true })} />
      <input {...register('email', { required: true })} />
    </form>
  );
}
```

**5. 派生状态而非存储状态**

```javascript
// 差：存储派生数据
const [items, setItems] = useState([]);
const [filteredItems, setFilteredItems] = useState([]);
const [totalCount, setTotalCount] = useState(0);

// 好：计算派生数据
const [items, setItems] = useState([]);
const filteredItems = items.filter(item => item.active);
const totalCount = items.length;

// 如果计算昂贵，用 useMemo
const expensiveResult = useMemo(() => 
  items.filter(item => item.active),
  [items]
);
```

---

### 51. 哪些状态 **不应该** 放进全局状态管理？

**答案：**

**1. 表单输入状态**

```javascript
// 差：表单状态放全局
const { formData } = useSelector(state => state.form);

// 好：表单状态放组件本地
function MyForm() {
  const [formData, setFormData] = useState({});
  // 只在提交时与全局交互
}

// 原因：
// - 表单状态频繁变化，放全局会导致大量不必要的更新
// - 表单状态通常只有当前组件需要
```

**2. UI 状态（临时性的）**

```javascript
// 差：Modal 开关状态放全局
const { isModalOpen } = useSelector(state => state.ui);

// 好：放组件本地或提升到最近的需要它的父组件
function Page() {
  const [isModalOpen, setIsModalOpen] = useState(false);
}

// 原因：
// - 这类状态只影响局部 UI
// - 切换页面时通常需要重置
```

**3. 服务端缓存状态**

```javascript
// 差：把 API 数据放 Redux
dispatch(fetchUsers());
const users = useSelector(state => state.users);

// 好：使用专门的数据获取库
const { data: users, isLoading } = useQuery(['users'], fetchUsers);

// 原因：
// - React Query/SWR 提供缓存、重新获取、失效等功能
// - 避免自己处理复杂的缓存逻辑
```

**4. 可以从 URL 派生的状态**

```javascript
// 差：把路由参数存到全局状态
const { currentTab } = useSelector(state => state.navigation);

// 好：直接从 URL 读取
const { tab } = useParams();
// 或
const searchParams = useSearchParams();

// 原因：
// - URL 本身就是状态
// - 支持直接分享链接
// - 浏览器前进后退自动工作
```

**5. 可以从 props 派生的状态**

```javascript
// 差：从 props 复制到 state
function Child({ value }) {
  const [localValue, setLocalValue] = useState(value);
  // 现在有两个"真相来源"
}

// 好：直接使用 props
function Child({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

// 或者使用 key 重置
<Child key={userId} defaultValue={user.name} />
```

**应该放全局的状态：**

| 状态类型 | 示例 |
|----------|------|
| 用户认证信息 | 当前登录用户、权限 |
| 应用配置 | 主题、语言、功能开关 |
| 跨页面共享的业务数据 | 购物车、通知列表 |
| 需要时间旅行调试的状态 | 复杂业务流程状态 |

---

## 七、React 常用库速查

| 分类 | 推荐库 | 说明 |
|------|--------|------|
| 状态管理 | Zustand、Redux Toolkit | Zustand 轻量，RTK 适合大型应用 |
| 数据请求 | TanStack Query、SWR | 自动缓存、重试、失效管理 |
| 路由 | React Router v6+ | 声明式路由 |
| UI 组件 | Ant Design、MUI、Shadcn UI | 企业级 / Material / 可定制 |
| 样式 | Tailwind CSS | 原子化 CSS |
| 虚拟列表 | react-window | 大列表性能优化 |
| 表单 | react-hook-form | 高性能、非受控表单 |
| 动画 | Framer Motion | 声明式动画 |
| 工程化 | Vite + pnpm | 快速构建 + 高效包管理 |

---

### Zustand 极简示例

```javascript
import { create } from 'zustand';

// 1. 创建 store
const useStore = create((set) => ({
  count: 0,                                      // 定义状态
  inc: () => set((s) => ({ count: s.count + 1 })), // 定义修改状态的 action
}));

// 2. 在组件中使用
function Counter() {
  const count = useStore((s) => s.count);  // 选择器：只订阅 count，其他状态变化不触发重渲染
  const inc = useStore((s) => s.inc);      // 获取 action（引用稳定，不会导致重渲染）
  return <button onClick={inc}>{count}</button>;
}
```

---

### React Query 极简示例

```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// 1. 查询数据（自动处理 loading/error/缓存/重试）
const { data, isLoading } = useQuery({
  queryKey: ['todos'],           // 缓存键：相同 key 的请求共享缓存
  queryFn: () => fetch('/api/todos').then(r => r.json()),  // 实际请求函数
});

// 2. 修改数据
const queryClient = useQueryClient();  // 获取 queryClient 实例
const mutation = useMutation({
  mutationFn: (newTodo) => fetch('/api/todos', {  // 发送 POST 请求
    method: 'POST', 
    body: JSON.stringify(newTodo) 
  }),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });  // 成功后使缓存失效 → 自动重新请求
  },
});

// 3. 触发修改：mutation.mutate({ title: 'New Todo' })
```

---

### react-window 极简示例

```javascript
import { FixedSizeList } from 'react-window';

// 模拟大数据：10000 条
const items = Array.from({ length: 10000 }, (_, i) => `Item ${i}`);

function VirtualList() {
  return (
    <FixedSizeList
      height={400}              // 可视区域高度
      itemCount={items.length}  // 数据总条数
      itemSize={35}             // 每行固定高度
      width="100%"              // 容器宽度
    >
      {({ index, style }) => (
        // style 包含 position/top/height，必须传给子元素实现虚拟定位
        <div style={style}>{items[index]}</div>
      )}
    </FixedSizeList>
  );
}
// 原理：只渲染可视区域内的 DOM 节点（约 10-20 个），滚动时动态替换内容
```

---

### react-hook-form 极简示例

```javascript
import { useForm } from 'react-hook-form';

function Form() {
  const { 
    register,      // 注册表单字段
    handleSubmit,  // 包装提交函数（验证通过才调用）
    formState: { errors }  // 验证错误信息
  } = useForm();

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      {/* register 返回 { name, ref, onChange, onBlur }，展开后绑定到 input */}
      <input {...register('email', { required: '邮箱必填' })} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input {...register('password', { minLength: { value: 6, message: '至少6位' } })} />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit">提交</button>
    </form>
  );
}
// 特点：非受控表单，输入时不触发重渲染，性能优于受控表单
```

---
