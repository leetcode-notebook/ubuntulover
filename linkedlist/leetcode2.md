### 描述
给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：

输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807

[点击此处查看原题](https://leetcode-cn.com/problems/add-two-numbers/)

###　思路

经过那么多的链表专题训练，回来看这个题，其实不难了。
我们来仔细分析一下难点和出问题的点在哪里。
首先，让我们模拟一下我们手工做加法的时候的情况，每一位相加，如何处理进位？
我们想到运算符的　`%` 和　`/` 这两个运算。
当然，我们需要一个哑节点来构造一个新的链表，这有利于我们写代码。

话不多说，丢代码：
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        // dummy node.
        ListNode dummy = new ListNode(-1); 
        ListNode p = l1, q = l2, curr = dummy;
        int carry = 0;
        while (p != null || q != null) {
            int x = p != null ? p.val : 0;
            int y = q != null ? q.val : 0; 
            int sum = x + y + carry;
            carry = sum / 10; // 这里注意进位上面的处理.
            curr.next = new ListNode(sum % 10); // 构造答案
            curr = curr.next;
            if (p != null) p = p.next;
            if (q != null) q = q.next;   
        }
        if (carry > 0) {
            curr.next = new ListNode(carry);
        }
        return dummy.next;
    }
}

```
