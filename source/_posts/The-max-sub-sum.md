---
title: 最大子串和
date: 2016-07-31 16:52:41
tags:
categories: 算法
---


## 问题描述

> 在长度为N的整形数组中，求连续子串的和的最大值。

> 例如：1 2 4 5 -11 5 -3，结果为6。

> 注意：要考虑到数组中元素都为负数的情况。

## O(n)解法

```java
public static int maxSubSum(int[] a) {
    int maxSum = a[0];
    int curSum = 0;
    for (int j = 0; j < a.length; j++) {
        curSum += a[j];
        if (curSum > maxSum) {
            maxSum = curSum;
        } else if (curSum < 0) {
            curSum = 0;
        }
    }
    return maxSum;
}
```

> **本算法的关键在于，对于一个子序列，如果其和为负数，则不可能作为最大子串的前缀。**
