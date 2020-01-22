### 描述

原题目是找到两个单链表的相交的起始节点。

编写一个程序，找到两个单链表相交的起始节点。

如下面的两个链表：



在节点 c1 开始相交。

 

示例 1：



输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Reference of the node with value = 8
输入解释：相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。


示例 2：



输入：intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Reference of the node with value = 2
输入解释：相交节点的值为 2 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。


示例 3：



输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
输入解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
解释：这两个链表不相交，因此返回 null。

[点击此处来查看原题](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

### 思路

这题有很多思路，比较暴力的就是对于每个节点来进行一次遍历。

时间复杂度：O(mn)

空间复杂度: O(1)

还有一种思路就是哈希表，使用哈希表来存储节点，来找到交点。

时间复杂度：O(m+n)

空间复杂度: O(m+n)

比较好玩的是第三中思路，快慢指针。

附上我的奇葩思路。。。

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        int lengthA = 0, lengthB = 0;
        ListNode tA = headA;
        ListNode tB = headB;
        // get the length
        while (tA != null) {
            tA = tA.next;
            lengthA++;
        }
        while (tB != null) {
            tB = tB.next;
            lengthB++;
        }
        if (lengthA >= lengthB) {
            // let it move lengthA - lengthB steps.
            int step = lengthA - lengthB;
            
            for (int i = 0; i < step; i++) {
                headA = headA.next;
            }            
            // then lets go.
            for (int i = step; i < lengthA; i++) {
                if (headA == headB) {
                    return headB;
                }
                headA = headA.next;
                headB = headB.next;
            }
            return null;
        }else {
            // let it move lengthA - lengthB steps.
            int step = lengthB - lengthA;
            
            for (int i = 0; i < step; i++) {
                headB = headB.next;
            }
            // then lets go.
            for (int i = step; i < lengthB; i++) {
                if (headA == headB) {
                    return headA;
                }
                headA = headA.next;
                headB = headB.next;
            }
            return null;
        }
    }
```

代码有点臭， 但是思路应该很明白了。。。。