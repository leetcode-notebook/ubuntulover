## 介绍
**树** 是一种非常常见的数据结构。
树的每个节点有一个根节点，还有一个包含所有子节点的列表。当然，从图的观点看，它是一个拥有`N个节点`和`N-1条边`的有向无环图。

`二叉树`是一种更加典型的树。如名字所示，一个节点最多包含两个子树的结构，通常称为“左子树”和“右子树”。

### 代码部分
我们来看看二叉树的定义。
C语言的：
```c
typedef _node TreeNode;
struct _node {
	int data;
	struct _node* left;
	struct _node* right;
};
```
Java的：
```java
class Node {
	int val;
	Node left, right;

	public Node(int val) {
		this.val = val;
		left = right = null;
	}
}
```
其他语言的都是大同小异。
我们以C语言为例子，来构建一棵树。
```c
#include<stdio.h>
#include<stdlib.h>

typedef _node TreeNode;
struct _node {
	int data;
	struct _node* left;
	struct _node* right;
};

/*构建一个节点*/
TreeNode* newNode(int);

int main(int argc, char const *argv[])
{
	TreeNode* root = newNode(1);
	/* 下面是构建出来的树。 
        1 
      /   \ 
     NULL  NULL   
  */
	root->left = newNode(2);
	root->right = newNode(3);
	 /* 2 和 3 成为了子节点。
           1 
         /   \ 
        2      3 
     /    \    /  \ 
    NULL NULL NULL NULL 
  */
	root->left->left = newNode(4);

	getchar();

	return 0;
}
TreeNode* newNode(int val) {
	TreeNode* node = malloc(sizeof(TreeNode));
	if (node == NULL) {
		fprintf(stderr, "%s\n", "malloc fail !");
		return NULL;
	}
	node->data = val;
	node->left = NULL;
	node->right = NULL;
	return node;
}
```
从代码上可以看到，二叉树是一个层次的结构。

### 二叉树的属性

看了二叉树的代码和实例之后，我们可以发现一些关于二叉树的性质。
- 在第n层中， 二叉树的节点数最多为 2^(n-1)个。
在根节点中，只有一个。
要证明这个结论，可以用递推。
- 一个高度为h的二叉树，最多有2^h - 1个节点。
- 有N个节点的二叉树，最小的高度是？ (Log2(N+1))
- 同理，有L个叶子的二叉树至少有多高（多少层）？ Log2L + 1


### 树的遍历
四种遍历： 
1. 前序遍历
2. 中序遍历
3. 后序遍历
4. 层次遍历

---
前序遍历

前序遍历就是先访问根节点，然后访问左子树， 最后遍历右子树。

           F 
         /   \ 
        B     G 
       / \     \ 
      A   D     I
         / \   /
        C   E H

 上面这个树的前序遍历是
 `F -> B -> A -> D -> C -> E -> G -> I -> H`.
 那么我们来看代码怎么写。
 Leetcode中提供了这题：

 [二叉树的前序遍历Leetcode](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)
 递归版本的很简单，直接看代码：
 ```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new LinkedList();
        helper(root, res);
        return res;
        
    }
    /**
	定义的helper函数，用来遍历。
    */
    public void helper(TreeNode root, List<Integer> res) {
        if (root != null) {
            res.add(root.val);
            if (root.left != null) {
                helper(root.left, res);
            }
            if (root.right != null) {
                helper(root.right, res);
            }
            
        }
        
    }
}
 ```
当然，从题目的角度上来说，它还希望我们使用迭代的方式来完成。
迭代的话，需要使用到栈。
具体也不是很难， 直接看代码：
```java
class Solution {
    List<Integer> res = new ArrayList();
    public List<Integer> preorderTraversal(TreeNode root) {
        // now lets use iterative method.
        Stack<TreeNode> stack = new Stack();
        if (root == null) return res;
        stack.push(root);
        while (!stack.isEmpty()) {
            TreeNode pop = stack.pop();
            res.add(pop.val);
            if (pop.right != null) stack.push(pop.right);
            if (pop.left != null) stack.push(pop.left);
            
        }
        return res;   
    }
}
```

---
 中序遍历

中序遍历就是先访问左子树， 然后访问根节点，最后右子树。

           F 
         /   \ 
        B     G 
       / \     \ 
      A   D     I
         / \   /
        C   E H

还是这个树，这个树的中序遍历是：
`A -> B -> C -> D -> E -> F -> G -> H -> I`.
注意遍历出来的顺序。
Leetcode也有中序遍历的练习，我们来看看。

[二叉树的中序遍历 Leetcode94](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)
同样的，迭代也不难，我们直接上代码：
```java
class Solution {
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
}
```
我们看看迭代的写法。
```java
public class Solution {
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
}
```
迭代的写法也是基于栈，但是写法上有点小小的注意，先走左子树，然后根节点，然后右子树。

---
后序遍历

