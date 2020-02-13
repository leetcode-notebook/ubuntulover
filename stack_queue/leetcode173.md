### 描述

实现一个二叉搜索树迭代器。你将使用二叉搜索树的根节点初始化迭代器。

调用 next() 将返回二叉搜索树中的下一个最小的数。

 

示例：

        7 
      /   \ 
     3     15 
          /  \
         9    20
    
BSTIterator iterator = new BSTIterator(root);
iterator.next();    // 返回 3
iterator.next();    // 返回 7
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 9
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 15
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 20
iterator.hasNext(); // 返回 false


提示：

next() 和 hasNext() 操作的时间复杂度是 O(1)，并使用 O(h) 内存，其中 h 是树的高度。
你可以假设 next() 调用总是有效的，也就是说，当调用 next() 时，BST 中至少存在一个下一个最小的数。
在真实的面试中遇到过这道题

[点击查看原题](https://leetcode-cn.com/problems/binary-search-tree-iterator)

### 思路

这题乍一看好像很难，但是请你回顾一下二叉搜索树的性质还有二叉树的中序遍历。

- 二叉搜索树(BST)的一个性质是，左孩子的值 <= 根节点的值 <= 右孩子的值。
- 二叉树的中序遍历是： 左孩子 - 根节点 - 右孩子

明白这两个点，这题你就手到擒来。

那么我们来复习一下二叉树的中序遍历的写法，递归的版本：

```java
public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList();
        helper(result, root);
        return result;
    }
    
    public void helper(List<Integer> list, TreeNode root) {
        if (root == null) return;
        helper(list, root.left);
        list.add(root.val);
        helper(list, root.right);
        
    }
```

迭代的也很简单：

```java
public List < Integer > inorderTraversal(TreeNode root) {
        List < Integer > res = new ArrayList < > ();
        Stack < TreeNode > stack = new Stack < > ();
        TreeNode curr = root;
        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            curr = stack.pop();
            res.add(curr.val);
            curr = curr.right;
        }
        return res;
    }
```

OK，这题答案就是中序遍历二叉搜索树，需要注意的是，这里用个队列来收集答案。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class BSTIterator {
    // looks like the in-order of binary tree. 
    // because it is a bst.
    Stack<TreeNode> stack;
    Queue<Integer> iterator;
    public BSTIterator(TreeNode root) {
        // so I think this one should 
        stack = new Stack();
        iterator = new LinkedList();

        // in-order 
        TreeNode curr = root;
        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            curr = stack.pop();
            iterator.add(curr.val);
            curr = curr.right;
        }
        
    }
    
    /** @return the next smallest number */
    public int next() {
        return iterator.poll();
    }
    
    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        return !iterator.isEmpty();
    }
}

/**
 * Your BSTIterator object will be instantiated and called as such:
 * BSTIterator obj = new BSTIterator(root);
 * int param_1 = obj.next();
 * boolean param_2 = obj.hasNext();
 */
```

那么这里存在一个问题，每次我们初始化迭代器，都是把一个二叉搜索树完全的塞进去了。那我们可以不用塞全部的，请看简化的：

```java
lass BSTIterator {

        Stack<TreeNode> stack = new Stack<>();

        public BSTIterator(TreeNode root) {
            push(root);
        }

        public void push(TreeNode root) {
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
        }

        /**
         * @return the next smallest number
         */
        public int next() {
            TreeNode node = stack.pop();
            if (node.right != null) {
                push(node.right);
            }
            return node.val;


        }

        /**
         * @return whether we have a next smallest number
         */
        public boolean hasNext() {
            return !stack.isEmpty();
        }
    }

```

大家注意多动脑子，多思考。