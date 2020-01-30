### 描述
删除单链表中等于给定值val的所有节点。
例如：
`1->2->3->4`, val = 2
输出：
1->3->4

[点击此处看原题](https://leetcode-cn.com/problems/remove-linked-list-elements/)

### 思路
这题的思路很直观，但是这题是经典的dummy Node使用的一个典型例子。同时也是理解递归的好题目。

思路非常简单，就是典型的删除节点的代码。
下面直接给出代码，供参考。
这是迭代版本，使用了dummy Node。
```java
class Solution {
  public ListNode removeElements(ListNode head, int val) {
    ListNode sentinel = new ListNode(0); // 构造哑节点
    sentinel.next = head;

    ListNode prev = sentinel, curr = head;
    while (curr != null) {
      if (curr.val == val) prev.next = curr.next;  // 如果符合要求，就删除
      else prev = curr; // 否则就下一个。
      curr = curr.next;
    }
    return sentinel.next;
  }
}

```
上述代码非常直观。下面我们可以研究一下递归。

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        // recursive
        if (head == null) return head; // 递归出口
        head.next = removeElements(head.next, val);
        return head.val == val? head.next:head; 
    }
}
```

>>> 愿追梦路上的人都有收获！