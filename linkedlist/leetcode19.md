原题目是：
给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

示例：

给定一个链表: 1->2->3->4->5, 和 n = 2.

当删除了倒数第二个节点后，链表变为 1->2->3->5.
说明：

给定的 n 保证是有效的。

[点击此处查看原题](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

这道题比较简单，考了dummyNode在删除，添加节点的时候的使用。
直接看答案：
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
        if (head == null) return head;
        // here is the typical dummy node usage.
        ListNode node = new ListNode(-1);
        node.next = head;
        // find the delete position
        int length = 0;
        ListNode tmp = head;
        while (tmp != null) {
            length++;
            tmp = tmp.next;
        }

        length -= n;
        tmp = node;
        while (length > 0) {
            length--;
            tmp = tmp.next;
        }
        tmp.next = tmp.next.next;
        return node.next;

    }
```

