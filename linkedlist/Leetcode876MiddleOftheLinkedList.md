### LeetCode 876 Middle Of the LinkedList
题目描述：
给定一个带有头结点 head 的非空单链表，返回链表的中间结点。

如果有两个中间结点，则返回第二个中间结点。

 

示例 1：

输入：[1,2,3,4,5]
输出：此列表中的结点 3 (序列化形式：[3,4,5])
返回的结点值为 3 。 (测评系统对该结点序列化表述是 [3,4,5])。
注意，我们返回了一个 ListNode 类型的对象 ans，这样：
ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, 以及 ans.next.next.next = NULL.

示例 2：

输入：[1,2,3,4,5,6]
输出：此列表中的结点 4 (序列化形式：[4,5,6])
由于该列表有两个中间结点，值分别为 3 和 4，我们返回第二个结点。
[点击此处查看相关原题](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

####　思路
##### 暴力方法
直接遍历一遍链表，然后找到相关的位置把节点返回。
```java
public ListNode middleNode(ListNode head) {
        int count = 0;
        int index;
        ListNode node = head;
        ListNode curr = head;
        while (node != null) {
            node = node.next;
            count++; // 记数
        }
        if (count == 0) return head;
            if (count % 2 == 1) {
            index = count / 2 + 1;
            while (index > 1) {
            	curr = curr.next;
            	--index; // 找到相关节点
            }
        } else {
            index = count / 2;
            while (index > 0) {
            	curr = curr.next;
            	--index;
            }
            
        }
        return curr;
        
    }
```
官方题解提供了一种快慢指针的思路：
```java
public ListNode middleNode(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }

```
复杂度分析：
- 时间复杂度：_O(N)_
- 空间复杂度：_O(1)_


