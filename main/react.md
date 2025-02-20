## 一、 类组件生命周期

### 1.1 React v16.0 前的生命周期

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/586c05295e8f4966b560cd79a59ee975~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2648&h=1116&s=199932&e=png&b=fdfbfb)

1.  挂载阶段:

+   `constructor`(构造函数)
+   `componentWillMount`(组件将要渲染)
+   `render`(渲染组件)
+   `componentDidMount`(组件渲染完成)

2.  更新阶段: 分两种情况一种是 `state` 更新、一种是 `props` 更新

+   `componentWillReceiveProps`(组件 `props` 变更)
+   `shouldComponentUpdate`(组件是否渲染)
+   `componentWillUpdate`(组件将要更新)
+   `render`(渲染组件)
+   `componentDidUpdate`(组件更新完成)

3.  卸载阶段:

+   `componentWillUnmount`(组件将要卸载)

### 1.2 React v16.0 后的生命周期

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3d44324b51047d7ae3ba73f9766883b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2095&h=1193&s=105941&e=png&b=faf7f7)

> +   删除了几个 `will` 相关的生命周期(原因下面解释)
> +   新增了两个生命周期 `getDerivedStateFromProps` `getSnapshotBeforeUpdate`

1.  挂载阶段:

+   `constructor`(构造函数)
+   `getDerivedStateFromProps`(派生 `props`)
+   `render`(渲染组件)
+   `componentDidMount`(组件渲染完成)

2.  更新阶段:

+   `getDerivedStateFromProps`(派生 `props`)
+   `shouldComponentUpdate`(组件是否渲染)
+   `render`(渲染组件)
+   `getSnapshotBeforeUpdate`(获取快照)
+   `componentDidUpdate`(组件更新完成)

3.  卸载阶段:

+   `componentWillUnmount`(组件将要卸载)

### 1.3 getDerivedStateFromProps

`getDerivedStateFromProps` 首先它是 `静态` 方法, 方法参数分别下一个 `props`、上一个 `state`, 这个生命周期函数是为了替代 `componentWillReceiveProps` 而存在的, 主要作用就是监听 `props` 然后修改当前组件的 `state`

```js
// 监听 props 如果返回非空值, 则将返回值作为新的 state 否则不进行任何处理
static getDerivedStateFromProps(nextProps, prevState) {
  const { type } = nextProps;

  // 返回 nuyll: 对于 state 不进行任何操作
  if (type === prevState.type) {
    return null;
  }

  // 返回具体指则更新 state
  return { type }
}
```

### 1.4 getSnapshotBeforeUpdate

`getSnapshotBeforeUpdate` 生命周期将在 `render` 之后 `DOM` 变更之前被调用, 此生命周期的返回值将作为 `componentDidUpdate` 的第三个参数进行传递, 当然通常不需要此生命周期, 但在重新渲染期间需要手动保留 `DOM` 信息时就特别有用

```js
getSnapshotBeforeUpdate(prevProps, prevState){
  console.log(5);
  return 999;
}

componentDidUpdate(prevProps, prevState, snapshot) {
  console.log(6, snapshot);
}
```

打印结果:

```text
5
6 999
```

**缘由:**

+   大多数开发者使用 `componentWillUpdate` 的场景是配合 `componentDidUpdate`, 分别获取 `渲染` 前后的视图状态, 进行必要的处理, 但随着 `React` `异步渲染` 等机制的到来, `渲染` 过程可以被分割成多次完成, 还可以被 `暂停` 甚至 `回溯`, 这导致 `componentWillUpdate` 和 `componentDidUpdate` 执行前后可能会间隔很长时间, 足够使用户进行交互操作更改当前组件的状态, 这样可能会导致难以追踪的 `BUG`
    +   所以就新增了 `getSnapshotBeforeUpdate` 生命周期, 目的就是就是为了解决上述问题并取代 `componentWillUpdate`, 因为 `getSnapshotBeforeUpdate` 方法是在 `componentWillUpdate` 后(如果存在的话), 在 `React` 真正更改 `DOM` 前调用的, 它获取到组件状态信息会更加可靠
    +   除此之外, `getSnapshotBeforeUpdate` 还有一个十分明显的好处: 它调用的结果会作为第三个参数传入 `componentDidUpdate` 避免了 `componentWillUpdate` 和 `componentDidUpdate` 配合使用时将组件临时的状态数据存在组件实例上浪费内存
    +   同时 `getSnapshotBeforeUpdate` 返回的数据在 `componentDidUpdate` 中用完即被销毁, 效率更高

### 1.5 React v16.0 之后为什么要删除 Will 相关生命周期

1.  **被删除的生命周期:**

+   `componentWillReceiveProps`
+   `componentWillMount`
+   `componentWillUpdate`

2.  **删除原因:**

+   这些生命周期方法经常被误解和巧妙地误用
+   它们的潜在误用可能会在异步渲染中带来更多问题, 所以如果现有项目中使用了这几个生命周期, 将会在控制台输出如下警告! 大致意思就是这几个生命周期将在 `18.x` 彻底下线, 如果一定要使用可以带上 `UNSAFE_` 前缀

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3a37aea15174abfa4272cad2f860021~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1130&h=104&s=11864&e=png&b=342b00)

3.  **为何移除 `componentWillMount`:** 因为在 `异步渲染机制` 中允许对组件进行中断停止等操作, 可能会导致单个组件实例 `componentWillMount` 被多次调用, 很多开发者目前会将事件绑定、异步请求等写在 `componentWillMount` 中, 一旦异步渲染时 `componentWillMount` 被多次调用, 将会导致:

+   进行重复的事件监听, 无法正常取消重复的事件, 严重点可能会导致内存泄漏
+   发出重复的异步网络请求, 导致 `IO` 资源被浪费
+   补充: 现在, `React` 推荐将原本在 `componentWillMount` 中的网络请求移到 `componentDidMount` 中, 至于这样会不会导致请求被延迟发出影响用户体验, `React` 团队是这么解释的: `componentWillMount`、`render` 和 `componentDidMount` 方法虽然存在调用先后顺序, 但在大多数情况下, 几乎都是在很短的时间内先后执行完毕, 几乎不会对用户体验产生影响。

4.  **为何移除 `componentWillUpdate`:**

+   大多数开发者使用 `componentWillUpdate` 的场景是配合 `componentDidUpdate`, 分别获取 `渲染` 前后的视图状态, 进行必要的处理, 但随着 `React` `异步渲染` 等机制的到来, `渲染` 过程可以被分割成多次完成, 还可以被 `暂停` 甚至 `回溯`, 这导致 `componentWillUpdate` 和 `componentDidUpdate` 执行前后可能会间隔很长时间, 足够使用户进行交互操作更改当前组件的状态, 这样可能会导致难以追踪的 `BUG`
+   所以后面新增了 `getSnapshotBeforeUpdate` 生命周期, 目的就是就是为了解决上述问题并取代 `componentWillUpdate`, 因为 `getSnapshotBeforeUpdate` 方法是在 `componentWillUpdate` 后(如果存在的话), 在 `React` 真正更改 `DOM` 前调用的, 它获取到组件状态信息会更加可靠
+   除此之外, `getSnapshotBeforeUpdate` 还有一个十分明显的好处: 它的返回结果会作为 `componentDidUpdate` 的第三个参数进行传递, 从而避免了 `componentWillUpdate` 和 `componentDidUpdate` 配合使用时将组件临时的状态数据存在组件实例上引起的浪费内存
+   同时 `getSnapshotBeforeUpdate` 返回的数据在 `componentDidUpdate` 中用完即被销毁, 效率更高

**参考**:

