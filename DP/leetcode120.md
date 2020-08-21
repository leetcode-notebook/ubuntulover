### 描述

给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。

相邻的结点 在这里指的是 下标 与 上一层结点下标 相同或者等于 上一层结点下标 + 1 的两个结点。

 

例如，给定三角形：

[
     [**2**],
    [**3**,4],
   [6,**5**,7],
  [4,**1**,8,3]
]
自顶向下的最小路径和为 11（即，2 + 3 + 5 + 1 = 11）。


[点击此处查看原题](https://leetcode-cn.com/problems/triangle)

### 思路

我的思维过程可能旁人看起来会比较奇怪。既然谈到动态规划，我们首先想到的是状态转移方程怎么列出来。OK， 我们来想这个题，

我们来看一个小一点的例子：

```java 
  1
 / \
2   3
```

这里， 我们发现，我们最短的路径是 1 + 2， 因为在1 可以去到的通路上，2是最小的值。

那么我们稍微归纳一下，假设我们有个函数可以完成这样的一件事情：

```java
public int minSum(int[][] arr, int i, int j)
// i 表示第 i 行， j 表示第 j 个数字
// 这个函数我们假设已经写出来，能够帮我们找到，在arr[i][j]位置上的最短路径和。
// 比如， 上面小三角形的例子， minSum(arr, 0, 0) = 3
```



那么抽象点，我们的状态转移方程就是：

```java
minSum(arr, i, j) = Math.min(minSum(arr, i+1, j), minSum(arr, i+1, j+1))
```

我的解决递归的思路是，先写出一个暴力的递归解，然后我们可以来优化他。（那种一眼秒出答案的神仙不是我，是tth）

```java
public int minSum(int[][] arr, int i, int j) {
 		if (i == arr.length - 1) {
      // 在数组的最下面一列，我们自然返回他自己。
      return arr[i][j];
    }
  	int res = Math.min(minSum(arr, i+1, j), minSum(arr, i+1, j+1));
  	return res;
}
```

我们稍微把递归树给画出来，我们可以发现这样的一个规律：

```java
// 为了方便，我直接拿上图的数字作为函数的参数了。
minSum(2) = Math.min(minSum(3), minSum(4))
minSum(3) = Math.min(minSum(6), **minSum(5)**)
minSum(4) = Math.min(**minSum(5)**, minSum(7))
.
.
.
```

可以发现，如果单纯的递归，我们会发现在计算 minSum(3) 和 minSum(4)的时候， minSum(5)被重复计算了。

我们可以开额外的空间，把这个东西记住，就减少了不必要的计算。

上我的题解：

```java

public int minimumTotal(List<List<Integer>> triangle) {
        int n = triangle.size();
        int[][] f = new int[n][n];
        f[0][0] = triangle.get(0).get(0);
        for (int i = 1; i < n; ++i) {
            f[i][0] = f[i - 1][0] + triangle.get(i).get(0);
            for (int j = 1; j < i; ++j) {
                f[i][j] = Math.min(f[i - 1][j - 1], f[i - 1][j]) + triangle.get(i).get(j);
            }
            f[i][i] = f[i - 1][i - 1] + triangle.get(i).get(i);
        }
        int minTotal = f[n - 1][0];
        for (int i = 1; i < n; ++i) {
            minTotal = Math.min(minTotal, f[n - 1][i]);
        }
        return minTotal;
}
```

虽然不是双百，但是也能看吧。。。。。。