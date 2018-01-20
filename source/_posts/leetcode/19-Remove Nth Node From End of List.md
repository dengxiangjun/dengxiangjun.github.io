---
title: 19. Remove Nth Node From End of List
tags:
    - leetcode 
    - 单链表
    - 双指针
---
Given a linked list, remove the nth node from the end of list and return its head.

For example,

   Given linked list: 1->2->3->4->5, and n = 2.

   After removing the second node from the end, the linked list becomes 1->2->3->5.
Note:
Given n will always be valid.
Try to do this in one pass.
## 双指针
两个指针pre和cur,cur先走n步。如果此时cur==null，说明已经走到了尾部，需要删除头指针，直接返回head.next；否则，cur和pre一起移动直道cur到达尾部，然后删除pre.next结点。
```java
    public static ListNode removeNthFromEnd(ListNode head, int n) {
        if (head.next == null) return null;
        ListNode pre = head, cur = head;
        while (n > 0) {
            cur = cur.next;
            n--;
        }
        if (cur == null) return head.next;
        while (cur.next != null) {
            cur = cur.next;
            pre = pre.next;
        }
        pre.next = pre.next.next;
        return head;
    }
```