+   [谈谈 React 新的生命周期钩子](https://zhuanlan.zhihu.com/p/42413419 "https://zhuanlan.zhihu.com/p/42413419")
+   [异步渲染更新](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#initializing-state "https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#initializing-state")

### 1.6 异步渲染

1.  时间分片 (`Time Slicing`):

+   `Time Slicing` 是 `Fiber` 的完全体形态, `React` 在 `渲染` 的时候, 会将任务拆分成多个小任务, 这些细分的任务则会在主线程空闲的时候进行执行, 在执行任务的期间可以随时进行暂停
+   使用时间切片的缺点是, 任务运行的总时间变长了, 这是因为它每处理完一个小任务后, 主线程会空闲出来, 并且在下一个小任务开始处理之前有一小段延迟, 但是为了避免卡死浏览器, 这种取舍是很有必要的
+   这里使用到了一个原生的 `API`, `window.requestIdleCallback()` 该方法参数是一个回调函数, 这个函数将在浏览器空闲时期被调用, 这使开发者能够在主事件循环上执行后台和低优先级工作, 而不会影响延迟关键事件, 如动画和输入响应
+   更多参考: [时间切片 (Time Slicing)](https://juejin.cn/post/6844903847249330184 "https://juejin.cn/post/6844903847249330184")

2.  悬停或者暂停 (`Suspense`): 调用 `render` 函数 -> 发现有异步请求 -> 悬停, 等待异步请求结果 -> 再渲染展示数据

+   在 `render` 函数中, 我们可以写入一个异步请求, 请求数据
+   `react` 会从我们缓存中读取这个缓存
+   如果有缓存了, 直接进行正常的 `render`
+   如果没有缓存, 那么会抛出一个 `异常`, 这个异常是一个 `promise`(很有意思, 通过抛出异常来实现)
+   当这个 `promise` 完成后(请求数据完成), `react` 会继续回到原来的 `render` 中 (实际上是重新执行一遍 `render`), 把数据`render` 出来
+   完全同步写法, 没有任何异步 `callback` 之类的东西

```js
import { Suspense } from 'react'

const Spinner = () => {}
const ProfilePage = () => {}

<Suspense fallback={<Spinner />}>
  <ProfilePage />
</Suspense>
```

> `Suspense` 的核心概念与错误边界非常相似, 错误边界能够在应用的任何地方捕捉未捕获的异常, 来处理从该组件下面抛出的所有异常。无独有偶, `Suspense` 组件捕获任何由子组件抛出的异常(`Promise`), 不同的是我们并不需要一个特定的组件来充当边界, 因为 `Suspense` 组件自己就是, 它可以让我们定义 `fallback` 来决定后备的渲染组件

## 二、虚拟 DOM

### 2.1 是什么?

> 虚拟 `DOM`: 本质上就是一个 `JS` 对象, 通过一个对象来描述了每个 `DOM` 节点的特征, 并且通过虚拟 `DOM` 就能够完整的绘制出对应真实的 `DOM`, 如下代码, 我们尝试将虚拟 `DOM` `ele` 打印出来, 看下对应的数据结构:

```js
const ele = (
  <div className='xxx'>
    111
  </div>
);

console.log(ele);
```

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e179c8a6ee204419afe54ca25a3c79f0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=790&h=235&s=11214&e=png&b=212124)

那么问题来了, 为什么 `JS` 中能够识别 `JSX` 呢? 这里其实还得多亏了 `babel`, 通过 `babel` 的 `react` 预设包(`@babel/preset-react`), 我们就可以对 `JSX` 进行转换: `JSX` 转为 `React.createElement(...)`

```js
// 转换前
const ele = (
  <div className='xxx'>
    111
  </div>
);

console.log(ele);
```

```js
// 转换后
React.createElement("div", { class: "xxx" }, "111")
```

### 2.2 虚拟 DOM 好处

> 虚拟 `DOM` 设计的核心就是用高效的 `js` 操作, 来减少低性能的 `DOM` 操作, 以此来提升网页性能, 然后使用 `diff` 算法对比新旧虚拟 `DOM`, 针对差异之处进行重新构建更新视图, 以此来提高页面性能, 虚拟 `DOM` 这让我们更关注我们的业务逻辑而非 `DOM` 操作, 这一点即可大大提升我们的开发效率

+   虚拟 `DOM` 本质上就是个对象, 对其进行任何操作不会引起页面的绘制
+   一次性更新: 当页面频繁操作时, 不去频繁操作真实 `DOM`, 而是构建新的虚拟 `DOM` 对虚拟 `DOM` 进行频繁操作, 然后一次性渲染, 这将大大提高性能(因为操作 `DOM` 比操作 `JS` 代价更大, 后面有讲)
+   差异化更新: 当状态改变时, 构建新的虚拟 `DOM`, 然后使用 `diff` 算法对比新旧虚拟 `DOM`, 针对差异之处进行重新构建更新视图, 这样也能够大大提高页面性能
+   提高开发效率: 虚拟 `DOM` 本质上就是个对象, 相对于直接操作 `DOM` 来, 直接操作对象相对来说简单又高效
+   虚拟 `DOM` 的总损耗等于 `虚拟 DOM 增删改 + diff 算法 + 真实 DOM 差异增删改 + 排版与重绘`
+   真实 `DOM` 的总损耗是 `真实 DOM 完全增删改 + 排版与重绘`
+   简单方便: 如果使用手动操作真实 `DOM` 来完成页面, 繁琐又容易出错, 在大规模应用下维护起来也很困难
+   性能方面: 使用虚拟 `DOM`, 能够有效避免真实 `DOM` 数频繁更新, 减少多次引起重绘与回流, 提高性能
+   跨平台: 虚拟 `DOM` 本质上就是用一种数据结构来描述界面节点, 借助虚拟 `DOM`, 带来了跨平台的能力, 一套代码多端运行, 比如: 小程序、`React Native`

### 2.3 缺点

+   极致性能: 在一些性能要求极高的应用中, 虚拟 `DOM` 无法进行针对性的极致优化: 因为从虚拟 `DOM` 到更新真实 `DOM` 之间还需要进行一些额外的计算(比如 `diff` 算法), 而这中间就多了一些消耗, 肯定没有直接操作 `DOM` 来得快
+   首次渲染: 首次渲染大量 `DOM` 时, 需要将虚拟树转换为实际的 `DOM` 元素, 并插入到页面中, 这个过程需要额外的计算和操作, 可能会比直接操作实际 `DOM` 更慢
+   适用度: 虚拟 `DOM` 需要在内存中创建和维护一个额外的虚拟树结构, 用于表示页面的状态。这可能会导致一定的内存消耗增加, 特别是在处理大型或复杂的应用程序时, 所以虚拟 `DOM` 更适用于动态或频繁变化的内容, 而对于静态内容 (几乎不会变化的部分), 虚拟 DOM 的优势可能不明显, 因为它仍然需要进行比较和更新的计算

### 2.4 虚拟 DOM 一定会比直接操作真实 DOM 快

+   同样的功能, 在虚拟 `DOM` 中必须需要进行更多的计算、损耗, 所以从理论上来讲虚拟 `DOM` 只会更慢, 但这里其实有个前提, 前提就是操作真实 `DOM` 的方式要做到最优, 但是单单这一点对于大部分开发人员来说其实是很难的、而且就算做到了也要耗费很多精力, 同时也会增加维护成本;

+   首次渲染或者所有节点都需要进行更新的时候, 这个时候采用虚拟 `DOM` 会比直接操作原生 `DOM` 多一重构建虚拟 `DOM` 树的操作, 这会更大的占用内存和延长渲染时间

+   对于频繁更新、删除操作: 直接操作真实 `DOM`(没有经过优化, 直接操作整个 `DOM` 树)的情况下, 虚拟 `DOM` 也行会更快, 因为相对来说操作 `DOM` 的消耗会比操作 `JS` 高

+   得失: 在构建一个实际应用的时候, 出于可维护性的考虑, 我们很难为每一个地方都去做手动优化吗, 但是呢? 虚拟 `DOM` 在不需要手动优化的情况下, 却能够给我们带来一系列的优化、同时带来更好的开发体验, 当然为此我们也只需要付出一点点性能

+   总结: 操作真实 `DOM` 如果能做到最优, 那么必然会比虚拟 `DOM` 更快, 否则结果就不好说咯


> 贴个 [babyfish-ct 大大](https://www.zhihu.com/people/babyfish-ct "https://www.zhihu.com/people/babyfish-ct") 在 [网上都说操作真实 DOM 慢, 但测试结果却比 React 更快, 为什么?](https://www.zhihu.com/question/31809713 "https://www.zhihu.com/question/31809713") 中的一个评论:
>
> +   举个例子, 一个列表, 如果要添加一个项, 那么直接 `insert` 一个新的 `DOM` 元素, 肯定最快。
> +   但是, 人性是懒惰的, 大部分人并不会直接基于原生 `DOM` 实现增量操作, 因为面向增量编程是痛苦的, 而面向全量编程是开心的。
> +   在这种懒惰的驱使下, 人们 会选择简单粗暴的办法, 把 `list` 下面所有项目清掉, 从新创建所有子项目。这样, 只是一个简单的循环, 不用考虑变化发生在什么位置。
> +   但为了一些局部变更, 把整个个列表子项全部清除再全部重建, 性能可想而知
> +   虚拟 `DOM` 的真正价值, 是把懒惰的人们喜欢的而面向全量编程, 转换为针对真实 `DOM` 的增量操作 (通过 `diff`, 找出发生变化的地方), 并保证这个过程引入的性能损失极可能低。即: 虚拟 `DOM` 以相对少的性能开销为代价, 让人们在不自不觉中以最高性能的方式操作真实 `DOM`。但和本身就坚持以最优方式操作真实 `DOM` 的程序相比, 其实它只会更慢。

### 2.5 为什么操作 DOM 会 JS 代价更大

1.  对比:

+   访问和修改 `DOM` 元素需要通过浏览器的底层接口提供的 `API` 来实现的, 与直接在内存中操作 `JavaScript` 对象相比, 通过浏览器接口进行 `DOM` 操作涉及到更多的层级和复杂性, 从而导致性能开销增加

+   `DOM` 操作引起页面重新渲染和重排, 当对 `DOM` 元素进行修改时, 浏览器需要重新计算元素的布局和样式, 并重新渲染整个页面或部分页面。这个过程称为重排 (`reflow`) 和重绘 (`repaint`), 它对于页面的性能和响应时间有一定的影响, 增加了页面的负担和性能开销


2.  为了减少对 `DOM` 操作的代价, 可以采取以下优化措施:

+   批量操作: 将多个 `DOM` 操作合并成一个批量操作, 减少页面的重排和重绘次数
+   使用文档片段 (`DocumentFragment`): 将多个 `DOM` 元素的操作放在文档片段中, 然后一次性插入到页面中, 减少页面渲染的次数
+   缓存 `DOM` 查询结果: 避免多次查询同一个 `DOM` 元素, 将查询结果缓存在变量中以提高性能。
+   使用事件委托: 将事件处理程序绑定在父元素上, 通过事件冒泡机制处理子元素的事件, 减少事件绑定的数量

> 总的来说, 由于 `DOM` 操作涉及到浏览器底层接口、页面重排和重绘等因素, 相比于操作 `JavaScript` 对象, 其代价较大, 因此, 在编写网页或应用程序时, 应尽量减少对 `DOM` 的频繁操作, 优化 `DOM` 操作的方式和时机, 以提高性能和用户体验

## 三、diff 算法

### 3.1 是什么?

`React` 在执行 `render` 过程中会产生新的虚拟 `DOM`, 在浏览器平台下, 为了尽量减少 `DOM` 的创建, `React` 会对新旧虚拟 `DOM` 进行 `diff` 算法找到它们之间的差异, 尽量复用 `DOM` 从而提高性能; 所以 `diff` 算法主要就是用于查找新旧虚拟 `DOM` 之间的差异

那么请问可以不做 `diff` 算法, 每次 `render` 都重新创建新的 `DOM` 是否可以? 当然没有问题, 但重点在于 `DOM` 创建的性能成本很高, 如果不做 `DOM` 的复用, 那性能就太差了, `diff` 算法的目的就是对比两次渲染结果, 找到可复用的部分, 然后剩下的该删除删除, 该新增新增

[传统diff算法是通过循环递归对树节点进行依次对比, 效率比低下, 算法复杂度达到 `O(n^3)`](tradition_diff.md), 而在 `React` 中针对该算法进行一个优化, 复杂度能达到 `O(n)`

参考: [复杂度理解](complexity.md)

### 3.2 diff 策略

1.  `tree` 层级(同层级比较): 考虑到在实际 `DOM` 操作中需要跨层级操作的次数很少很少, 所以在进行 `diff` 操作时只会对 `同一层级` 进行比较, 这样只需要对树遍历一次就 `OK` 了, 如下图, `react` 会按同层级进行比较, 发现新树中 `R` 节点下没有了 `A`, 那么直接删除 `A` 在 `D` 节点下创建 `A` 以及下属所有节点

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfeb0ddd14dd402da6a062a0cf254287~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1364&h=741&s=15418&e=png&b=f9fafb)

2.  `component` 层级: 如果是同一个类型的组件, 则会继续往下 `diff` 运算, 如果不是一个类型组件, 那么将直接删除这个组件下的所有子节点, 然后创建新的 `DOM`, 如下图所示, 当 `D` 类型组件换成了 `G` 后, 即使两者的结构非常类似, 也会将 `D` 类型的组件删除再重新创建 `G`

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/358fc56dcd14482bbaee60c9d477aecf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1303&h=459&s=71493&e=png&b=ffffff)

3.  `element` 层级: 是同一层级的节点的比较规则, 根据每个节点在对应层级的唯一 `key` 作为标识, 并且对于同一层级的节点操作只有 `3` 种, 分别为 INSERT\_MARKUP(插入)、MOVE\_EXISTING(移动)、REMOVE\_NODE(删除)

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0d82acd7182433b85ecd29238556f26~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1452&h=673&s=80656&e=png&b=fefefe)

