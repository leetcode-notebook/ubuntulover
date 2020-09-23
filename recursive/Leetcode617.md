### 题目描述



给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。

你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。

示例 1:

```
输入: 
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  
输出: 
合并后的树:
	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7
```



[原题链接](https://leetcode-cn.com/problems/merge-two-binary-trees/)

### 分析

这题属于简单题范畴。 拿出来分析这题是想记录一下递归。递归我们可能会分析一下出口：

- 如果有一个节点为空，返回另一个

递归下去的条件是：

- 两者都不为空，创建新节点，节点值为两个树节点的和。



### 代码

```java
class Solution {
  
  public TreeNode helper(TreeNode t1, TreeNode t2) {
    if (t1 == null) return t2;
    if (t2 == null) return t1;
    TreeNode root = new TreeNode(t1.val + t2.val);
    root.left = helper(t1.left, t2.left); // 递归下去
    root.right = helper(t1.right, t2.right); 
    return root; // 这里是树的先序遍历
  }
  public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
    return helper(t1, t2);
  }
}
```

