---
title:  Leetcode 203 —— 链表中删除节点
date: 2021-04-17 21:22:36
tags: leetcode,算法
---

# 思路

设置一个哨兵节点dummy，dummy不存储数据，cur指针初始指向dummy节点，cur.next为真正的头节点。遍历链表，如果cur的后继结点（拿后继节点判断是因为如果删除cur节点会找不到前一段链表）等于指定val，则删除后继节点，如果不相等，则cur节点后移。遍历完成返回真正头节点。

# Java实现

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;
        cur.next = head;
        
        while (cur != null && cur.next != null) {
            if (cur.next.val == val) {
            cur.next = cur.next.next;
            } else {
                cur = cur.next;
            }
        }

        return dummy.next;
    }
}
```