如上场景比较规则: 通过 key 发现新旧集合中的节点都是相同的节点, 因此无需进行节点删除和创建, 只需要将旧集合中节点的位置进行移动, 更新为新集合中节点的位置即可, 判断伪代码如下, 参考资料查看 [这里](https://vue3js.cn/interview/React/diff.html#%E4%BA%8C%E3%80%81%E5%8E%9F%E7%90%86 "https://vue3js.cn/interview/React/diff.html#%E4%BA%8C%E3%80%81%E5%8E%9F%E7%90%86")

```js
const old = ['a', 'b', 'c', 'd']
const newList = ['b', 'a', 'd', 'c']

let maxIndex = 0

newList.forEach((v, index) => {
  const oldIndex = old.indexOf(v)

  maxIndex = Math.max(oldIndex, maxIndex)

  if (oldIndex < maxIndex) {
    // 移动: 将 v 节点移动到 index 处
    console.log(index, v)
  }
})
```

### 3.3 注意事项

1.  `key` 的值必须保证 `唯一` 且 `稳定`, 有了 `key` 属性后, 就可以与组件建立了一种对应关系, `react` 根据 `key` 来决定是销毁还是重新创建组件, 是更新还是移动组件

2.  `index` 的使用存在的问题: 大部分情况下可能没有啥问题, 但是如果涉及到数据变更(更新、新增、删除), 这时 `index` 作为 `key` 会导致展示错误的数据, 其实归根结底, 使用 `index` 的问题在于两次渲染的 `index` 是相同的, 所以组件并不会重新销毁创建, 而是直接进行更新

3.  下面写法的问题: 每次 `render` 时 `Com` 都重新声明, 导致在进行 `diff` 时 `Com` 都会被认为是新的组件, 需要被销毁、重新创建


```js
const App = () => {
  const Com = () => (<div>3</div>)

  return (
    <div>
      <div>1</div>
      <div>2</div>
      <Com />
      <div>4</div>
    </div>
  )
}
```

## 四、Render 相关

1.  类组件 `render` 函数返回 `JSX`

```js
class Foo extends React.Component {
  render() {
    return <h1> Foo </h1>;
  }
}
```

2.  函数直接组件 `return` 出 `JSX`

```js
function Foo() {
  return <h1> Foo </h1>;
}
```

3.  在 `React` 中, 我们会通过 `babel` 将我们会编写的 `jsx` 转化成我们熟悉的 `js` 格式, 这里会用到一个 `babel` 中 `react` 的预设 `@babel/preset-react`

```js
// 编译前
return (
  <div className='cn'>
    <Header> hello </Header>
    <div> start </div>
    Right Reserve
  </div>
)

// 编译后
return (
  React.createElement(
    'div',
    {
      className : 'cn'
    },
    React.createElement(
      Header,
      null,
      'hello'
    ),
    React.createElement(
      'div',
      null,
      'start'
    ),
    'Right Reserve'
  )
)
```

4.  我们都知道如果在 `js` 文件中写了 `jsx`, 就需要再顶部引入 `React`, 而之所以要引入 `React` 从上面 👆🏻 编译结果也能看出来, `JSX` 将会被编译为 `React.createElement` 如果不引入将会报错(`React` 未定义)

5.  `React 17` 不再需要引入在组件中显式地引入 `React` 这又是为什么呢?


+   `React` 更新引入了 `react/jsx-runtime`, 改变了 `JSX` 编译模式, 不再是 `React.createElement`

```js
_jsx('h1', { children: 'Hello world' });
```

+   同时编译工具(`react` 的预设 `@babel/preset-react`), 针对 `jsx` 不但会帮我们进行编译, 还会帮我们手动引入所需要的包

```js
// 由编译器引入（禁止自己引入！）
import { jsx as _jsx } from 'react/jsx-runtime';

function App() {
  return _jsx('h1', { children: 'Hello world' });
}
```

+   那早期版本是不是更新了 `@babel/preset-react` 也可以不需要手动引入? 不可以, 因为这里是使用新的编译方式, 旧的版本并不支持

6.  渲染流程

+   `state` 或者 `props` 更新, 会触发 `render`, 当然这里也有例外(`props` 可通过 `shouldComponentUpdate`、`memo` 进行控制, 并且在 `useState` 中如果设置了相同的 `state` 也不会触发 `render`)
+   每次 `render` 时, 整个 `UI` 都将以 `虚拟 DOM` 的形式进行呈现
+   使用 `diif` 算法, 计算新旧 `虚拟 DOM` 对象之间的差异
+   计算完成, 将只更新实际更改的真实 `DOM` 节点

## 五、React 事件机制

### 5.1 原生事件和 React 事件监听方法:

+   `React` 事件通过 `JSX` 方式绑定的事件, 比如 `onClick={() => this.handle()}`
+   原生事件使用 `addEventListener`

```js
const ref = useRef()
const onClick = useCallback(() => {
}, []);

useEffect(() => {
  // 绑定原生事件
  ref.current.addEventListener('click', event => {});
}, []);

return (
  <div
    ref={ref}
    onClick={onClick} // React 事件
  />
);
```

### 5.2 合成事件

如下代码 `e` 就是所谓的合成事件, 它并不是原生的一个 `事件对象`, 而是 `React` 根据 `W3C` 规范定义出来的一个合成事件, 所以使用合成事件对象我们就不需要担心浏览器的兼容性问题了, 同时如果我们想要访问原生的事件对象, 可通过 `nativeEvent` 属性来获取

```js
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
  }
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```

补充: 从 `v0.14` 开始, 事件处理函数, 返回 `false` 时, 不再阻止事件传递, 这里需要手动调用 `e.stopPropagation()` 或 `e.preventDefault()` 作为替代方案

### 5.3 对原生事件的升级和改造

1.  react 在给注册事件的时候也是对浏览器兼容性处理

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6892d10575ae4e8c8b015405ffa84d21~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=755&h=596&s=119814&e=png&b=fefdfd)

2.  对于有些 `dom` 元素事件, 我们进行事件绑定之后, `react` 并不是只绑定处理我们所声明的事件类型, 还会额外的增加一些其他的事件, 帮助我们提升交互的体验, 这里就举一个例子来说明下:

> 我们都知道, 在原生事件中对于 `input` 我们如果只绑定 `onchange` 事件, 那么在持续输入时是无法触发该事件的, 只有在失去焦点时才会触发该事件! 但这个大部分情况下并不是一个好的体验! 所以在 `React` 中我们如果为 `input` 绑定 `onChange` 事件, 实际上 `React` 并不是只注册了 `onchange` 事件, 还会帮我们添加额外的事件, 做很多处理, 来弥补这个缺陷, 使得我们在每次输入内容时都能够正确触发 `onChange` 事件

```js
import React, { useRef, useEffect } from 'react';

export default () => {
  const inputRef = useRef();

  useEffect(() => {
    const handler = (e) => {
      console.log('手动绑定:', e.target.value);
    };

    inputRef.current.addEventListener('change', handler);

    return () => document.removeEventListener('change', handler);
  }, []);

  return (
    <input
      ref={inputRef}
      onChange={(e) => console.log('React 绑定事件: ', e.target.value)}
    />
  );
};
```

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bccddc8946845e5875cfacf4700da25~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1488&h=453&s=13251&e=png&b=202124)

### 5.4 事件注册机制

1.  通过 `事件委托` 的方式, 将所有事件都绑定在了 `document` 来进行统一处理
2.  每次绑定都会将事件处理函数, 存储起来

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72f61f70149b46f4abf2ca2eff2260db~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=763&h=523&s=31945&e=png&b=f3f1f4)

3.  问: 对于同一个 `DOM` 分别绑定原生事件、合成事件, 在原生事件中阻止事件冒泡为什么会阻止合成事件的执行？

> 答: 合成事件是`事件委托`的一种实现, 主要是利用`事件冒泡`机制将所有事件在 `document` 进行统一处理, 根据 `事件流`, 事件执行顺序为 `捕获阶段`、`目标阶段`、`冒泡阶段`, 当我们在`原生事件`上阻止`事件冒泡`, 那么事件就无法冒泡到 `document`, 那么合成事件自然无法执行！

```js
const ref = useRef()

const onClick = event => {
  event.stopPropagation();
  console.log('[ 合成事件 ]', event);
};

useEffect(() => {
  ref.current.addEventListener('click', event => {
    event.stopPropagation();
    console.log('[ 原生事件 ]', event);
  });
}, []);

return (
  <div 
    ref={ref} 
    onClick={onClick} 
  />
);
```

补充: 会先执行原生事件，然后处理 `React 事件` 原生事件(阻止冒泡)会阻止合成事件的执行 合成事件(阻止冒泡)不会阻止原生事件的执行 所以两者最好不要混合使用, 避免出现一些奇怪的问题

4.  问: `React` 为什么要将所有事件绑定在 `document` 上, 这么做有什么优缺点吗?

**优点:**

+   减少事件注册, 减少内存消耗, 提升性能, 不需要注册那么多的事件了, 一种事件类型只在 `document` 上注册一次即可; 举个例子, 若有 `10w` 项列表, 点击列表某一项要提示这一列表的某个信息, 若在每一个 `li` 节点挂载事件, `10w` 个事件将会极大程度上拖慢你的浏览器性能
+   统一处理, 并提供合成事件对象, 抹平浏览器的兼容性差异

**缺点:** 如果层级过多, 冒泡过程中可能会被某层给阻止掉

5.  从 `v17.0.0` 开始, `React` 不再将事件处理添加到 `document` 上, 而是将事件处理添加到渲染 `React` 树的根容器中这又是为什么呢?

+   如果页面上有多个 `React` 版本, 事件都会被附加在 `document` 上, 这时嵌套的 `React` 树调用 `e.stopPropagation()` 停止了事件冒泡, 外部的树仍会接收到该事件(因为只是阻止了 `React` 事件的冒泡), 这就使嵌套不同版本的 `React` 难以实现

+   如果你系统只用了一个 `react` 版本, 那没啥区别; 但有些复杂的系统, 由于历史原因, 或者用了微前端, 它就同时用很多个版本的 `react`, 这就不一样了, 如果很多个版本的 `react`, 都往 `document` 上去绑定, 就容易出现混乱


## 六、Fiber

### 6.1 缘由

1.  首先 `React` 组件的渲染主要经历两个阶段:

+   调度阶段(`Reconciler`): 这个阶段 `React` 用新数据生成新的虚拟 `DOM`, 遍历虚拟 `DOM`, 然后通过 `Diff` 算法, 快速找出需要更新的元素, 放到更新队列中去
+   渲染阶段(`Renderer`): 这个阶段 `React` 根据所在的渲染环境, 遍历更新队列, 将对应元素更新(在浏览器中, 就是更新对应的 `DOM` 元素)

2.  对于调度阶段, 新老架构中有不同的处理方式:

+   早期 `16` 之前 `React` 在 `diff` 阶段是通过一个`自顶向下递归算法`, 来查找需要对当前 `DOM` 进行更新或替换的操作列表, 一旦开始, 会`持续占用主线程`, 很难被中断, 当虚拟 `DOM` 特别庞大的时候, 主线程就被长期占用, 页面的交互、布局、渲染会被停止, 造成页面的卡顿, **这里举个例子:** 假设更新一个组件需要 `1ms`, 如果有 `200` 个组件要更新, 那就需要 `200ms`, 在这 `200ms` 的更新过程中, 浏览器唯一的主线程都在专心运行更新操作, 无暇去做任何其他的事情。想象一下，在这 200ms 内，用户往一个 input 元素中输入点什么，敲击键盘也不会获得响应，因为渲染输入按键结果也是浏览器主线程的工作，但是浏览器主线程被 React 占用，抽不出空，最后的结果就是用户敲了按键看不到反应，等 React 更新过程结束之后，那些按键会一下出现在 input 元素里，这就是所谓的界面卡顿。
+   `Fiber` 是 `React 16` 中采用的新的调度处理方法, 主要目标是支持虚拟 `DOM` 的一个渐进式渲染

### 6.2 Fiber 的设计思路

> 因为浏览器的页面是一帧一帧绘制出来的, 当每秒绘制的帧数(`FPS`)达到 `60` 时, 页面是流畅的, 小于这个值时, 用户会感觉到卡顿; 转换成时间就是 `16ms(1000 / 60)` 内如果当前帧内执行的任务没有完成, 就会造成卡顿;

1.  `Fiber`: 是实现了一个基于优先级和 事件循环中`requestIdleCallback`(执行的前提条件是当前浏览器处于空闲状态) 的一个循环 `任务调度` 算法, 他在渲染虚拟 `DOM`、 `diff` 阶段将任务拆分为多个小任务、这样的话就可以随时进行中止和恢复、同时又根据每个任务的优先级来执行任务

2.  `Fiber` 是把 `render/update` 分片, 拆解成多个小任务来执行(`时间切片`), 每次只检查树上部分节点, 做完此部分后, 若当前一帧 (`16ms`) 内还有足够的时间就继续做下一个小任务, 时间不够就停止操作, 等主线程空闲时再恢复执行

3.  `Fiber` 是根据一个 `fiber` 节点 (`VDOM` 节点) 进行来拆分, 以 `fiber node` 为一个任务单元, 一个组件实例都是一个任务单元, 任务循环中, 每处理完一个 `fiber node`, 可以中断/挂起/恢复

4.  `双缓冲`渲染,React 会同时维护两个 Fiber 树：一个是正在渲染的树（current fiber tree），另一个是即将要渲染的树（work-in-progress fiber tree）,允许 React 在渲染时不中断 UI 更新，而是平滑过渡。

5. 不同的任务分配不同的优先级, `Fiber` 根据任务的优先级来动态调整任务调度, 先做高优先级的任务


+   `Immediate`: 最高优先级, 不中断立即执行，通常是与用户交互相关的任务
+   `High`: 稍后执行，通常是动画或者动画帧更新等
+   `Normal`: 普通等级的, 常规的渲染任务/网络请求等
+   `Low`: 低优先级的, 这种任务可以延后, 但最后始终是要执行的
+   `Idle`: 最低等级的任务, 可以被无限延迟的, 比如 `console.log()`

6. `优先级的动态变化`, react18的调度器会根据任务的紧急性和系统的负载情况动态调整任务的优先级。高优先级任务会打断低优先级任务的执行,且当调度器发现一些低任务由于长时间未执行可能影响用户体验时，会主动对任务的优先级进行提升,从而得到执行.


### 6.3 带来的影响

