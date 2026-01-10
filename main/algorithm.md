# 前端高级面试常见算法

> 精选高级前端面试中最常见的算法题，覆盖 Vue3 Diff、数据处理、树遍历等核心场景

---

## 0. 时间复杂度基础

### O(n²) - 平方复杂度

两层嵌套循环，如矩阵遍历：

```js
function square(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length; j++) {
      console.log(i, j);  // 执行 n×n 次
    }
  }
}
```

### O(n³) - 立方复杂度

三层嵌套循环：

```js
function cube(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length; j++) {
      for (let k = 0; k < arr.length; k++) {
        console.log(i, j, k);  // 执行 n×n×n 次
      }
    }
  }
}
```

### O(log n) - 对数复杂度

每次操作将问题规模减半，如二分查找：

```js
function logFunc(n) {
  if (n === 0) return "Done";
  n = Math.floor(n / 2);
  return logFunc(n);
}
// n=8 → 3次调用（8→4→2→1→0）
// log₂(8) = 3
```

### 常见复杂度对比

| 复杂度 | 名称 | 典型算法 |
|--------|------|----------|
| O(1) | 常数 | 数组访问 |
| O(log n) | 对数 | 二分查找 |
| O(n) | 线性 | 遍历数组 |
| O(n log n) | 线性对数 | 快排、归并 |
| O(n²) | 平方 | 冒泡排序 |
| O(2ⁿ) | 指数 | 斐波那契递归 |

---

## 1. 最长递增子序列（LIS）

> Vue3 Diff 算法核心，用于优化节点移动

```js
function getSequence(arr) {
  const p = arr.slice(); // 记录前驱索引
  const result = [0];    // 存储 LIS 索引序列

  const len = arr.length;
  let i, j, u, v, c;
  for (i = 0; i < len; i++) {
    const current = arr[i];
    if (current !== 0) {
      j = result[result.length - 1];
      if (arr[j] < current) {
        p[i] = j;
        result.push(i);
        continue;
      }
      // 二分查找
      u = 0;
      v = result.length - 1;
      while (u < v) {
        c = ((u + v) / 2) | 0;
        if (arr[result[c]] < current) {
          u = c + 1;
        } else {
          v = c;
        }
      }
      if (current < arr[result[u]]) {
        if (u > 0) p[i] = result[u - 1];
        result[u] = i;
      }
    }
  }
  // 回溯构造最终结果
  let uLen = result.length;
  let z = result[uLen - 1];
  while (uLen-- > 0) {
    result[uLen] = z;
    z = p[z];
  }
  return result;
}

// 测试
const arr = [10, 5, 6, 7, 4, 1, 2, 8, 9];
console.log(getSequence(arr)); // [1, 2, 3, 7, 8] -> [5, 6, 7, 8, 9]
```

**时间复杂度**: O(n log n)

---

## 2. 快速排序

```js
function quickSort(arr) {
  if (arr.length <= 1) return arr;
  const pivot = arr[arr.length - 1];
  const left = [];
  const right = [];
  
  for (let i = 0; i < arr.length - 1; i++) {
    if (arr[i] < pivot) {
      left.push(arr[i]);
    } else {
      right.push(arr[i]);
    }
  }
  
  return [...quickSort(left), pivot, ...quickSort(right)];
}

console.log(quickSort([5, 2, 9, 1, 5, 6])); // [1, 2, 5, 5, 6, 9]
```

**时间复杂度**: 平均 O(n log n)，最坏 O(n²)

---

## 3. 二分查找

```js
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return -1;
}

console.log(binarySearch([1, 2, 3, 4, 5, 6, 7], 4)); // 3
```

**时间复杂度**: O(log n)

---

## 4. 数组去重

```js
// 方法 1：Set
const unique1 = arr => [...new Set(arr)];

// 方法 2：filter + indexOf
const unique2 = arr => arr.filter((item, index) => arr.indexOf(item) === index);

// 方法 3：reduce
const unique3 = arr => arr.reduce((acc, cur) => 
  acc.includes(cur) ? acc : [...acc, cur], []);

console.log(unique1([1, 2, 2, 3, 3, 3])); // [1, 2, 3]
```

---

## 5. 数组扁平化

```js
// 方法 1：flat()
const flatten1 = arr => arr.flat(Infinity);

// 方法 2：递归
function flatten2(arr) {
  return arr.reduce((acc, val) => 
    Array.isArray(val) ? acc.concat(flatten2(val)) : acc.concat(val), 
  []);
}

// 方法 3：栈（非递归）
function flatten3(arr) {
  const stack = [...arr];
  const result = [];
  while (stack.length) {
    const item = stack.pop();
    if (Array.isArray(item)) {
      stack.push(...item);
    } else {
      result.unshift(item);
    }
  }
  return result;
}

console.log(flatten2([1, [2, [3, [4, 5]]]])); // [1, 2, 3, 4, 5]
```

