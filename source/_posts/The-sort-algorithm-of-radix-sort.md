---
title: 常用排序算法总结8一一基数排序
date: 2016-09-05 21:25:32
tags: [sort, algorithm]
toc: true
categories: 算法
---

## 定义

> 基数排序（英语：Radix Sort）是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

![基数排序过程](http://7xsd89.com1.z0.glb.clouddn.com/sort_radix.jpg)

<!--more-->

## 算法步骤

- 将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。
- 然后，从最低位开始，依次进行一次排序。
- 这样从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列。

基数排序的方式可以采用***LSD***（Least significant digital）或***MSD***（Most significant digital），LSD的排序方式由键值的最右边开始，而MSD则相反，由键值的最左边开始。

> 注意本次演示采用LSD的方式实现

## 代码实现（java）

``` java
/** 基数排序的简单实现，目前只能排序正整数
 *
 * @param: nums
 * @return: int[]
 * @throws
 */
public static int[] radixSort(int[] nums)
{
    int BASE_NUM = 10; // 整数基数
    int len = nums.length;
    int[] buffer = new int[len];

    int maxValue = nums[0], exp = 1;

    // 找出nums数组中最大的数
    for (int i = 1; i < len; i++) {
        if (nums[i] > maxValue) {
            maxValue = nums[i];
        }
    }

    while (maxValue / exp > 0) {
        int[] bucket = new int[BASE_NUM];

        for (int i = 0; i < bucket.length; i++) {
            bucket[i] = 0;
        }

        // 从数的低位开始进行桶排序
        for (int i = 0; i < len; i++) {
            bucket[(nums[i] / exp) % BASE_NUM]++;
        }

        // 按照当前位给nums排序
        // 确定各个数对应的大概位置buket[(nums[i] / exp) % BASE]的值
        // 即为新位置的下标
        for (int i = 1; i < BASE_NUM; i++) {
            bucket[i] += bucket[i - 1];
        }

        // 按当前位进行排序存入到新数组
        for (int i = len - 1; i >= 0; i--) {
            int index = (nums[i] / exp) % BASE_NUM;
            buffer[--bucket[(nums[i] / exp) % BASE_NUM]] = nums[i];
        }

        for (int i = 0; i < len; i++) {
            nums[i] = buffer[i];
        }

        exp *= BASE_NUM;
    }
    return nums;
}
```

## 参考文章

- [基数排序](https://wikipedia.org/wiki/%E5%9F%BA%E6%95%B0%E6%8E%92%E5%BA%8F#C)