由于 `Fiber` 采用了全新的调度方式, 任务的更新过程可能会被打断, 这意味着在组件更新过程中, `render` 及其下面几个生命周期函数可能会被调用多次, 所以这几个生命周期函数中不应出现副作用:

> 同时考虑到 `componentWillMount` `componentWillReceiveProps` `componentWillUpdate` 这几个生命周期经常被误用, 所以干脆就废弃了, 同时新增了几个生命周期用于替代(这里具体可参考上文中, 生命周期部分)

+   shouldComponentUpdate
+   componentWillMount(UNSAFE\_componentWillMount)
+   componentWillReceiveProps(UNSAFE\_componentWillReceiveProps)
+   componentWillUpdate(UNSAFE\_componentWillUpdate)

### 6.4 React 调度流程图

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b82de163473a433d8d8f0c920776191d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=924&h=1545&s=47333&e=png&b=fdfdfd)

### 6.5 参考

+   [「React Fiber」 详细解析](https://zhuanlan.zhihu.com/p/424967867 "https://zhuanlan.zhihu.com/p/424967867")
+   [React Fiber 的作用和原理](https://cloud.tencent.com/developer/article/1882296 "https://cloud.tencent.com/developer/article/1882296")
+   [面试官: 说说对Fiber架构的理解? 解决了什么问题?](https://vue3js.cn/interview/React/Fiber.html#%E4%BA%8C%E3%80%81%E6%98%AF%E4%BB%80%E4%B9%88 "https://vue3js.cn/interview/React/Fiber.html#%E4%BA%8C%E3%80%81%E6%98%AF%E4%BB%80%E4%B9%88")
+   [React Fiber技术解读: 你需要知道面试官最关心的话题!](https://juejin.cn/post/7243450433812070455 "https://juejin.cn/post/7243450433812070455")

## 七、React State 那些事

### 7.1 是什么

一个组件的显示形态, 可以由 `内部状态` 和 `外部参数` 所决定, `外部参数` 指的则是 `props` 而 `内部状态` 则是 `state`, 同时需要注意的是只有通过 `setState` 或者 `useState` 中指定的方法修改状态才会触发 `render`

```js
export default () => {
  const [count, setCount] = useState(1);

  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]);

  return (
    <div onClick={handleClick}>
      {count}
    </div>
  );
};
```

注意的是, `setState` 或者 `useState` 中修改状态的方法, 它们的第一个参数还可以是一个函数, 函数的参数是当前的状态, 同时函数的返回值将最为新的状态值

```js
// 类组件
this.setState((pre) => ({ count: pre.count + 1 }));

// 函数组件
const [count, setCount] = useState(1);
setCount((pre) => (pre + 1));
```

同时, `setState` 还有第二个参数, 当状态更新后, 并且组件已经重新渲染的时候会被调用, 一般用于获取修改后的状态

```js
handleClick = () => {
  this.setState(
    (pre) => ({ count: pre.count + 1 }),
    () => {
      // 获取修改后的状态
      this.preState = this.state;
    },
  );
};
```

### 7.2 React 的更新机制: 异步 OR 同步

1.  常见答案:

+   在组件生命周期或 `React` 事件中, `setState` 是异步
+   在 `setTimeout/setInterval` 或者原生 `dom` 事件中, `setState` 是同步

2.  本质上来讲 `setState` 是同步的, 之所以出现异步的假象是因为要进行 `状态合并` 或者说是 `批处理`, 需要等生命周期、事件处理器执行完毕, 再批量修改状态! 当然在实际开发中, 在合成事件和生命周期函数里, 完全可以将其视为异步的

3.  `setState` 机制:


+   在 `React` 的 `setState` 函数实现中, 会根据一个变量 `isBatchingUpdates` 判断是直接更新 `this.state` 还是放到队列中回头再说
+   `isBatchingUpdates` 默认是 `false`, 当 `React` 在执行生命周期或调用事件处理函数之前会将其设置为 `true`, 当执行完生命周期或者事件处理函数再改为 `false` 然后才会一起更新状态、更新组件, 所以整个过程看起来像异步的

4.  当然实际开发中如果需要, 我们可以通过第二个参数 `setState(partialState, callback)` 中的 `callback` 拿到更新后的结果

5.  在原生事件中, 由于不会调用 `React` `批处理机制`, 所以 `isBatchingUpdates` 一直是 `false`, 所以如果调用 `setState` 会直接更新 `this.state`, 整个过程看起来就像是同步

6.  那么在 `setTimeout/setInterval` 中又为什么看起来像同步的呢? 这里主要和微任务和宏任务有关, 如下是个演示代码, `setTimeout` 里面回调会等到, 主体代码执行完才会执行, 这时 `isBatchingUpdates` 已经是 `false`, 这时执行 `setState` 后会直接修改 `this.state`, 所以整个过程看起来就像是同步


```js
isBatchingUpdates = true

// 即便延时为 0 也要主体代码全部执行完, 才会执行回调函数里面的代码, 这时 isBatchingUpdates 已经被改为 false, 
setTimeout(() => {
  this.setState({
    count: this.state.count + 1
  })
}, 0)

isBatchingUpdates = false
```

7.  **补充说明:** `React 18` 中, 实现自动批处理, 所以不管任何情况所有状态的修改都会进行批处理了, 简单理解就是在 `React 18` 所以状态的修改都将是 `"异步"` 的, 具体参考 [自动批处理](#92-自动批处理)

### 7.3 为什么要设计成异步(批处理)

> 参考: [React 中 setState() 为什么是异步的?](https://juejin.cn/post/6844903557200609287#heading-2 "https://juejin.cn/post/6844903557200609287#heading-2")

1.  保证 `state` 和 `props` 的一致性

+   `props` 必然异步, 因为只有因为当父组件重渲染了我们才知道 `props` 是啥
+   那么保证 `props` 和 `state` 一致性就很重要了, 因为实际开发中我们经常会将状态提升到父组件, 和兄弟组件进行共享, 这时如果 state 和 props 表现不一致那么这个操作很大概率就会引起一些 bug
+   所以 `React` 更愿意保证内部的一致性和状态提升的安全性, 而不总是追求代码的简洁性

2.  提高性能: 在渲染前会有意地进行 `等待`, 直到所有在组件的事件处理函数内调用的 `setState()` 完成之后, 统一更新 `state`, 这样可以通过避免不必要的重新渲染来提升性能

3.  提升渲染性能: 当切换当新页面, 通过 `setState` 异步, 让 `React` 幕后渲染页面,平滑过渡, 提升用户体验


## 八、高阶组件

> 高阶组件: 是 `React` 中用于复用组件逻辑的一种技巧, 是一种基于 `React` 特性而形成的设计模式

### 8.1 简述

1.  本质: 本质上就是一个函数, 是一个参数为组件, 返回值为新组件的函数
2.  高阶组件内部实现方式:

+   属性代理: 创建新组件并渲染传入的组件, 通过 `props` 属性来为组件添加值或方法
+   反向继承: 通过继承方式实现, 继承传人的组件, 然后新增一些方法、属性

```js
// 方法一
const hoc1 = (Com) => {
  class NewCom extends Component {
    state = { count: 0 };

    updateCount = (count) => {
      this.setState({ count });
    };

    render () {
      return (
        <Com
          {...this.props}
          count={this.state.count}
          updateCount={this.updateCount}
        />
      );
    }
  }

  return NewCom;
};

// 方法二, 继承
const hoc2 = (Com) => {
  class NewCom extends Com {
    updateCount = (count) => {
      this.setState({ count });
    };
  }

  return NewCom;
};

```

3.  调用方式:

+   `@`修饰符
+   直接调用

```js
// 使用修饰符
@hoc
class App extends {}


// 直接调用
class App extends {}
const AppUseHoc = hoc(App)
```

### 8.2 作用

1.  强化 `props`: 类似 `withRouter` 为组件添加 `props` 属性, 强化组件功能
2.  为组件添加事件: 为传入的组件包裹一层, 并绑定事件
3.  劫持控制渲染逻辑: 通过反向继承方式, 拦截原组件的生命周期、渲染、内部组件状态...
4.  添加loading/错误处理组件等

### 8.3 hooks 能取代 hoc 高阶组件吗？

完全替代是不能的(因为高阶组件被滥用了):

1.  官方给出的答案是可以替代的, 因为高阶组件的出现主要目的就是为了复用状态相关逻辑(强化 `props`), 在这块 `hooks` 是可以完全替代的, 而还有其独到的优势
2.  但是后来高阶组件除了用于逻辑的复用还被滥用:

+   在内部实现动态渲染, 根据 `props` 动态渲染: 这个完全可以通过组件的方式来实现, 组件在渲染上拥有更高的自由度, 可以根据父组件提供的数据进行动态渲染
+   通过继承拦截生命周期、或者篡改 `props`: 本身就不应该这么做, 容易出现各种问题 ...

### 8.4 缺点

1.  属性代理方式的缺点:

+   无法直接获取原始组件的状态, 需要通过 `ref` 获取
+   无法直接继承静态属性，需要额外实现或者使用第三方库才行
+   `ref` 被隔断, 如果需要保持 `ref` 的正确指向，需要配合 `forwardRef` 转发 `ref` 到原始组件上

2.  反向继承缺点:

+   代理组件与原始组件高耦合
+   函数组件无法使用
+   嵌套使用有风险, 内层组件的生命周期会覆盖外层组件的生命周期

3.  无法区分属性来源: 高阶组件的属性来源不清晰, 无法直观的看出属性是来自哪个高阶组件

```js
const Com = hoc3(hoc2(hoc1(App())))
<Com a="1" b="2" c="3" d="4" />
```

4.  无法区分依赖位置: 某些高阶组件可能会依赖于其他高阶组件, 但是无法直观的看出依赖关系

5.  产生无用的空组件:加深层级组件多层嵌套, 增加复杂度与理解成本

6.  重复命名的问题: 若父子组件有同样名称的 `props`, 或使用的多个 `HOC` 中存在相同名称的 `props` 则存在覆盖问题, 而且 `react` 并不会报错, 当然可以通过规范命名空间的方式避免

7.  实例化开销大:高阶组件需要实例化一个父组件来实现, 不管是在代码量还是性能上, 都不如 `hooks`


### 8.5 参考

+   [为什么 React Hooks 优于 HOCs](https://xie.infoq.cn/article/95bd927b294d1a49fb04c571c "https://xie.infoq.cn/article/95bd927b294d1a49fb04c571c")
+   [一文弄懂 React HOC](https://juejin.cn/post/7175890701433962555#heading-14 "https://juejin.cn/post/7175890701433962555#heading-14")

## 九、render props

### 9.1 简述

> `render prop` 是指在 `React` 组件中使用一个值为函数的属性(`props`)来渲染代码块的技术(类似vue slot插槽)

1.  组件允许通过属性传入一个函数, 该函数返回一个 `React 元素`
2.  组件内部通过调用该函数, 来渲染部分内容
3.  组件内调用函数时允许为函数传递任意参数, 可以是组件内部状态、方法、或其他任意数据

```js
const renderHeader = (data) => {
  return (<h1>Hello {data.target}</h1>)
}

<DataProvider render={renderHeader}/>
```

### 9.2 好处

1.  `render` 函数可以通过参数, 可以拿到组件内部状态、方法、任意数据; 在方法内也可调用当前组件的状态、方法、`props` 等任何数据;

> `render` 函数中, 既可以拿到父组件的数据、也可以拿到子组件的数据

```js
renderDom = data => {
  // data 是组件内部调用时的传参, 可以是任意数据(状态、组件内方法、组件内的 Props、或其他数据)
  // 这里也能拿到当前组件的, 状态、方法、props等任何数据
  return <h1>Hello {data.target}</h1>
}
<DataProvider renderDom={this.renderDom}/>
```

2.  可以进行组件的复用, 把组件无关的视图渲染逻辑抽象出来, 交给用户自己定义

### 9.3 注意点(缺点)

如果在 `render` 方法里直接创建函数, `render prop` 会使得 `PureComponent` 或 `shouldComponentUpdate` 无效, 因为每次 `render` 总会重新创建函数, 导致浅比较总是返回 `false`

```js
// 不推荐写法
render() {
  return <DataProvider renderDom={() => { }}/>
}
```

## 十、错误边界

### 10.1 简述

+   默认情况下, 若一个组件在渲染期间 `render` 发生错误, 会导致整个组件树全部被卸载(页面白屏), 这当然不是我们期望的结果
+   部分组件的错误不应该导致整个应用崩溃, 为了解决这个问题, `React 16` 引入了一个新的概念 —— 错误边界
+   错误边界是一种 `React` 组件, 这种组件可以捕获发生在其子组件树任何位置的异常, 我们可以针对这些异常进行打印、上报等处理, 同时渲染出一个降级(备用) `UI`, 而并不会渲染那些发生崩溃的子组件树
+   白话就是, 被错误边界包裹的组件, 内部如果发生异常会被错误边界捕获到, 那么这个组件就可以不被渲染, 而是渲染一个错误信息或者是一个友好提示！避免发生整个应用崩溃现象

### 10.2 实现代码

1.  `componentDidCatch()`: 捕获错误, 在这儿可以打印出错误信息、也可以对错误信息进行上报
2.  `static getDerivedStateFromError()`: 捕获错误, 返回一个对象, 更新 `state`

```js
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    // 发生错误则: 更新 state
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 捕获到错误: 可以打印或者上报错误
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>深感抱歉, 系统出现错误!! 开发小哥正在紧急维护中.... </h1>;
    }
    return this.props.children; 
  }
}

// 错误边界使用
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

### 10.3 注意事项(缺点)

1.  错误边界目前只在类组件中实现了, 没有在 `hooks` 中实现: 因为 `Error Boundaries` 的实现借助了 `this.setState` 可以传递 `callback` 的特性, `useState` 无法传入回调, 所以无法完全对标
2.  错误边界无法捕获以下四种场景中产生的错误: 仅处理渲染子组件期间的同步错误

+   自身的错误
+   异步的错误
+   事件中的错误
+   服务端渲染的错误

3.  补充: 错误边界只能在类组件中实现了, 并不是指 `Error Boundary` 对 `Hooks` 不生效, 而是指 `Error Boundary` 无法以 `Hooks` 方式指定, 但是对功能是没有影响! 你依然可以使用错误边界组件包裹使用了 `hooks` 的组件

## 十一、Redux

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3669ee205c374e64a79e4e4d511707fe~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=995&h=512&s=32152&e=png&b=fefefe)

+   页面上用户通过 `dispatch` 方法触发一个 `Action`: `dispatch(Action)`
+   `Store` 接收到 `Action`
+   `Store` 调用 `Reducer` 函数, 并将 `Action` 和当前状态作为参数传递给它
+   `Reducer` 函数根据 `Action` 类型执行相应的处理, 并返回新的状态
+   `Store` 更新状态, 并通知所有订阅状态的组件(视图)
+   组件(视图)收到通知, 获取新状态, 重新渲染

### 1.1 createStore 实现原理

1.  一个状态 `state` 用于存储状态
2.  一个监听器列表, 当状态改变时会遍历该列表, 执行里面的所有方法
3.  `subscribe`: 注册监听器
4.  `action`: 有效载体, 必须包含 `action.type`, 以及额外数据
5.  `dispatch`: 执行 `reducer(state, action)`、遍历执行所有监听器(触发组件状态更新、从而引起页面重新渲染)
6.  `reducer`: 纯函数 `(state, action)` ==> 根据 `action.type` 处理计算 ==> 返回新状态

### 1.2 redux
1. `dispatch`  派发action(一个有type与参数的对象)使用对应reducer函数生成新state,触发subscript
2. `getState`  获取store的state数据
3. `subscript` 订阅store变化
4. `createStore` 创建store

### 1.3 redux-thunk 中间件

理解: 中间件其实就是要对 `redux` 的 `store.dispatch` 方法做一些改造, 来定制一些功能

`Redux-thunk`: 实现原理

1.  本来 `dispatch` 参数只能是 `action` 对象, `redux-thunk` 中间件对 `dispatch` 进行了封装, 允许 `action` 是一个函数
2.  在 `dispatch` 中如果发现 `action` 是函数则执行 `action(dispatch, getState);`(延迟 `dispatch`), 否则执行 `dispatch(action)`.中间件,能够使dispatch的action为函数参数,当为函数参数时,会执行  action(dispatch,getState) , 主要就是为了异步调用或执行单次操作执行多次dispatch

```js
// 下面方式使用了 mapDispatchToProps
// 正常情况下, openModalAction 函数应该返回一个 action 对象
// redux-thunk 中间件对 dispatch 进行了封装, 所以允许 action 是一个函数
export const openModalAction = ({ code, data, ...rest }) => {
  return dispatch => {
    dispatch(openModal({ code, data, ...rest }));
  };
};
```
3. 可被RTK的 createAsyncThunk 替代

### 1.4 redux 优缺点

**优点:**

1.  单一数据源: 所有状态都存在一个对象中, 使得开发、调试都会变得比较容易
2.  `State` 是只读的: 如果要修改状态只能通过触发 `action` 来修改, `action` 是一个普通对象, 可以很方便被日志打印、序列化、储存…… 因此状态的修改过程就会变得有迹可寻, 比较方便得跟踪数据的变化
3.  `redux` 使用纯函数(`reducer`)来修改状态, 同一个 `action` 返回的 `state` 相同, 这样的话让状态的修改过程变得可控, 测试起来也方便

**缺点:** 冗余,要写很多模板代码,可用redux Toolkit代替

```js
export default createSlice({
  initialState,
  name: 'user',
  reducers: {
    updateUser: (state, { payload }) => ({ ...state, ...payload }),
  },
});
```

### 1.5 reduxToolkit(RTK)
1. `configStore` 创建store
2. `createSlice`  创建片段reducer,就是一个模块store
3. `createAsyncThunk` 就是创建一个extrReducer可以使用的异步函数

### 1.6 react-redux

1.  `Provider组件`: 用来包裹应用,注入store
2.  `connect`: 包裹组件的高阶组件,能将 store 中的状态和 dispatch 方法以props的方式传入组件
    + 使用方法: connect(mapStateToProps?,mapDispatchToProps?,megerToProps?,options?)(组件)
```js
import React from 'react';
import { connect } from 'react-redux';

const MyComponent = ({ count, increment, decrement }) => {
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>Increment</button>
            <button onClick={decrement}>Decrement</button>
        </div>
    );
};

// mapStateToProps: 将 Redux 状态映射到组件的 props
const mapStateToProps = (state) => ({
    count: state.count,  // 假设 Redux 状态有一个 `count` 属性
});

// mapDispatchToProps: 将 `dispatch` 映射到组件的 props
const mapDispatchToProps = (dispatch) => ({
    increment: () => dispatch({ type: 'INCREMENT' }),
    decrement: () => dispatch({ type: 'DECREMENT' }),
});

// 使用 connect 包裹组件
export default connect(mapStateToProps, mapDispatchToProps)(MyComponent);
```
3. `useSelector` 获取state的值
4. `useDispatch` 获取dispatch
5. `useStore` 获取store

**代码示例:**
```js
// store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export default configureStore({
    reducer: {
        counter: counterReducer,
    },
});
```
```js
// counterSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk('user/fetchUser', async () => {
    const response = await client.get('/fetchUser')
    return response.data
})

export const counterSlice = createSlice({
    name: 'counter',
    initialState: {
        value: 0,
        user: {},
        status: ''
    },
    reducers: {
        increment: (state) => {
            // Redux Toolkit 允许我们在 reducers 中编写 mutating 逻辑。
            // 它实际上并没有 mutate state 因为它使用了 Immer 库，
            // 它检测到草稿 state 的变化并产生一个全新的基于这些更改的不可变 state
            state.value += 1;
        },
        decrement: (state) => {
            state.value -= 1;
        },
        incrementByAmount: (state, action) => {
            state.value += action.payload;
        },
    },
    // 异步数据处理
    extraReducers(builder) {
        builder
            .addCase(fetchUser.pending, (state, action) => {
                state.status = 'loading'
            })
            .addCase(fetchUser.fulfilled, (state, action) => {
                state.status = 'succeeded'
                state.posts = action.payload
            })
            .addCase(fetchUser.rejected, (state, action) => {
                state.status = 'failed'
                state.error = action.error.message
            })
    }
});

// 为每个 case reducer 函数生成 Action creators
export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```
```js
// index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import store from './app/store';
import { Provider } from 'react-redux';


// 从 React 18 开始
const root = ReactDOM.createRoot(document.getElementById('root'));

root.render(
    <Provider store={store}>
        <App />
    </Provider>
);

```

```js
// App.js
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { increment, decrement, fetchUser } from './counterSlice';
const App = () => {
  const count = useSelector((state) => state.counter);
  const dispatch = useDispatch();
  const { userId, value } = useSelector((state) => state.user);
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
      <button onClick={() => dispatch(fetchUser())}>fetchUser</button>
    </div>
  );
};

export default App;

```


## 十二、组件之间传参方法

### 2.1 父子间通信

> 这种父子通信方式也就是典型的单向数据流, 父组件通过 `props` 传递数据, 子组件不能直接修改 `props`, 而是必须通过调用父组件函数的方式告知父组件修改数据

1.  父组件通过 `props` 传递数据给子组件

2.  子组件通过调用父组件传来的 `函数` 传递数据给父组件(自定义事件)

3.  非常规方法: 父组件通过 `ref` 获取子组件的实例对象


### 2.2 兄弟间通信

状态提升: 在父组件中创建共同的状态、事件函数, 其中一个兄弟组件调用父组件传递过来的事件函数修改父组件中的状态, 然后父组件将状态传递给另一个兄弟组件

### 2.3 任意组件之间进行通信

1.  使用 `Context`

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

function App({ children }) {
  const theme = useContext(ThemeContext);
  return (<div>{theme}</div>)
}

function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <App />
    </ThemeContext.Provider>
  )
}
```

2.  使用 `Redux` 等状态管理工具

## 十三、受控组件和非受控组件

### 3.1 受控组件

> 组件内部 `state` 或值完全受 `prop` 控制的组件

就像 `antd` 里 `Input` 组件, 可以通过 `props` 传一个 `value` 使得 `Input` 变为受控组件, `Input` 组件内部状态(值)就由 `props` 控制

```js
import { Input } from 'antd';
<Input value="写死或者设置为状态值"/>
```

### 3.2 非受控组件

> 组件内部 `state`或值不受 `props` 控制的组件, 由组件内部自己管理

就像 `antd` 里 `Input` 组件, 如果不给组件传 `value` 值, 那么组件就是非受控组件, `Input` 组件内由自己管理 `value`, 这时如果要想拿到表单的 `value` 则只能通过 `ref` 等手段, 手动获取

注意的是: `Input` 组件内部, 使用了 `input` 标签将 `value` 和状态进行绑定, 那么对于 `input` 标签来说它是受控的, 所以受控组件只是相对

```js
import { Input } from 'antd';
<Input/>
```

### 3.3 什么时候使用受控组件、什么时候使用非受控

> 当组件内部值或状态和外部存在交互逻辑时, 则需要将其作为受控组件进行使用

1.  当组件状态(值)只由自身交换控制, 不受外部影响时, 可使用非受控组件: 比如 `Antd` `Input` 组件, 如果输入框的内容只随着用户输入时改变, 那么就可以使用非受控组件

2.  当组件状态(值)除了受自身交换控制、还受到外部影响时, 可使用受控组件: 比如 `Antd` `Input` 组件, 需要和其他控件产生联动对组件的值进行相应的格式化

3.  当组件状态(值)和外部需要交换时, 可使用受控组件: 比如 `Antd` 单选框, 当选中时需要隐藏页面上内容时, 一般就会将单选框最为受控组件进行使用


### 3.4 参考

+   [组件设计 —— 重新认识受控与非受控组件](https://zhuanlan.zhihu.com/p/93335058 "https://zhuanlan.zhihu.com/p/93335058")
+   [受控组件与非受控组件](https://juejin.cn/post/6844903896389779469 "https://juejin.cn/post/6844903896389779469")

## 十四、Ref 相关

### 4.1 作用

1.  在函数组件中, 当我们希望组件能够 `记住` 或者说 `存储` 某些信息, 但呢又不希望该信息触发新的渲染时, 就可以使用 ref 来存储
2.  用于访问真实 `DOM` 元素
3.  当父组件需要获取子组件实例对象时, 也可通过 `ref` 来实现

### 4.2 获取真实 `DOM`: 三种创建方式

1.  推荐使用 `API`: `React.createRef()`、`useRef`

```js
// 类组件, 使用 createRef
this.ref = React.createRef();
<div ref={this.ref}></div>


// 函数组件, 使用 useRef
const ref = React.useRef();
<div ref={ref}></div>
```

2.  `ref` 回调函数方式

```js
// 类组件
bindRef = ele => {
  this.bodyRef = ele;
};

<div ref={this.bindRef}></div>


// 函数组件
const bindRef = useCallback((ele) => {
  
}, []);
<div ref={bindRef}></div>
```

3.  字符串(仅限类组件中使用)

```js
// 会自动在 this 上绑定 bodyRef, 等于当前元素
<div ref="bodyRef"></div>
```

### 4.3 获取子组件实例

1.  子组件为类组件, 直接绑定 `ref`, 就能够拿到整个子组件的实例对象

```js
class A extends Component {}

const App = () => {
  const ref = useRef()
  return (<A ref={ref}/>)
}
```

2.  函数组件: `forwardRef` + `useImperativeHandle`

```js
import React, { useState, forwardRef, useImperativeHandle, useRef } from 'react';

/*
* props：组件的所有 props。
* ref：传递给组件的 ref。
* */
const MyComponent = forwardRef((props, ref) => {
    
    const [count, setCount] = useState(0);

    // 使用 useImperativeHandle 自定义暴露给父组件的 ref 方法
    useImperativeHandle(ref, () => ({
        increment: () => setCount(count + 1),
        reset: () => setCount(0),
    }));

    return <div>{count}</div>;
});

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

### 4.4 转发 ref

1.  可使用高阶组件 `React.forwardRef` 用于转发 ref 到子组件中的某个 DOM 元素或类组件实例上

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

2.  使用传入props将 `ref` 进行转发(常见于类组件, 毕竟 `forwardRef` 不能用于类组件)

```js
class A extends Component {
  render () {
    return (
      <div ref={this.props.innerRef}>
        1
      </div>
    );
  }
}

const bodyRef = useRef()
<A  innerRef={bodyRef} />
```

## 十五、Fragment

在 `React` 中如果需要渲染多个元素, 需要使用元素进行包裹, 否则将会报错

+   报错原因, 主要原因还是在 `JSX` 编译这块

```js
// 编译前
const dom = (
  <ChildA />
  <ChildB />
  <ChildC />
);

// 编译后, 这样很明显是有问题的
const dom = (
  React.createElement(……) 
  React.createElement(……) 
  React.createElement(……)
);
```

+   上面错误代码解决办法就是, 使用 `div` 等标签进行包裹, 这样就能够通过编译

```js
// 编译前
const dom = (
  <div>
    <ChildA />
    <ChildB />
    <ChildC />
  </div>
);

function MyComponent() {
  return React.createElement(
    'div', 
    null,
    React.createElement(……),
    React.createElement(……),
    React.createElement(……),
  );
}
```

+   上面处理会有个问题, 就是会添加了额外节点, `Fragment` 出现就为了解决上面的问题, 通过 `Fragment` 可以将子列表分组, 最终在渲染为真实 `DOM` 节点时会将其忽略(不会进行渲染)

```js
// 编译前
const dom = (
  <React.Fragment>
    <ChildA />
    <ChildB />
    <ChildC />
  </React.Fragment>
);

// 编译后
function MyComponent() {
  return React.createElement(
    React.Fragment, 
    null,
    React.createElement(……),
    React.createElement(……),
    React.createElement(……),
  );
}
```

+   `Fragment` 简写形式 `<></>`

```js
const dom = (
  <>
    <ChildA />
    <ChildB />
    <ChildC />
  </>
);
```

+   `Fragment` 对应 `ReactElement` 元素类型(`type`) 为 `Symbol('react.fragment')`

```js
console.log(
  <>
    <div> 1</div>
    <div> 2</div>
  </>,
);

{
  type: Symbol('react.fragment'),
  ...
}
```

**补充:** 有一点是需要注意的, `<>` 并不接受任何属性, 包括 `key`、`ref` 属性

## 十六、React 元素中 $$typeof 的作用

> 用于标识 `React` 元素, 该属性值为 `Symbol`, 主要为了防止 `XSS` 攻击

> 补充: `XSS` 攻击通常指的是通过利用网页开发时留下的漏洞, 通过巧妙的方法注入恶意指令代码到网页, 使用户加载并执行攻击者恶意制造的网页程序。

1.  已知 `JSX` 语法将被编译为 `React.createElement` 后返回一个对象(`React` 元素)

```js
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol('react.element'), // 标识 React 元素
}
```

2.  由于服务器可以存储任意的 `JSON` 数据, 如果在没有 `$$typeof` 情况下, 就很容易被伪造(手动创建 `React` 元素, 在页面进行注入)

```js
// 假设后端返回了这样一串数据(React 元素)
const message = {  
  type: 'div',  
  props: {    
    dangerouslySetInnerHTML: {    
      __html: '/* 把你想的搁着 */'    
    },  
  },
};

// 前端这么现实数据
<p>{message}</p>
```

3.  `由于服务器存储的 JSON 不支持 Symbol 类型数据`, 所以只要在 React 元素中添加 Symbol 类型数据 $$typeof, React 在处理元素时只需`判断 $$typeof 是否非法`就能够识别出 `非法元素`(伪造元素)

4.  如果浏览器不支持 `Symbols` 怎么办？


+   那这种保护方案就无效了 但是 React 仍然会加上 $$typeof 字段以保证一致性
+   但这样只会设置一个数字 —— `0xeac7`
+   而之所以设置 `0xeac7`, 只是因为 `0xeac7` 看起来有点像 `React`

5.  参考: [为什么React元素有一个$$typeof属性?](https://overreacted.io/zh-hans/why-do-react-elements-have-typeof-property/ "https://overreacted.io/zh-hans/why-do-react-elements-have-typeof-property/")

## 十七、Hooks

> `React 16.8` 的新增特性, 它可以让你在不编写 `class` 的情况下使用 `state` 以及其他的 `React` 特性

### 17.1 和类组件对比有什么优点

**优点:**

1.  更简洁: 使用 `Hooks` 你可以在函数组件中使用状态和其他 `React` 特性, 无需编写 `class` 从而避免了繁琐的 `class` 组件的声明和继承、同时也无需考虑 `this` 指向等问题

2.  易复用: 自定义 `Hooks` 允许将组件之间的状态逻辑进行抽离, 作为一个独立的可组合和可共享单元, 从而减少了重复代码的出现

3.  易测试： 通过 `Hooks` 可以将组件渲染、和业务逻辑分离进行分离, 使得组件的测试变得更加容易。可以针对每个 `Hook` 编写单独的测试，确保其正确性, 同时保持组件测试的简洁性。

4.  易维护,易拆分粒度小: 使用 `Hooks` 可以更容易地拆分组件, 将组件的不同部分拆分成更小的逻辑单元，有助于更好地组织和管理代码。

5.  可扩展: `Hooks` 的设计允许你在组件内部使用多个不同的 `Hook`, 这使得你可以在一个函数组件中使用各种各样的特性, 而不必担心组件层次的嵌套和复杂性

6.  性能好: 类组件只能使用高阶组件进行逻辑复用, 而 `Hooks` 可以在不增加组件嵌套的情况下复用状态逻辑, 从而减少了组件的渲染次数, 提高了性能


**缺点**:

1.  学习成本: 对于那些熟悉传统 `class` 组件的开发者来说, 学习 `Hooks` 可能需要一些时间。`Hooks` 改变了组件的编写方式, 并且需要理解如何正确地使用 `useState`、`useEffect`、`useContext` 等钩子函数

2.  使用规则: `Hooks` 有一些使用规则, 例如在条件语句中不可使用, 或者只能在函数组件的最顶层使用。违反这些规则可能导致 `bug` 和意想不到的行为。

3.  错误使用产生性能问题: 尽管 `Hooks` 通常可以优化组件逻辑, 但不正确地使用它们可能导致性能问题。比如, 在 `useEffect` 中没有正确处理依赖项数组可能会导致不必要的重复执行。


### 7.2 常用的 Hooks

+   `useState`: 用于定义组件状态, 需要注意的是该方法在更新状态时会进行浅比较, 如果待更新状态值和当前状态值一致, 则不会进行更新, 不会引起组件的重新渲染

```js
const [state, setState] = useState(0);
setState(0); // 不会引起组件重新渲染
```

+   `useRef`: 获取 `DOM` 元素对象、记录非状态数据、获取子组件实例对象
+   `useImperativeHandle` 用于控制暴露给父组件的属性
+   `useEffect`: 让函数型组件拥有处理 `副作⽤` 的能⼒, 每次依赖项改变, 都会触发回调函数的执行, 通过它可模拟类似 `类组件` 中的部分⽣命周期, 如 `componentDidMount`、`componentDidUpdate`、`componentWillUnmount`
+   `useLayoutEffect`: 与 `useEffect` 相同, 但它会在所有的 `DOM` 变更之后同步调用
+   `useInsertionEffect`: 在任何 `DOM` 突变之前触发, 主要是解决 `CSS-in-JS` 在渲染中注入样式的性能问题

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


### 7.3 useEffect、useLayoutEffect、useInsertionEffect 之间的区别

1.  `useInsertionEffect`: 应该是 `DOM` 变更之前执行

2.  `useLayoutEffect`: `DOM` 已经按照 `VDOM` 更新了, 此时 `DOM` 已经在内存中更新了, 但是还没有更新到屏幕上

3.  `useEffect`: 则是浏览器完成渲染之后执行

4.  所以三者执行顺序: `useInsertionEffect(DOM 变更前)`、`useLayoutEffect(DOM 变更后)`、`useEffect`

5.  `useLayoutEffect` 与 `useEffect` 基本相同, 但它会在所有的 `DOM` 变更之后 `同步` 调用, 一般可以使用它来读取 `DOM` 布局并同步触发重渲染, 为了避免阻塞视觉更新, 我们需要尽可能使用标准的 `useEffect`

6.  `useEffect` 和 `useLayoutEffect` 都可用于模拟 `componentDidUpdate` `componentDidMount`, 首次渲染先执行useEffect监听函数内容不执行return内容,rerender的时候先执行return内容,再执行监听函数内容.

7.  当父子组件都用到 `useEffect` 时, 子组件中的会比父组件中的先触发

8.  参考: [面试官:useLayoutEffect和useEffect的区别](https://cloud.tencent.com/developer/article/1918923 "https://cloud.tencent.com/developer/article/1918923")


### 7.4 React.memo

1.  在类组件的时代时代, 为了性能优化我们经常会选择使用 `PureComponent`, 组件每次默认会对 `props` 进行一次 `浅比较`, 只有当 props 发生变更, 才会触发 render

```js
class MyComponent extends PureComponent {
  render () {}
}
```

2.  当然在类组件中, 我们除了使用 `PureComponent` 还可以在 `shouldComponentUpdate` 生命周期中, 对 `props` 进行比较, 进行更深层次的控制;

> 补充:
>
> +   `shouldComponentUpdate` 当收到新的 `props` 或 `state` 时, 在渲染之前都会被调用
> +   这里的比较可以是浅比较、也可以是深比较, 主要看代码实现
> +   当 `shouldComponentUpdate` 返回为 `true` 的时候, 当前组件进行 `render`, 如果返回的是 `false` 则不进行 `render`

```js
class MyComponent extends Component {
  shouldComponentUpdate(){
    if (需要 Render) {
      // 会进行渲染
      return true
    }

    // 不会进行渲染
    return false
  }
  render () {}
}
```

+   在函数组件中, 我们是无法使用上面两种方式来限制 `render` 的, 但是 `React` 贴心的提供了 `React.memo` 这个 `HOC(高阶组件)`, 它的作用和 `PureComponent` 很相似, 只是它是专门为函数组件设计的

> React.memo 使用说明
>
> +   默认情况下会对组件 `props` 进行 `浅比较`, 只有 `props` 变更才会触发 `render`
> +   允许传入第二参数, 该参数是个函数, 该函数接收 `2` 个参数, 两个参数分别是新旧 `props`,
> +   注意: 与 `shouldComponentUpdate` 不同的是, `arePropsEqual` 返回 `true` 时, 不会触发 `render`, 如果返回 `false` 则会, 和 `shouldComponentUpdate` 刚好与其相反

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

> 作用: 性能优化, 如果本组件中的数据没有发⽣变化, 阻⽌组件更新, 类似类组件中的 `PureComponent` 和 `shouldComponentUpdate`

### 7.5 使用时需要注意什么

1.  遵守 `Hooks` 使用规则: `Hooks` 只能在函数组件的顶层使用, 或者在自定义 `hooks` 中使用, 不能在循环、条件或嵌套函数中使用 `hooks`

2.  尽量避免通过 `useEffect` 来处理 `actions`: `useEffect` 监听某个状态 `A`, 内部又去修改 `A`, 这样就容易造成死循环

3. 依赖数组: 在使用 `useEffect` 或 `useCallback` 等 `hooks` 时, 务必提供依赖数组作为第二个参数。忽略或者错误的依赖数组可能导致意外行为, 比如过度重新渲染或内存泄漏

4. 避免无限循环: 在使用 `useEffect` 时要小心无限循环, 确保依赖数组中有正确的依赖项, 并且 `effect` 的逻辑不会触发不必要的重新渲染

5. 状态不可变性: 避免直接修改状态对象, 也不要试图通过 `push`、`pop`、`splice` 等直接更改数组

6. 如果某个数据的变更不需要触发 `render`, 或者该数据没有在 `jsx` 中被使用, 那么就不要使用 `useState` 改用 `useRef` 进行记录

7. 单一职责 `组件`、`useEffects`

### 7.6 为什么 hooks 不能写在循环或者条件判断语句里?

> `Hooks` 只能在函数组件的顶层使用, 或者在自定义 `hooks` 中使用, 不能在循环、条件或嵌套函数中使用 `hooks`

```js
export default () => {
  const [name, setName] = useState('1');

  if (!name) {
    return null;
  }

  const [age, setAge] = useState();

  const handler = useCallback(() => {
    setName(null);
  }, []);

  return (
    <div onClick={handler}>
      点击我
    </div>
  );
};
```

**原因:** `React` 需要利用 `调用顺序` 来正确更新相应的状态, 以及调用相应的钩子函数, 一旦在循环或条件分支语句中调用 `Hooks`, 就容易导致调用顺序的不一致性, 从而产生难以预料到的后果

这里拿 `useState` 来举例:

1.  `hooks` 为了在函数组件中引入状态, 维护了一个有序表
2.  首次执行时会将每个 `useState` 的初始值, `依次` 存到有序表里
3.  每次更新也都会按照 `索引` 修改指定位置的值
4.  每次 `render` 会将对应 `索引` 的值作为状态返回
5.  那么试想下, 如果我们将 `useState` 写在判断条件下, 可能会导致 `useState` 不执行, 那么这个有序列表就会出现混乱

```js
export default () => {
  const [name, setName] = useState('1');

  if (!name) {
    return null;
  }

  const [age, setAge] = useState();

  const handler = useCallback(() => {
    setName(null);
  }, []);

  return (
    <div onClick={handler}>
      点击我会报错
    </div>
  );
};
```

> 总结: `hooks` 是将 `state` 原子化, 使用类似索引的方式来记录状态值, 当连续创建状态 `A` `B`, 就会有索引 `0` 对应着 `A`, 索引 `1` 对应这 `B`, 如果使用在循环、条件、嵌套函数内使用 `Hook` 就很容易造成索引错乱

### 7.7 如何打破了 React Hook 必须按顺序、不能在条件语句中调用的枷锁?

总结一下: 之前是通过顺序来查找, 现在通过唯一 `key` 来查找

实现则需要去修改源码, 参考: [我打破了 React Hook 必须按顺序、不能在条件语句中调用的枷锁](https://juejin.cn/post/6939766434159394830 "https://juejin.cn/post/6939766434159394830")

### 7.8 为什么 useState 返回的是一个数组?

1.  `useState` 要返回两个值, 一个是当前状态, 另一个则是修改状态的方法, 那么这里它就有两种方式可以返回这两个值: 数组、对象

2.  那么问题就回到, 数组和对象解构赋值的区别了:


+   数组的元素是按次序排列的, 数组解构时变量的取值由数组元素的位置决定, 变量名可以任意命名, 如下:

```js
const [name, setName] = useState()
const [age, setAge] = useState()
```

+   对象的属性没有次序, 解构时变量名必须与属性同名才能取到正确的值, 假设 `useState` 返回的是一个对象, 那么就得这么使用:

```js
const { state: name, setState: setName } = useState()
const { state: age, setState: setAge} = useState()
```

+   上面例子可以得出结果, `useState` 返回数组相比于对象会更灵活、解构起来也会更简洁、方便

3.  当然最终 `useState` 返回的是啥, 还是由具体实现决定, 如果 `useState` 返回的是对象, 也不是不行

### 7.9 简单实现 hooks

```js
// 一、实现useState
const { render } = require("react-dom");
let memoriedStates = [];
let lastIndex = 0;
function useState(initialState) {
  memoriedStates[lastIndex] = memoriedStates[lastIndex] || initialState;
  function setState(newState) {
    lastIndex+=1;
    memoriedStates[lastIndex] = newState;
    // 状态更新完毕，调用render函数。重新更新视图
    render();
  }
  // 返回最新状态和更新函数
  return [memoriedStates[lastIndex], setState];
}

// 二、实现useEffect
let lastDendencies; // 存放依赖项的数组
function useEffect(callback, dependencies) {
  if (lastDendencies) {
    // 判断传入的依赖项是不是都没有变化，只要有以一项改变，就需要执行callback
    const isChange = dependencies && dependencies.some((dep, index) => dep !== lastDendencies[index]);
    if (isChange) {
      // 一开始没有值，需要更新一次(相当于componentDidMount)
      typeof callback === 'function' && callback();
      // 更新依赖项
      lastDendencies = dependencies;
    }
  } else {
    // 一开始没有值，需要更新一次(相当于componentDidMount)
    typeof callback === 'function' && callback();
    // 更新依赖项
    lastDendencies = dependencies;
  }
}

// 三、实现useCallback
let lastCallback; // 最新的回调函数
let lastCallbackDependencies = []; // 回调函数的依赖项
function useCallback(callback, dependencies = []) {
  if (lastCallback) {
    const isChange = dependencies && dependencies.some((dep, index) => dep !== lastCallbackDependencies[index]);
    if (isChange) {
      // 只要有一个依赖项改变了，就更新回调(重新创建)
      lastCallback = callback;
      lastCallbackDependencies = dependencies;
    }
  } else {
    lastCallback = callback;
    lastCallbackDependencies = dependencies;
  }
  // 最后需要返回最新的函数
  return lastCallback;
}

// 四、实现useRef
let lastRef;
function useRef(initialValue = null){
  
  lastRef = lastRef != undefined ? lastRef : initialValue;
  // 本质上就是返回一个对象，对象种有一个current属性，值为初始化传入的值，如果没有传入初始值，则默认为null
  return {
    current: lastRef
  }
}

// 五、实现useContext
function useContext(context){
  // 很简单，就是返回context的_currentValue值
  return context._currentValue;
}

// 六、实现useReducer
let lastState;
function useReducer(reducer, initialState){
  lastState = lastState !== undefined ? lastState : initialState;
  // dispatch一个action，内部就是自动调用reducer来计算新的值返回
  function dispatch(action){
    lastState = reducer(lastState, action);
    // 更新完毕后，需要重新渲染视图
    render();
  }
  // 最后返回一个的状态值和派发action的方法
  return [lastState, dispatch];
}
```

### 7.10 useCallback 和 useMemo 的区别?

1.  可以 `useMemo` 来实现 `useCallback` 吗?

可以, `useMemo` 只要返回一个函数即可

> 拓展知识: `useCallback` 是「`useMemo` 的返回值为函数」时的特殊情况, 是 `React` 提供的便捷方式。在 `React Server Hooks` 代码 中, `useCallback` 就是基于 `useMemo` 实现的, 尽管 `React Client Hooks` 没有使用同一份代码, 但 `useCallback` 的代码逻辑和 `useMemo` 的代码逻辑仍是一样的

## 十八、性能优化

1.  跳过不必要的组件更新

+   `PureComponent`、`React.memo`、`shouldComponentUpdate`
+   `useMemo`、`useCallback` 来生成缓存内容,避免不必要的重新计算/更新
+   状态下放, 缩小状态影响范围: 如果一个状态只在某部分子树中使用, 那么可以将这部分子树提取为组件, 并将该状态移动到该组件内部
+   列表项使用 `key` 属性

2.  按需加载:

+   懒加载: 通过 `Webpack` 的动态导入和 `React.lazy` 方法来实现
+   虚拟列表: 懒渲染指当组件进入或即将进入可视区域时才渲染数据
+   时间分片: 通过 `requestIdleCallback` 来实现, 该方法会在浏览器空闲时执行回调, 从而避免阻塞主线程

3.  批量更新:

+   类组件, `setState` 自带批量更新操作
+   函数组件, 尽量将相关的状态进行合并, 然后进行批量更新

4.  按优先级更新, 及时响应用户: 举个例子当页面弹出一个 `Modal`, 当用户点击 `确定` 按钮后, 代码将执行两个操作, 1、关闭 Modal; 2、 处理 `Modal` 传回的数据并展示给用户; 同时假设第二个操作需要执行 `500ms` 时, 那么用户会明显感觉到从点击按钮到 `Modal` 被关闭之间的延迟, 如下代码如果 `setNumbers` 这一步处理时间耗时, 那么就会出现明显的卡顿

```js
const slowHandle = () => {
  setShowInput(false)
  // 计算耗时 500s
  setNumbers([...numbers, +inputValue].sort((a, b) => a - b))
}
```

解决办法:
+ 通过`useDeferredValue`或`useTransition`延迟更新
+ 通过 `setTimeout` 将耗时任务放到下一个宏任务中去执行
    
```js
    const fastHandle = () => {
      // 优先响应用户行为
      setShowInput(false)
      // 将耗时任务移动到下一个宏任务执行
      setTimeout(() => {
        setNumbers([...numbers, +inputValue].sort((a, b) => a - b))
      })
    }
```

7.  通过 `debounce`、`throttle` 优化频繁触发的回调函数

8. 在组件注册的全局事件、定时器等, 在卸载前要清理掉, 防止内存泄漏

9. 不要使用内联函数定义, 会导致每次 `render` 时都会创建新的函数, 从而导致不必要的 `render`

10. 避免使用内联样式属性, 会导致每次 `render` 时都会创建新的样式对象, 从而导致不必要的 `render`

11. 为组件创建错误边界, 防止错误影响整个应用

### 8.1 参考

+   [React 性能优化 | 包括原理、技巧、Demo、工具使用](https://juejin.cn/post/6935584878071119885#heading-30 "https://juejin.cn/post/6935584878071119885#heading-30")
+   [React 性能优化最佳实践（十九）](https://juejin.cn/post/7064804207722758157#heading-18 "https://juejin.cn/post/7064804207722758157#heading-18")

## 十九、React 18 更新内容有哪些?

> 主基调: 并发性是 React 18 的主要优势之一

### 9.1 彻底放弃 IE

1.  `17` 修复 `IE` 兼容问题
2.  `18` 彻底放弃 `IE` 的支持

### 9.2 自动批处理

1.  我们都知道在 `React 18` 之前:

+   在合成事件、生命周期中如果多次修改 `state`, 会进行批处理, 然后只会触发一次 `render`
+   在定时器、`promise.then`、原生事件处理函数中不会进行批处理
+   这里之所以会有两种不同情况, 主要原因是早期对于 `批处理` 是通过一个状态作为批处理依据, 具体可查阅上文 `7.2 React 的更新机制: 异步 OR 同步` 部分内

```js
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount((c) => c + 1); 
    setFlag((f) => !f); 
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={flag ? { color: "blue" } : { color: "black" }}>{count}</h1>
    </div>
  );
}
```

2.  在 `React 18` 之后所有的更新都将自动批处理:

+   主要原因是不再通过 `状态` 来作为批处理依据, 改为基于 `fiber` 的调度器(Scheduler)以任务优先级为依据来进行批处理
+   通过 `Scheduler` 来进行任务调度, 在事件中是同步状态更新(合成事件、生命周期)则立即进行状态合并,是异步状态更新(定时器、`promise.then`、原生事件处理函数)则等待事件执行完成再进行状态更新合并,最后合并渲染只进行一次DOM更新,从而实现了自动批处理
+   参考: [React18精读一: Automatic Batching 自动批处理](https://zhuanlan.zhihu.com/p/523683561 "https://zhuanlan.zhihu.com/p/523683561")

3.  如何退出批处理: `flushSync` 强制同步更新

```js
import React, { useState } from 'react';
import { flushSync } from 'react-dom';

const App: React.FC = () => {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);

  return (
    <div
      onClick={() => {
        // 第一次更新
        flushSync(() => {
          setCount1(count => count + 1);
        });
        // 第二次更新
        flushSync(() => {
          setCount2(count => count + 1);
        });
      }}
    >
      <div>count1: {count1}</div>
      <div>count2: {count2}</div>
    </div>
  );
};

