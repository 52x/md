---
title: 单链表反转问题
date: 2016-10-10 20:12:02
tags: [LeetCode, algorithm]
toc: true
categories: 算法
---

## 基本问题
如何将单链表反转？

## 单链表结构定义
``` java
/** 单链表定义
 *
 * @author: crane-yuan
 * @date: 2016-9-17 下午12:11:13
 */
public class ListNode {
    public int val;
    public ListNode next;

    public ListNode(int x) {
        val = x;
    }
}
```

<!--more-->

## 算法实现
``` java
/** 单链表反转
 *
 * @param head
 * @return ListNode
 */
public static ListNode reverseList(ListNode head) {
    if (head == null) {
        return head;
    }
    ListNode prev = null ;
    ListNode current = head;
    ListNode next = null ;
    while (current != null) {
        next = current.next ;
        current.next = prev;
        prev = current;
        current = next;
    }
    head = prev;
    return head;
}
```
## 进阶问题
如何将单链表在指定区间内进行反转？

## 问题分析
这个问题是上面问题的一个变形，难度也加大了不少，主要的难点之处就在于对边界条件的检查。
实现思路，主要就是按照给定的区间得到需要整体反转的一个子链表然后进行反转，最后就是把链表按正确的顺序拼接在一起。

## 算法实现
``` java
/** 单链表反转，反转指定区间内的节点
 *
 * @param head
 * @param m
 * @param n
 * @return ListNode
 */
public static ListNode reverseBetween(ListNode head, int m, int n) {
    // 合法性检测
    if (head == null || m >= n || m < 1 || n < 1) {
        return head;
    }
    /** 将链表按[m,n]区间分成三段
     *
     * first,second,third分别为每一段的头节点(注意，m=1也就是first与second相等的情况的处理)
     * first --> firstTail
     * second
     * third
     */
    ListNode first = head;
    ListNode firstTail = first;
    ListNode second = first;
    ListNode third = first;
    ListNode current = first;
    int  i = 0;
    while (current != null) {
        i++;
        if (i == m - 1) {
            firstTail = current;
        }
        if (i == m) {
            second = current;
        }
        if (i == n) {
            third = current.next ;
            break ;
        }
        current = current.next ;
    }
    // 进行中间second段的reverse
    current = second;
    ListNode prev = third;
    ListNode next =  null ;
    while (current != third) {
        next = current. next ;
        current. next  = prev;
        prev = current;
        current = next;
    }
    if (m == 1) {
        first = prev;
    } else {
        firstTail.next = prev;
    }
    return first;
}
```

## 参考文章
- [reverse-linked-list](https://leetcode.com/problems/reverse-linked-list/)
