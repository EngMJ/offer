![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc3ad5f9db8948a2872385f41d86caf6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

很多文章提到的都是新旧DOM树需要两两对比，但是没有说清楚为什么。

### 思考

1.  大家想一下，如果让你来设计将一棵树转换为另一棵树，你会怎么设计？

+   可能是直接暴力的根据index遍历比较，相同保留，不同就替换？

+   也可能是用动态规划计算新旧两个节点变换所有情况的最小DOM操作次数？Min(新增，删除，替换)

    等等，我相信还有很多种可能。

    第一种非常粗暴，第二种是假设所有操作的优先级是相同的。第二种方案也就是我们传统的diff算法的核心方案，下面我们就此展开讨论


2.  首先思考一个问题，创建一颗树的需要的复杂度是多少？

    很简单，因为树是一种递归的数据结构，需要递归的创建，复杂度O(n)。但是DOM的操作是非常耗性能的！

3.  再思考一下，将一棵树转换为另一颗树，每个节点如何操作可以最少次数的操作DOM？

    太抽象了想不清楚没关系。下面我们来简化一下问题。

4.  思考一下，将一个字符串转换为另一个字符串所需的最小操作次数，要如何计算？[wiki Edit distance](https://en.wikipedia.org/wiki/Edit_distance "https://en.wikipedia.org/wiki/Edit_distance")

    题目还是理解的不是很清晰？ 看下面的示例！[leetcode 72.编辑距离](https://leetcode-cn.com/problems/edit-distance/ "https://leetcode-cn.com/problems/edit-distance/")


### 编辑距离

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/811edf912e214a2ba6eb64ca27c52879~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

可能有部分同学已经想到了，直观的方式是用动态规划，通过这种记忆化搜索减少时间复杂度！

下面就展开介绍一下如何用动态规划解这道题

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9de27245f5b14998b713d348efbe487d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2735f872c8b24c7aa9c3db9b8a636421~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 代码

```ini
/**
 * @param {string} word1
 * @param {string} word2
 * @return {number}
 */
var minDistance = function(word1, word2) {
  //1.初始化
  let n = word1.length, m = word2.length
  let dp = new Array(n+1).fill(0).map(() => new Array(m+1).fill(0))
  for (let i = 0; i <= n; i++) {
      dp[i][0] = i
  }
  for (let j = 0; j <= m; j++) {
      dp[0][j] = j
  }
  //2.dp
  for(let i = 0; i <= n; i++) {
      for(let j = 0; j <= m; j++) {
          if(i*j) {
              dp[i][j] = word1[i-1] == word2[j-1] ? dp[i-1][j-1]: (Math.min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1)
          } else {
              dp[i][j] = i + j
          } console.log(dp[i][j])
      }
  }
  return dp[n][m]
};
```

得到字符串的最小编辑距离需要`O(n^2)`复杂度 -> 树的最小编辑距离需要`O(n^3)`。