export default App;
```

### 9.3 Render API

> 修改了将组件挂载到 `root` 节点的一个 `api`

1.  旧版本

```js
import React from 'react';
import ReactDOM from 'react-dom';
const root = document.getElementById('root');
ReactDOM.render(<App />, root);
```

2.  `18` 版本: 支持并发模式渲染

```js
// React 18
import React from 'react';
import ReactDOM from 'react-dom/client';

const root = document.getElementById('root');

ReactDOM.createRoot(root).render(<App />);
```

### 9.4 删除: 卸载组件时的更新状态的警告

> 参考: [react-18/discussions/82](https://github.com/reactwg/react-18/discussions/82 "https://github.com/reactwg/react-18/discussions/82")

1.  背景: `18` 之前我们如果在组件卸载后, 尝试修改状态就会在控制台抛出异常

```js
useEffect(() => {
  function handleChange() {
    setState(store.getState())
  }
  store.subscribe(handleChange)

  return () => store.unsubscribe(handleChange)
  // 这里如果没有解除订阅, 那么控制台将会抛出如下错误
  // Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.
}, [])
```

2.  目的: 如上代码如果忘记调用 `unsubscribe` 解除订阅, 就会出现内存泄露!! 所以为了避免出现这种问题, 所以就设置了上面的警告信息!!

3.  为什么要去除: 主要原因还是警告具有误导性, 如下代码是一个比较常见的场景, 在执行 `post('/someapi')` 期间如果组件卸载, 后面调用 `setPending`, 在 `18` 之前这里将会抛出错误, 但实际上这里并没有太大的毛病, 也并没有存在内存泄露问题!!!


```js
async function handleSubmit() {
  setPending(true)
  await post('/someapi') // component might unmount while we're waiting
  setPending(false)
}
```

4.  在 `18` 之前人们为了抑制这个错误, 经常写如下代码:

```js
let isMountedRef = useRef(false)
useEffect(() => {
  isMountedRef.current = true
  return () => {
    isMountedRef.current = false
  }
}, [])