后序遍历就是先遍历左子树，然后右子树，最后根节点。

           F 
         /   \ 
        B     G 
       / \     \ 
      A   D     I
         / \   /
        C   E H
还是这个树，这个的后序遍历是：
`A -> C -> E -> D -> B -> H -> I -> G -> F.`
同样的，我们来看后序遍历怎么写。[二叉树的后序遍历 Leetcode145](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)
```java
class Solution {
    List<Integer> res = new ArrayList();
    public List<Integer> postorderTraversal(TreeNode root) {
        if (root == null) return res;
        postorderTraversal(root.left);
        postorderTraversal(root.right);
        res.add(root.val);
        return res;
    }
}
```
同样的，为了更好的理解后序遍历，我们来看看迭代的写法怎么写。
代码有很多，我这里提供一种思路。
> 我们回顾一下前序遍历的代码，我们把左右子树都分别压栈，然后从栈里面取元素出来。需要注意的是，我们先访问左子树，根据栈的特性，我们先压栈右子树。

那我们在后序遍历的时候要考虑的一个问题是，我们到根节点的时候，不可以直接pop掉，因为我们后面还要回来。
怎么办呢？
我们把每个节点都push两次，然后判断当前pop节点和栈顶节点是否相同。
如果相同的话，就是从左子树到的根节点。
如果不同的话，就是从右子树到的根节点，此时就可以把节点加入到list里面。

```java
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> list = new ArrayList<>();
    if (root == null) {
        return list;
    }
    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode cur = stack.pop();
        if (cur == null) {
            continue;
        }
        if (!stack.isEmpty() && cur == stack.peek()) {
            stack.push(cur.right);
            stack.push(cur.right);
            stack.push(cur.left);
            stack.push(cur.left);
        } else {
            list.add(cur.val);
        }
    }
    return list;
}
```
有兴趣的同学，可以去了解一下莫里斯遍历（Morris Traversal）

---
层次遍历

二叉树的层次遍历就略为简单了，还是上面的那个树：

           F 
         /   \ 
        B     G 
       / \     \ 
      A   D     I
         / \   /
        C   E H

它的层次遍历是：
`[[F], [B, G], [A, D, I], [C, E, H]]`

每一层的节点都是一个list里面放入一个新的大list里面。这里我们要使用到一个叫队列的数据结构来帮助我们做广度优先搜索（BFS）。如果你对队列不理解的话，可以回去linkedlist那里面看我写的Quack这部分代码，它实现了栈和队列。[查看Quack](linkedlist/)区别只是在于，当你把它当栈用的时候，调用push来压栈，当把它当成队列的时候，使用qush来入队。
那么我们来看层次遍历怎么写：
```java
public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList();
        Queue<TreeNode> queue = new LinkedList();
        if (root == null) return result;
        int level = 0;
        queue.add(root); // try to start.
        while (!queue.isEmpty()) {
            result.add(new ArrayList<Integer>());
            int level_length = queue.size();
            for (int i = 0; i < level_length; i++) {
                TreeNode node = queue.remove();
                result.get(level).add(node.val);
                if (node.left != null) queue.add(node.left);
                if (node.right != null) queue.add(node.right);

            }
            level++;


        }
        return result;

```
注意到使用了一个变量来跟踪当前的层数。

---
关于删除
删除二叉树的节点是件麻烦的事情。一种思路是，按照后序遍历的方法来走，先删除它的左子树，然后删除右子树，最后删除节点自己。


### 相关有意思的题解

#### 给出中序遍历和后序遍历，恢复二叉树

根据我们上述讲的东西，我们可以来考虑题目怎么做了。我们知道，中序遍历的顺序是左子树 - 根节点 - 右子树的顺序，而后序遍历的顺序是， 左子树 - 右子树 - 根节点。我们以Leetcode的题目为例子，我们来加强对二叉树的学习理解，同时，加深对于递归的理解。
[点击此处查看原题](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

?> 要注意的一个点是，我们假设二叉树里面没有重复的元素。

我们具体来看题目：

例如，给出

中序遍历 inorder = `[9,3,15,20,7]`

后序遍历 postorder = `[9,15,7,20,3]`
返回如下的二叉树：

     3
    /  \
    9  20
      /  \
     15   7

我们观察后序遍历的最后一个元素3， 根据后序遍历的特点，它应该是整个树的根，我们回去看中序遍历，他被分成两个部分：`[9]`和`[15,20,7]`这两个部分。那么根据中序遍历的定义，`[9]`这个部分，应当属于根节点为`3`的左子树，而`[15,20,7]`这部分，则是右子树。此时后序遍历结果里面`[9, 15, 7, 20]`,我们从上面知道，`[15, 7, 20]`里面，`20`应该是右子树的根节点，`7`是`20`的右子树，`15`是`20`的左子树，`9`是`3`的左子树，整个树就恢复完成了。

思路我们有了，那我们来看代码怎么写：