---

## 6. 两数之和

```js
function twoSum(nums, target) {
  const map = new Map();
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    map.set(nums[i], i);
  }
  return [];
}

console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
```

**时间复杂度**: O(n)

---

## 7. 最大子数组和（Kadane 算法）

```js
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];
  
  for (let i = 1; i < nums.length; i++) {
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }
  
  return maxSum;
}

console.log(maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4])); // 6
```

---

## 8. 字符串操作

### 反转字符串
```js
const reverseString = str => str.split('').reverse().join('');
console.log(reverseString('hello')); // 'olleh'
```

### 回文检测
```js
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  let left = 0, right = cleaned.length - 1;
  
  while (left < right) {
    if (cleaned[left] !== cleaned[right]) return false;
    left++;
    right--;
  }
  return true;
}

console.log(isPalindrome('A man, a plan, a canal: Panama')); // true
```

---

## 9. 深度优先搜索（DFS）

> 常用于 DOM 树遍历、组件树遍历

```js
function dfs(graph, start, visited = new Set()) {
  if (visited.has(start)) return;
  visited.add(start);
  console.log(start);
  
  for (const neighbor of graph[start] || []) {
    dfs(graph, neighbor, visited);
  }
}

const graph = {
  A: ['B', 'C'],
  B: ['D', 'E'],
  C: ['F'],
  D: [], E: ['F'], F: []
};
dfs(graph, 'A'); // A B D E F C
```

---

## 10. 广度优先搜索（BFS）

> 常用于层序遍历、最短路径

```js
function bfs(graph, start) {
  const visited = new Set();
  const queue = [start];
  const result = [];
  
  while (queue.length > 0) {
    const node = queue.shift();
    if (visited.has(node)) continue;
    
    visited.add(node);
    result.push(node);
    
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        queue.push(neighbor);
      }
    }
  }
  
  return result;
}

console.log(bfs(graph, 'A')); // ['A', 'B', 'C', 'D', 'E', 'F']
```

---

## 11. 链表操作

### 链表节点定义
```js
class ListNode {
  constructor(val) {
    this.val = val;
    this.next = null;
  }
}
```

### 反转链表
```js
function reverseList(head) {
  let prev = null;
  let current = head;
  
  while (current !== null) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }
  
  return prev;
}
```

### 检测链表是否有环
```js
function hasCycle(head) {
  if (!head || !head.next) return false;
  
  let slow = head;
  let fast = head;
  
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  
  return false;
}
```

---

## 12. 二叉树遍历

### 树节点定义
```js
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = null;
    this.right = null;
  }
}
```

### 前/中/后序遍历
```js
// 前序：根 -> 左 -> 右
const preorder = root => root ? [root.val, ...preorder(root.left), ...preorder(root.right)] : [];

// 中序：左 -> 根 -> 右
const inorder = root => root ? [...inorder(root.left), root.val, ...inorder(root.right)] : [];

// 后序：左 -> 右 -> 根
const postorder = root => root ? [...postorder(root.left), ...postorder(root.right), root.val] : [];
```

### 层序遍历（BFS）
```js
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const level = [];
    const size = queue.length;
    
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    result.push(level);
  }
  
  return result;
}
```

### 最大深度
```js
const maxDepth = root => root ? 1 + Math.max(maxDepth(root.left), maxDepth(root.right)) : 0;
```

---

## 13. 位运算

### 找出唯一出现一次的数字
```js
const singleNumber = nums => nums.reduce((xor, num) => xor ^ num, 0);
console.log(singleNumber([4, 1, 2, 1, 2])); // 4
```

### 判断是否为 2 的幂
```js
const isPowerOfTwo = n => n > 0 && (n & (n - 1)) === 0;
```

### 交换两个变量（不用临时变量）
```js
function swap(a, b) {
  a = a ^ b;
  b = a ^ b;
  a = a ^ b;
  return [a, b];
}
```

---

## 14. 斐波那契数列

```js
// 动态规划（空间优化）
function fib(n) {
  if (n <= 1) return n;
  let prev = 0, curr = 1;
  for (let i = 2; i <= n; i++) {
    [prev, curr] = [curr, prev + curr];
  }
  return curr;
}

console.log(fib(10)); // 55
```

---

## 算法复杂度速查表

| 算法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|-----------|-----------|----------|
| 快速排序 | O(n log n) | O(log n) | 通用排序 |
| 二分查找 | O(log n) | O(1) | 有序数组查找 |
| LIS | O(n log n) | O(n) | Vue3 Diff |
| 两数之和 | O(n) | O(n) | 哈希查找 |
| DFS/BFS | O(V+E) | O(V) | 图/树遍历 |
| 链表反转 | O(n) | O(1) | 链表操作 |

---

## [更多算法题库](all_algorithm.md)
