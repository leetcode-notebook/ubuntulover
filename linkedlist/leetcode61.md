### 描述
给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

示例 1:

输入: 1->2->3->4->5->NULL, k = 2
输出: 4->5->1->2->3->NULL
解释:
向右旋转 1 步: 5->1->2->3->4->NULL
向右旋转 2 步: 4->5->1->2->3->NULL
示例 2:

输入: 0->1->2->NULL, k = 4
输出: 2->0->1->NULL
解释:
向右旋转 1 步: 2->0->1->NULL
向右旋转 2 步: 1->2->0->NULL
向右旋转 3 步: 0->1->2->NULL
向右旋转 4 步: 2->0->1->NULL

[点击此处查看原题](https://leetcode-cn.com/problems/rotate-list/)

### 思路
这题我的思路很直接：
1. 先把链表闭合成环
2. 找到新的地方断开

把链表闭合成环我们很简单，直接把最后一个node的下一个指针指向头就可以了。
那么我们在哪里断开呢？ 不难看出，在
`n-k%n-1`处， n为链表节点个数。
需要注意的是，k有可能比n大。
那么我们直接就上代码了：

```java
public ListNode rotateRight(ListNode head, int k) {
        // step1. know the total length.
        if (head == null || k == 0) return head;
        int length = 0;
        ListNode tmp = head;
        while (tmp != null) {
            tmp = tmp.next;
            length++;
        }
        tmp = head;
        // step 2. actually, it equals to modify k % length pos.
        int step = length - (k % length);
        // step 3. who is the last one? 
        ListNode last = head;
        while (last.next != null) {
            last = last.next;
        }
        //System.out.print(last.val);
        last.next = tmp;
        while (step > 0) {
            step--;
            last = last.next;
            head = head.next;   
        }
        last.next = null;
        // System.out.println(last.val);
        return head;
    }

```
内存效率也不是很高。。。。。。就这么看着吧。。。