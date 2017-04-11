---
title: 去除已排序链表中的重复元素
date: 2016-09-17 21:08:12
tags: [LeetCode, algorithm]
toc: true
categories: 算法
---

## 题目描述

给定一个已排序的单链表，去除单链表中的重复元素，只保留一个重复的元素，并且返回新的单链表。

例如：
给出1->1->2，你的函数调用之后必须返回1->2。

## 输入

一个已排序的单链表，例如1->1->2。

## 输出

返回1->2。

## 代码示例

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

public static ListNode deleteDuplicates(ListNode head)
{
    if (head == null) {
        return null;
    }
    ListNode cur, prev;
    prev = head;
    cur = head.next;
    while (cur != null) {
        if (cur.val == prev.val) {
            prev.next = cur.next;
        } else {
            prev = cur;
        }
        cur = prev.next;
    }
    return head;
}
```

![算法演示](http://img.blog.csdn.net/20160917143349791)

<!--more-->

## 扩展

去除单链表中重复元素，不保留任何重复的元素。

例如：
1->1->2->3->3->4，返回2->4

``` java
public static ListNode deleteDuplicatesAll(ListNode head)
{
    if (head == null) {
        return head;
    }

    ListNode dummy = new ListNode(Integer.MAX_VALUE); // 头结点
    dummy.next = head;
    ListNode prev, cur;
    prev = dummy;
    cur = head;
    while (cur != null) {
        boolean duplicated = false;
        while (cur.next != null && cur.val == cur.next.val) {
            duplicated = true;
            cur = cur.next;
        }
        if (duplicated) { // 删除重复的最后一个元素
            cur = cur.next;
            continue;
        }
        prev.next = cur;
        prev = prev.next;
        cur = cur.next;
    }
    prev.next = cur;
    return dummy.next;
}
```

## 参考文章

- [remove-duplicates-from-sorted-list](https://leetcode.com/problems/remove-duplicates-from-sorted-list/)
