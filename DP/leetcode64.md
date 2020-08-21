

### 描述

给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

说明：每次只能向下或者向右移动一步。

示例:

输入:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 7
解释: 因为路径 1→3→1→1→1 的总和最小。


原题链接： https://leetcode-cn.com/problems/minimum-path-sum

### 思路

如果你看过前面 120 的题解，相信这题你可以秒杀。

本质上，他还是开额外的空间，来避免重复的计算。

这题跟120 题很类似，如果你没思路，可以按照我那个思路来写。

### 题解

```java
public int minPathSum(int[][] grid) {

        int m = grid.length;
        int n = grid[0].length;

        int[][] dp = new int[m][n];
        int sum = 0;
        for (int i = 0; i < n; i++) {
            sum += grid[0][i];
            dp[0][i] = sum;
        }
        sum = 0;
        for (int i = 0; i < m; i++) {
            sum += grid[i][0];
            dp[i][0] = sum;
        }
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[m-1][n-1];

    }
```

