# 算法

## 前端常用算法

### 1. **排序算法**
排序算法是面试中常考的基础算法，考察对数据排序和优化的理解。

- **冒泡排序**：通过不断交换相邻元素，使数组逐渐有序。
- **选择排序**：每次找到最小或最大的元素，将其放在已排序部分的末尾。
- **插入排序**：将每个元素插入到已排序的部分，保持排序状态。
- **快速排序**：分治法的实现，通过选择一个基准元素，分割数组，递归排序。
- **归并排序**：分治法的实现，将数组递归分成两部分，分别排序后合并。
- **堆排序**：使用堆数据结构进行排序，时间复杂度为 O(n log n)。

**示例**：快速排序
```javascript
function quickSort(arr) {
  if (arr.length <= 1) return arr;
  let pivot = arr[arr.length - 1];
  let left = [];
  let right = [];
  
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

### 2. **查找算法**
查找算法用于在数据中寻找特定的元素。

- **线性查找**：逐个元素遍历，时间复杂度 O(n)。
- **二分查找**：在有序数组中通过将搜索范围折半，时间复杂度 O(log n)。

**示例**：二分查找
```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    let mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) {
      return mid;
    }
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return -1; // 未找到
}
console.log(binarySearch([1, 2, 3, 4, 5, 6, 7], 4)); // 3
```

### 3. **字符串操作**
字符串处理是前端面试中常见的基础题型。

- **反转字符串**：例如将字符串 "hello" 反转为 "olleh"。
- **字符串查找和替换**：例如找出某个子字符串的位置。
- **回文检测**：判断字符串是否为回文，如 "racecar"。
- **字符串排列**：判断两个字符串是否为字母重排，如 "listen" 和 "silent"。

**示例**：反转字符串
```javascript
function reverseString(str) {
  return str.split('').reverse().join('');
}
console.log(reverseString('hello')); // 'olleh'
```

### 4. **数组操作**
数组是前端开发中的重要数据结构，面试中常考以下算法：

- **去重**：从数组中移除重复元素。
- **数组扁平化**：将多维数组转化为一维数组。
- **最大子数组和**：如使用 Kadane 算法寻找数组中和最大的连续子数组。
- **两数之和**：在数组中找到两个数之和等于目标值。

**示例**：数组去重
```javascript
function removeDuplicates(arr) {
  return [...new Set(arr)];
}
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5])); // [1, 2, 3, 4, 5]
```

### 5. **动态规划 (Dynamic Programming)**
动态规划是用来解决优化问题，尤其是在有重叠子问题和最优子结构的场景中。

- **斐波那契数列**：通过递归或迭代计算第 `n` 个斐波那契数。
- **背包问题**：如 0/1 背包问题。
- **最小路径和**：在网格中寻找从左上到右下的最短路径。

**示例**：计算斐波那契数
```javascript
function fib(n) {
  let dp = [0, 1];
  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  return dp[n];
}
console.log(fib(10)); // 55
```

### 6. **贪心算法 (Greedy Algorithm)**
贪心算法通过选择当前最优解来构建全局最优解，通常用于求解最优子结构的问题。

- **活动选择问题**：选择最多的互不重叠的活动。
- **零钱兑换问题**：使用最少的硬币找零。

**示例**：找零钱问题
```javascript
function minCoins(coins, amount) {
  coins.sort((a, b) => b - a); // 按降序排序
  let count = 0;
  for (let coin of coins) {
    while (amount >= coin) {
      amount -= coin;
      count++;
    }
  }
  return amount === 0 ? count : -1;
}
console.log(minCoins([1, 2, 5], 11)); // 3 (5 + 5 + 1)
```

### 7. **图算法**
图算法用于解决与图相关的问题，如最短路径、连通性等。

- **深度优先搜索 (DFS)**：遍历或搜索图的一个分支，递归实现。
- **广度优先搜索 (BFS)**：逐层遍历图，广泛用于最短路径问题。
- **Dijkstra 算法**：寻找从起点到其他所有节点的最短路径。
- **A* 算法**：结合了启发式搜索和 Dijkstra 算法的最优路径搜索。

**示例**：深度优先搜索 (DFS)
```javascript
function dfs(graph, start, visited = new Set()) {
  if (visited.has(start)) return;
  visited.add(start);
  console.log(start);
  
  for (let neighbor of graph[start]) {
    dfs(graph, neighbor, visited);
  }
}

const graph = {
  A: ['B', 'C'],
  B: ['D', 'E'],
  C: ['F'],
  D: [],
  E: ['F'],
  F: []
};
dfs(graph, 'A');
```

### 8. **位运算**
位运算是对二进制位的操作，常用于优化问题或解决特定的计算问题。

- **单一数字**：找出数组中唯一出现一次的数字。
- **奇偶问题**：找出一个数组中出现次数为奇数的元素。
- **数字交换**：使用位运算交换两个变量的值。

**示例**：找出唯一出现一次的数字
```javascript
function singleNumber(nums) {
  return nums.reduce((xor, num) => xor ^ num, 0);
}
console.log(singleNumber([4, 1, 2, 1, 2])); // 4
```


## [彻底疯狂算法题库](all_algorithm.md)
