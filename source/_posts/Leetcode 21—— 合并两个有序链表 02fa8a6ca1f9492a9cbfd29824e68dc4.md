---
title: Leetcode 21—— 合并两个有序链表
date: 2021-04-17 21:22:36
tags: leetcode,算法
---

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        // 设置虚拟节点
        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;

        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                cur.next = l1;
                cur = cur.next;
                l1 = l1.next;
            } else {
                cur.next = l2;
                cur = cur.next;
                l2 = l2.next;
            }
        }

        if (l1 != null ) {
            cur.next = l1;
        } else {
            cur.next = l2;
        }

        return dummy.next;
    }
}
```