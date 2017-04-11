---
title: 去除已排序数组中的重复元素
date: 2016-09-16 20:18:12
tags: [algorithm, LeetCode]
toc: true
categories: 算法
---

## 题目描述

给定一个已排序的数组，去除数组中的重复元素，只保留一个重复的元素，并且返回新的数组长度。

要求：
不要给数组分配额外的空间，你必须使用常量的内存大小进行原地操作。

例如：
给出数组A=[1,1,2]，你的函数调用之后必须返回长度length=2，并且A现在变成[1,2]。

## 输入

一个已排序的数组，例如[1,1,2]。

## 输出

返回数组新的长度，例如length=2。

## 快慢指针法

设置fast指针遍历数组，slow指针指向不重复元素的下一位。

<!--more-->

``` java
public static int removeDuplicates(int[] nums)
{
    if (nums.length < 1)
        return nums.length;
    int slow = 1;
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow - 1]) {
            nums[slow++] = nums[fast];
        }
    }
    return slow;
}
```

动画演示：

![动画演示](http://img.blog.csdn.net/20160916210024639)

## 扩展
去除已排序数组中的重复元素，保留指定位数。
``` java
public static int removeDuplicatesN(int[] nums, int repeatN)
{
    if (nums.length <= repeatN)
        return nums.length;
    int index = repeatN;
    for (int i = repeatN; i < nums.length; i++) {
        if (nums[i] != nums[index - repeatN]) {
            nums[index++] = nums[i];
        }
    }
    return index;
}
```

## 参考文章

- [ remove-duplicates-from-sorted-array ](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)