async function handleSubmit() {
  setPending(true)
  await post('/someapi')
  if (!isMountedRef.current) {
    setPending(false)
  }
}

```

5.  看起来上面解决方法实际比原来的问题更加糟糕, 所以最后还是删了吧.... 后面看看有没其他手段来规避内存泄露

### 9.5 关于 React 组件的返回值

> 参考: [react-18/discussions/75](https://github.com/reactwg/react-18/discussions/75 "https://github.com/reactwg/react-18/discussions/75")

1.  在 `React 17` 中, 如果需要返回一个空组件, 只允许返回 `null`, 如果返回了 `undefined` 控制台则会在运行时抛出一个错误

2.  在 `React 18` 中, 既能返回 `null`, 也能返回 `undefined` (但是 `React 18` 的 `dts` 文件还是会检查, 只允许返回 `null`, 这里我们可以忽略这个类型错误

3.  之前为什么要这么设计: 在编码过程中忘记 `return`, 是比较容易犯的一个错误, 为了帮助用户发现这个问题, 所以就有了这个警告

4.  那现在为什么又允许了:


+   在 `Suspense` 中允许为 `fallback` 为 `undefined`, 所以为了保持一致性, 顾允许返回 `undefined`
+   考虑到现在类型系统和 `Eslint` 都已经非常成熟、健壮, 通过它们就可以很好避免这类低级错误了

### 9.6 严格模式下第二次渲染期间抑制日志

1.  背景:

+   为了防止组件内有什么意外的副作用, 而引起 `BUG`, 所以严格模式下 `React` 在开发模式中会刻意执行两次渲染, 尽可能把问题提前暴露出来, 来提前预防
+   而 `React` 为了让日志更容易阅读, 通过修改 `console` 中的方法, 取消了其中一次渲染的控制台日志

2.  问题: 开发人员在调试过程中会存在很多困惑

3.  展望未来: `React` 将不再默认在第二次渲染期间抑制日志, 如果安装了 `React DevTools > 4.18.0`, 第二次渲染期间的日志现在将以柔和的颜色显示在控制台中


### 9.7 Suspense

> 参考: [react-18/discussions/72](https://github.com/reactwg/react-18/discussions/72 "https://github.com/reactwg/react-18/discussions/72")

1.  更新前: 如果 `Suspense` 组件没有提供 `fallback` 属性, `React` 就会跳过它, 继续讲错误向上传递, 直到被最近的 `Suspense` 捕获到

```js
<Suspense fallback={<Loading />}> // 这个边界被使用，显示 Loading 组件
  <Suspense>  // 这个边界被跳过，没有 fallback 属性
    <Page />
  </Suspense>
