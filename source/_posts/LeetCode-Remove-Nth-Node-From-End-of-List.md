---
title: LeetCode-Remove Nth Node From End of List
date: 2016-06-06 11:12:17
tags: LeetCode
toc: true
categories: 算法
---

## Question

> Given a linked list, remove the nth node from the end of list and return its head.
For example,
   Given linked list: 1->2->3->4->5, and n = 2.
   After removing the second node from the end, the linked list becomes 1->2->3->5.
Note:
Given n will always be valid.
Try to do this in one pass.

## 解说

这道题的意思是，如何反向删除链表的第n个节点，最后返回链表。

<!--more-->
## Quick Navigation
[Solution](#Solution)
[Approach #1 (Two pass algorithm)](#Approach1)
[Approach #2 (One pass algorithm)](#Approach2)

## <span id="Solution">Solution</span>

### <span id="Approach1">Approach #1 (Two pass algorithm)</span>

#### Intuition

We notice that the problem could be simply reduced to another one : Remove the (L - n + 1)th node from the beginning in the list , where L is the list length. This problem is easy to solve once we found list length L.

#### Algorithm

First we will add an auxiliary "dummy" node, which points to the list head. The "dummy" node is used to simplify some corner cases such as a list with only one node, or removing the head of the list. On the first pass, we find the list length L. Then we set a pointer to the dummy node and start to move it through the list till it comes to the (L - n)th node. We relink next pointer of the (L - n)th node to the (L - n + 2)th node and we are done.

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    int length  = 0;
    ListNode first = head;
    while (first != null) {
        length++;
        first = first.next;
    }
    length -= n;
    first = dummy;
    while (length > 0) {
        length--;
        first = first.next;
    }
    first.next = first.next.next;
    return dummy.next;
}
```

### <span id="Approach2">Approach #2 (One pass algorithm)</span>

#### Algorithm

The above algorithm could be optimized to one pass. Instead of one pointer, we could use two pointers. The first pointer advances the list by n+1 steps from the beginning, while the second pointer starts from the beginning of the list. Now, both pointers are exactly separated by nn nodes apart. We maintain this constant gap by advancing both pointers together until the first pointer arrives past the last node. The second pointer will be pointing at the nnth node counting from the last. We relink the next pointer of the node referenced by the second pointer to point to the node's next next node.

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode first = dummy;
    ListNode second = dummy;
    // Advances first pointer so that the gap between first and second is n nodes apart
    for (int i = 1; i <= n + 1; i++) {
        first = first.next;
    }
    // Move first to the end, maintaining the gap
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    second.next = second.next.next;
    return dummy.next;
}
```

## Reference
- [LeetCode-Remove Nth Node From End of List](https://leetcode.com/articles/remove-nth-node-end-list/)
