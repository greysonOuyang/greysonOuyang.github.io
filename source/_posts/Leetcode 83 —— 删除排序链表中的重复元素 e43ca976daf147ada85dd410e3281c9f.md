---
title:  Leetcode 83 —— 删除排序链表中的重复元素
date: 2021-04-17 21:22:36
tags: leetcode,算法
---

# 思路

设置cur指针指向链表头节点，遍历链表判断当前节点是否和后继节点相等，相等则删除此后继节点，不相等则将当前节点后移。

边界处理：

注意，不需要设置虚拟头节点，cur初始时指向头节点即可；遍历实际上只需要走到倒数第二个节点即可，当cur走到倒数第二个节点的时候能够判断最后一个节点是否重复。判定倒数第二个节点的条件是当前节点不为空且其后继节点不为空。

# Java实现

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        
        ListNode cur = head;

        while (cur != null && cur.next != null) {
            if (cur.val == cur.next.val) {
                cur.next = cur.next.next;
            } else {
                cur = cur.next;
            }
        }
        return head;
    }
}
```