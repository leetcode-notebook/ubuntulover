### 描述

给定一个链表和一个特定值 x，对链表进行分隔，使得所有小于 x 的节点都在大于或等于 x 的节点之前。

你应当保留两个分区中每个节点的初始相对位置。

示例:

输入: head = 1->4->3->2->5->2, x = 3
输出: 1->2->2->4->3->5
### 思路
这题的思路官方题解和我想的类似。我想的是局部排序问题，官方题解的更为清晰，因此这里详细阐述一下官方题解。
根据题意，我们分析可以得知，需要找到一个点，把原来的list分割成两个部分，`larger` 和 `smaller`， 而且， `larger` + `smaller` = `original`。
那么这题就很显然了：
创建两个dummy node;
遍历整个list；
拼接两个list。

不多说废话，上代码：
### 代码
```java
public ListNode partition(ListNode head, int x) {
        if (head == null) return head;
        ListNode before = new ListNode(-1);
        before.next = null;
        ListNode after = new ListNode(-1);
        after.next = null;
        ListNode node = after;
        ListNode res = before;

        while (head != null) {
            // System.out.print(head.val + "-");
            if (head.val < x) {
                before.next = head;
                before = before.next;
            }else {
                after.next = head;
                after = after.next;
            }
            head = head.next;
        }
        // how to combine ?
        after.next = null;
        before.next = node.next;
        return res.next;
    }
```
时间复杂：O（n）， 我们遍历了一边list

空间复杂度：O（1）， 我们没有开新的空间。