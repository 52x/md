---
title: 删除单链表中的倒数第n个节点
date: 2016-10-12 19:20:34
tags: [LeetCode, algorithm]
toc: true
categories: 算法
---

## 基本问题
如何删除单链表中的倒数第n个节点？

## 常规解法
先遍历一遍单链表，计算出单链表的长度，然后，从单链表头部删除指定的节点。

<!--more-->

## 代码实现

``` java
/** 删除单链表倒数第n个节点，常规解法.
 *
 *  @param  head
 *  @param  n
 *  @return  ListNode
 */
public static ListNode removeNthFromEnd(ListNode head, int n) {
    if(head == null) {
        return null ;
    }

    //get length of list
    ListNode p = head;
    int len = 0;

    while(p !=  null) {
        len++;
        p = p.next ;
    }

    //if remove first node
    int fromStart = len - n + 1;

    if(fromStart == 1)
        return  head. next ;

    //remove non-first node
    p = head;
    int i = 0;

    while(p != null) {
        i++;

        if(i == fromStart - 1) {
            p.next = p.next.next ;
        }

        p = p.next ;
    }

    return  head;
}
```

## 一次遍历法
使用快慢指针。快指针比慢指针提前n个单元。当快指针到达单链表尾部时，慢指针指向待删除节点的前节点。

## 代码实现
``` java
/** 删除单链表倒数第n个节点，快慢指针法.
 *
 *  @param  head
 *  @param  n
 *  @return  ListNode
 */
public static ListNode removeNthFromEnd(ListNode head, int n) {
    if(head == null)
        return null ;

    ListNode fast = head;
    ListNode slow = head;

    for(int i = 0; i < n; i++) {
        fast = fast.next ;
    }

    //if remove the first node
    if(fast == null) {
        head = head.next ;
        return head;
    }

    while(fast.next != null) {
        fast = fast.next ;
        slow = slow.next ;
    }

    slow.next = slow.next.next ;
    return head;
}
```

## 参考文章
- [Remove Nth Node From End of List](Remove-the-Nth-node-from-end-of-list)
