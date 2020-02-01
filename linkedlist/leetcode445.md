### 描述

给定两个非空链表来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储单个数字。将这两数相加会返回一个新的链表。

 

你可以假设除了数字 0 之外，这两个数字都不会以零开头。

进阶:

如果输入链表不能修改该如何处理？换句话说，你不能对列表中的节点进行翻转。

示例:

输入: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
输出: 7 -> 8 -> 0 -> 7


[点击此处查看原题](https://leetcode-cn.com/problems/add-two-numbers-ii)

###　思路

如果你是从两数相加过来的，那么其实这题对你来说不难。想想看，这题和两数相加不痛之处在哪里？

- 高位在前面，低位在后面

其他没有什么区别了。

直接说答案：

**两数之和 + 反转链表 = 两数之和II**

```java
class Solution {
	// 反转链表
    public ListNode reverseList(ListNode list) {
        if (list == null || list.next == null) return list;
        ListNode p = reverseList(list.next);
        list.next.next = list;
        list.next = null;
        return p;
    }
	// 两数之和
    public ListNode sum(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        ListNode dummy = new ListNode(0);
        ListNode p = l1, q = l2, curr = dummy;

        int carry = 0;
        while (p != null || q != null) {
            int x = p == null ? 0 :p.val;
            int y = q == null ? 0 :q.val;
            int sum = x + y + carry;
            carry = sum / 10;
            curr.next = new ListNode(sum % 10);
            curr = curr.next;
            if (p != null) p = p.next;
            if (q != null) q = q.next;

        }
        if (carry > 0) {
            curr.next = new ListNode(carry);
        }
        return dummy.next;

    }
	// 这就是答案
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        // first we review the old method.
        l1 = reverseList(l1);
        l2 = reverseList(l2);

        return reverseList(sum(l1, l2));

    }
}
```

我们继续来考虑一下更深层次的问题， 我们要如何满足进阶的要求。

首先我们会想到的是，高位在前，低位在后面，而我们的相加又是从低位往前面加的，也就是意味着，我们要从后往前来看这个链表。那么我们要考虑的是，链表不是等长的。有一种思路是，我们先把短的list，先用0来补成和长链一样，我们再从后往前走，那就可以了。

```java
class Solution{
    // 获取链表长度
    public static int listSize(ListNode node) {
        if (node == 0) return 0;
        int size = 0;
        while (node != null) {
            size++;
            node = node.next;
        }
        return size;
    }
    // 填充0， num为有多少个节点需要填充
    public static ListNode fillZero(ListNode list, int num) {
        if (num == 0) return list;
        ListNode head = list;
        int i = 0;
        while (i < num) {
            ListNode tmp = head;
            head = new ListNode(0);
            head.next = tmp;
        }
        return head;
    }
    // 相加过程。
    public static ListNode add(ListNode l1, ListNode l2, ListNode head) {
        if (l1.next == null && l2.next == null) {
            head.next = new ListNode(l1.val + l2.val);
            return head.next;
        }
        head.next = new ListNode(-1);
        head = head.next;
        ListNode node = add(l1.next == null?l1:l1.next, l2.next == null?l2:l2.next, head);
        int tmp = node.val % 10;
        head.val = tmp;
        return head;
    }
    // 那我们就可以了。
    public ListNode addTwoNumbers(ListNode 11, ListNode l2) {
        int l1Len = listSize(l1);
        int l2Len = listSize(l2);
        // 填充短链
        if (l1Len - l2Len > 0) {
            l2 = fillZero(l2, l1Len - l2Len);
        }else {
            l2 = fillZero(l1, l2Len - l1Len);
        }
        ListNode node = add(l1, l2, new ListNode(-1));
        // 处理overflow。
        if (node.val >= 10) {
            ListNode newNode = new ListNode(node.val % 10);
            node.val = node.val / 10;
            newNode.next = node.next;
            node.next = newNode;
        }
        return node;
    }
}
```

到此我们就结束了。难的地方是在于如何从后往前递归，也是一种自底向上(bottom to up)的一种递归，后面会继续开专题来讲一下递归，这个有意思的俄罗斯套娃。