</Suspense>
```

2.  更新后: 如果 `Suspense` 组件没有提供 `fallback` 属性, 错误不会往外层传递, 而是展示为空

```js
<Suspense fallback={<Loading />}> //  不使用
  <Suspense> //  这个边界被使用, 将 fallback 渲染为 null
    <Page />
  </Suspense>
</Suspense>
```

3.  为什么要做调整: `Suspense` 的错误如果一直往外透传, 那么这样会导致混乱、难以调试的情况发生

### 9.8 Concurrent Mode(并发模式)

> 并不是一个功能, 而是一个底层设计

1.  使用新版本的 `createRoot(root).render` 来挂载节点将启用 `并发模式`, 但是并没开启 `并发更新`, 要相启用相应的 `并发更新` 需要使用相应的 `api`

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10519f26155944128decd0d49d2c76f4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=581&h=599&s=17383&e=png&b=fefefe)

2.  开启并发更新: `useTransition`, 如下代码 `useTransition` 返回两个数组参数

+   `isPending` 表示是否正在等待中
+   `useTransition` 接收一个回调函数, 函数中的状态修改将被标记为 `非紧急渲染` 任务, 这样的话在大量的任务下也能保持 `UI` 能够快速的响应, 从而来显著改善用户交互

```js
import React, { useState, useEffect, useTransition } from 'react';

