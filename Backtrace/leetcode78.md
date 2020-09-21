### 题目描述

给定一组**不含重复元素的整数数组**_nums_, 返回改数组所有可能的子集。

**说明：**解集不能包含重复的子集。

示例：

```
输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```





### 思路

这题的思路是回溯法。

给定一个数组如`[1, 2, 3]`, 我们是怎么写出来所有的子集呢？

题目中空集也满足要求，所以不再赘述。

我们会考虑写出所有长度为1的子集：

```
[1], [2], [3]
```

跟着长度为 2的子集：

```
[1, 2], [1, 3], [2, 3]
```

然后长度是3的子集：

```
[1, 2, 3]
```

最后我们把整个都组合在一起我们就得到了答案。



那么我们把这个过程泛化一下，假设有一个长度为`n`的数组，
$$
[a_0, a_1, a_2, .... , a_n]
$$
然后需要写出长度为

$$
k
$$

的子集。

我们可以有如下选择：

选择 a0 然后我们继续在剩下的`[a1, a2.... an]`里面选择 `k-1`个数，凑成一个子集

选择 a1 然后我们继续在剩下的`[a0, a2.... an]`里面选择 `k-1`个数，凑成一个子集

...

由此可见，递归下去就可以解决。



### 代码



我们设计一个函数，函数的签名是：

```java
public void helper(List<Integer> temp, int[] nums, int start, int len)
```

`temp` 这个参数用来记录我们的结果， `nums`是数据来源， `start`是表示从数组`nums[start]`开始做选择，`len`表示我们的最终要求长度，对应上文的`k`.

```java
public void helper(List<Integer> temp, int[] nums, int start, int len) {
    // 边缘条件
    if (temp.size() == len) {
        // temp的长度已经满足答案要求，记录一下答案，返回。
        recordAns();
        return;
    }
    if (start == nums.length) return; // 没有得选择了
    for (int j = i; j < nums.length; j++) {
        temp.add(nums[j]); // 回溯里面的做选择
        helper(temp, nums, j + 1, len); 
        temp.remove(temp.size() - 1); // 撤销这一步的选择
    }
}
```



那么最终版本的答案：

```java

class Solution {

    List<List<Integer>> ans = new LinkedList<>();

    public void helper(LinkedList<Integer> temp, int[] nums, int i, int len) {
        if (temp.size() == len) {
            ans.add(new LinkedList<>(temp));
            return;
        }
        if (i == nums.length) {
            return;
        }
        for (int j = i; j < nums.length; j++) {
            temp.add(nums[j]);
            helper(temp, nums, j + 1, len);
            temp.removeLast();
        }

    }

    public List<List<Integer>> subsets(int[] nums) {
        
        LinkedList<Integer> temp = new LinkedList();
        ans.add(new LinkedList<>()); // 添加空集
        for (int i = 1; i <= nums.length; i++)
            helper(temp, nums, 0, i); // 把所有子集长度都做一遍选择
        return ans;
    }
}
```

