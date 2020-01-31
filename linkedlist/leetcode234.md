### 描述
请判断一个链表是否为回文链表。

示例 1:

输入: 1->2
输出: false
示例 2:

输入: 1->2->2->1
输出: true

[点击此处查看原题](https://leetcode-cn.com/problems/palindrome-linked-list/)


### 思路

这个题目在难度分类是简单题，但是这题给我很多启发，个人认为应该属于中等题。
题目有个进阶的要求是，能否在O(n)和O(1)的空间复杂度解决这个问题，首先我们考虑一种比较暴力的想法是，我们开一个数组，然后遍历一下这个链表，把值都抄进去，那么我们可以用双指针的方法来判断是不是回文了(因为单链表没法倒着来)。那么这里就不浪费时间在这个代码上，大家可以自行把代码写完。

这里想介绍一个递归的思路，然后加强一下自己对递归的理解。
我们再次回顾一下递归，我们想一下反向打印字符串的代码。

假如我们有个char[],我们怎么反向打印呢？

我们可能可以想出答案：自底向上。


```c
void printArr(const char* str) {
	if (*str == '\0') {
		return;
	}
	printArr(str + 1);
	putchar(*str);
}
```
举一反三， 栈帧可以帮助我们来从后往前，那我们只需要一个从前往后的指针就可以了。

```java
class Solution {
    ListNode frontHead; // 从前往后的指针

    public boolean recursiveCheck(ListNode head) {
        if (head != null) {
            if (!recursiveCheck(head.next)) return false; // 我们先摸到后面来
            if (head.val != frontHead.val) return false;
            frontHead = frontHead.next;
        }
        return true;
    }
    public boolean isPalindrome(ListNode head) {
        frontHead = head;
        return recursiveCheck(head);
    }
}
```

相信走到这里，对于递归的理解能加深不少。
当然，这个解法也不是符合进阶要求的O(n)时间复杂度，O(1)空间复杂度的解法。
大家有兴趣可以去探索一下。