const App = () => {
  const [list, setList] = useState([]);
  const [isPending, startTransition] = useTransition();

  useEffect(() => {
    // 使用了并发特性，开启并发更新
    startTransition(() => {
      setList(new Array(10000).fill(null));
    });
  }, []);
  
  return (
    <>
      {list.map((_, i) => (
        <div key={i}>{i}</div>
      ))}
    </>
  );
};
```

3.  开启并发更新: `useDeferredValue`, 返回一个延迟响应的值, 可以让一个 `state` 延迟生效, 只有当前没有紧急更新时, 该值才会变为最新值! 同 `useTransition` 一样都是标记了一次非紧急更新

```js
import React, { useState, useEffect, useDeferredValue } from 'react';

const App = () => {
  const [list, setList] = useState([]);

  // 使用了并发特性，开启并发更新
  const deferredList = useDeferredValue(list);

  useEffect(() => {
    setList(new Array(10000).fill(null));
  }, []);

  return (
    <>
      {deferredList.map((_, i) => (
        <div key={i}>{i}</div>
      ))}
    </>
  );
};
```

4.  `useDeferredValue` 与 `useTransition` 的区别

+   相同: 从功能、作用以及内部的实现上来讲, 他们是一样的都是标记成了延迟更新任务
+   不同: `useTransition` 是一段逻辑延迟更新, 而 `useDeferredValue` 是把一个值延迟更新.

5.  简单总结: 所有的东西都是基于 `fiber` 架构实现的, `fiber` 为状态更新提供了可中断的能力

+   并发更新的意义就是 `交替执行` 不同的任务(任务可以划分优先级, 高优先级的先执行), 当预留的时间不够用时, `React` 将线程控制权交还给浏览器, 等待下一帧时间到来, 然后继续被中断的工作
+   并发模式是实现并发更新的基本前提, 同时时间切片是实现并发更新的具体手段

### 9.9 几个新的 API

1.  `useId`:

+   参考: [为了生成唯一id, React18专门引入了新Hook: useId](https://zhuanlan.zhihu.com/p/437913203 "https://zhuanlan.zhihu.com/p/437913203")
+   主要适用于 `SSR`(服务端渲染), 让服务端渲染的组件生成 `id` 与客户端渲染的组件的 `id` 一致

2.  `useSyncExternalStore`:
    +   参考: [React 18 撕裂介绍](https://juejin.cn/post/6999778495077302302 "https://juejin.cn/post/6999778495077302302")
    +   参考: [React 的并发悖论](https://zhuanlan.zhihu.com/p/623324430 "https://zhuanlan.zhihu.com/p/623324430")
    +   视图撕裂: 对于开启并发更新的 `React`, 更新流程可能中断, 因任务更新时间不一致,渲染内容冲突的现象
    +   `React` 的 `API` 已经原生的解决的并发特性下的撕裂(`tear`)问题, 但是对于 `redux` 等外部框架它在控制状态时可能并非直接使用的 `React`的 `API`(`useState`), 而是自己在外部维护了一个 `store` 对象, 它脱离了 `React` 的管理, 也就无法依靠 `React` 自动解决撕裂问题。因此, `React` 对外提供了这样一个 `API`, 帮助这类框架开发者(有外部 `store` 需求的)解决撕裂问题
    +   对于如何解决外部框架的并发特性下的撕裂(`tear`)问题, `React` 目前并没有好的一个方案, 目前 `useSyncExternalStore` 的作用其实是状态管理库触发的更新都以同步的方式执行, 这样就不会有同步时机的问题了
3.  `useInsertionEffect`: 这个 `Hooks` 只建议 `css-in-js` 库来使用, 这个 `Hooks` 执行时机在 `DOM` 生成之后, `useLayoutEffect` 之前, `它的工作原理大致和 useLayoutEffect` 相同, 只是此时无法访问 `DOM` 节点的引用, 一般用于提前注入 `<style>` 脚本

## 二十、使用 React 需要注意的事项有哪些?

1.  `state` 不可直接进行修改

2.  不要在循环、条件或嵌套函数中调用 `Hook`, 必须始终在 `React` 函数的顶层使用 `Hook`

3.  列表渲染设置唯一的 `key`

4.  以大写字母作为组件的名称开头

5.  组件单一职责

6.  类组件中注意事件函数中 `this` 指向: 使用普通函数声明事件, 函数中 `this` 不指向该组件.通过 `bind()` 来改变事件函数的this或使用箭头函数声明函数.

7.  避免过度使用 `Redux`


## 二十一、React-router
+   参考: [React-router API列表](./react_router.md)
