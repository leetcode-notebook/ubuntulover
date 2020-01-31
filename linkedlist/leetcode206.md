### 描述
反转一个单链表。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

[点击此处查看题解](https://leetcode-cn.com/problems/reverse-linked-list/)

### 思路
这题也是属于简单题。如何反转呢？
我们在遍历列表的时候，我们就把当前节点的next指针改成指向上一个元素。但是由于是单链表，节点没有引用上一个节点，因此我们需要额外的空间来存储上一个元素，当然，在更改引用之前，还需要另一个指针来存储下一个节点。想想看，要返回的是什么？

下面给出迭代版本的题解：
```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode nextTemp = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nextTemp;
    }
    return prev;
}

```

#### 复杂度分析
 - 时间复杂度：__O(n)__, n是列表长度。
 - 空间复杂度：__O(1)__，只跟踪了常数个变量。

### 递归思路
这题的递归略微难想。我们回想一下，写递归我们一般是怎么做的？
1. 递归出口
2. 递归条件

那么我们显而易见，递归出口是当给定的链表是空的，或者只有当前一个节点。
那么递归条件呢？
假设我们已经有了一个列表：

1 -> 2 -> 3 -> 4 <- 5 

我们可以看到， 5 -> 4  这个子list被反转了，此时，我们在3 这个节点上，我们希望的事情是， 4的下一个节点指向 3。 所以， 
`
3.next.next = 3
`
这就反转了。不理解的同学可以拿出纸笔画一下，相信很快就能理解这个过程。
这里要注意的是，1 的下一个节点必须指向 NULL， 如果你没有注意这个问题，你的链表会产生循环。你可以试一试长度为2的链表来试试看，模拟这个过程， 就可以很明显的看到这个错误。下面给出代码。

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode p = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return p;
}
```
####　复杂度分析
 - 时间复杂度：__O(n)__, n是列表长度。
 - 空间复杂度：__O(ｎ)__，函数的栈帧。
走到这里，相信对递归有了更深刻的理解。
