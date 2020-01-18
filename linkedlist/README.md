### LinkedList的概念
单链表中的每个结点不仅包含值，还包含链接到下一个结点的引用字段。通过这种方式，单链表将所有结点按顺序组织起来。

这是C语言的定义：
```c
typedef _node Linkedlist;
struct _node {
	int val; // data here
	struct _node* next;
};
```
Java的写法类似：
```java
public class Linkedlist {
	int val;
	Linkedlist next;
	// constructor.
	Linkedlist(int x) {this.val = x;}
}
```

绝大多数的时候，我们会用头结点来表示整个列表。一个成语可以很好的解释这个：**顺藤摸瓜**

### LinkedList的相关操作
我们设计出一个数据结构，势必要涉及到跟他相关的一些操作：增删改查。
我们首先来看如何往单链表里面添加元素。
#### 添加
1. 使用给定的值去初始化新的节点cur;
2. 插入链条当中。

[LeetCode876 题解](linkedlist/Leetcode876MiddleOftheLinkedList.md)

[LeetCode19 题解](linkedlist/leetcode19.md)

[LeetCode 61题解](linkedlist/leetcode61.md)