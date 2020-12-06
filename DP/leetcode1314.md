

给你一个 m * n 的矩阵 mat 和一个整数 K ，请你返回一个矩阵 answer ，其中每个 answer[i][j] 是所有满足下述条件的元素 mat[r][c] 的和： 

i - K <= r <= i + K, j - K <= c <= j + K 
(r, c) 在矩阵内。



示例 1：

输入：
``` 

mat =
    [[1,2,3],
     [4,5,6],
     [7,8,9]], 
     K = 1
```
输出：
``` 
[[12,21,16]
[27,45,33]
[24,39,28]]  
```
示例 2：

输入：
``` 
mat = [[1,2,3],
       [4,5,6],
       [7,8,9]], K = 2

```
输出：
``` 
[[45,45,45],
[45,45,45],
[45,45,45]]
```



提示：

m == mat.length
n == mat[i].length
1 <= m, n, K <= 100
1 <= mat[i][j] <= 100

[原题链接](https://leetcode-cn.com/problems/matrix-block-sum)



### 思路

这题其实是考虑动态规划。或者叫做矩阵前缀和，不懂相关知识的可以自行深入了解。

这里我们从动态规划的角度进行思考，假设我们有个坐标原点，为矩阵左上角的顶点，坐标为 `(0, 0)`。

那么我们规定，`dp[i][j]` 的值， 是从坐标原点，到坐标 `(i, j)`所围成的矩阵的所有数字的和。 那么很明显，

状态转移方程是：

```
dp[i][j] = mat[i][j] + do[i][j - 1] + dp[i - 1][j] - dp[i - 1][j - 1];
```

那么有了这个之后，有什么用呢？

你可以试试看画一个韦恩图，相信你就能明白的。

假设结果是rs， 那么有

`rs[i][j] = dp[r2][c2] - dp[r1-1][c2] - dp[r2][c1-1] + dp[r1-1][c1-1]`

### 代码

```java

class Solution {
     public int[][] matrixBlockSum(int[][] mat, int K) {
        int m = mat.length, n = mat[0].length;
        int[][] s = new int[m][n];      //记忆矩阵，记忆左上角所有元素的和
        s[0][0] = mat[0][0];
        for(int i = 1; i < m; ++i) s[i][0] = s[i - 1][0] + mat[i][0]; // 开头一列
        for(int j = 1; j < n; ++j) s[0][j] = s[0][j - 1] + mat[0][j]; // 开头第一行
        for(int i = 1; i < m; ++i){ // 注意这里的索引
            for(int j = 1; j < n; ++j){
                s[i][j] = mat[i][j] + s[i][j - 1] + s[i - 1][j] - s[i - 1][j - 1];
            }
        }
        int[][] r = new int[m][n];      //维恩图，容斥原理，交并补（自己画个图就明白了）
        for(int i = 0; i < m; ++i){
            for(int j = 0; j < n; ++j){
                int x1 = Math.max(0, i - K), x2 = Math.min(m - 1, i + K);
                int y1 = Math.max(0, j - K), y2 = Math.min(n - 1, j + K);
                r[i][j] = s[x2][y2]
                        - (y1 == 0? 0: s[x2][y1 - 1])
                        - (x1 == 0? 0: s[x1 - 1][y2])
                        + (x1 == 0 || y1 == 0? 0: s[x1 - 1][y1 - 1]);
            }
        }
        return r;
    }


}
```





