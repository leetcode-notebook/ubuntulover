### 描述

对单链表进行插入排序。

#### 什么是插入排序？

插入排序其实也是一种很简单的排序方法，时间复杂度是O(n^2)。网上介绍这个排序算法的文章有很多，个人推荐去看算法导论的相关章节，讲的非常清楚。个人的理解是，类似我们平时玩扑克牌，手上有一把乱序的牌，每次我们拿一张牌出来，再看我们已经是有序的序列里面，我们找到他的位置，然后进行插入。（这也是它为什么叫插入排序？）

附上原题链接：

[点击此处查看原题](https://leetcode-cn.com/problems/insertion-sort-list/)

### 思路

这题思路比较简单，明白了插入排序我们就好下手了。

这里主要是注意代码的写法。

我们需要一个哨兵节点（dummy node）来保存头结点的信息。

然后我们需要两个辅助节点来帮我们交换节点。话不多说，贴代码：

```java
public ListNode insertionSortList(ListNode head) {
        if (head == null || head.next == null) return head;
        ListNode dummy = new ListNode(0); // 哨兵节点的使用
        dummy.next = head;
        ListNode h1;
        ListNode h2 = head.next;
        head.next = null;
        while (h2 != null) {
            h1 = dummy;
            // find the position.
            while (h1.next != null && h1.next.val <= h2.val) {
                h1 = h1.next;
            }
            ListNode nextTemp = h2.next;
            h2.next = h1.next;
            h1.next = h2;
            h2 = nextTemp;

        }
        return dummy.next;
    }
```

大致上就是这样，代码也比较简单，不懂的自己可以画个图来理解，感觉这几天的刷题还是对自己有点帮助的。多刷题就有进步。

> > 愿在追梦路上的人都有收货！

