---
title: 常用排序算法总结1一一冒泡排序 
date: 2016-08-28 19:38:42
tags: [sort, algorithm]
toc: true
categories: 算法
---

## 定义

> 冒泡排序（英语：Bubble Sort）是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

![冒泡排序过程](http://7xsd89.com1.z0.glb.clouddn.com/sort_bubble_animate.gif)

<!--more-->

## 算法步骤

冒泡排序算法的运作如下：

- 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
- 针对所有的元素重复以上的步骤，除了最后一个。
- 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

伪代码如下：
```
function bubble_sort (array, length) {
    var i, j;
    for(i from 0 to length-1){
        for(j from 0 to length-1-i){
            if (array[j] > array[j+1])
                swap(array[j], array[j+1])
        }
    }
}
```

## 代码实现（java）

### 一般实现

``` java
/** 冒泡排序简单实现
 *
 * @param: nums
 * @return: void
 */
public static void bubbleSort(int[] nums)
{
    int tmp;
    for (int i = 0; i < nums.length; i++) {
        for (int j = 0; j < nums.length - i - 1; j++) {
            if (nums[j] > nums[j + 1]) {
                tmp = nums[j];
                nums[j] = nums[j + 1];
                nums[j + 1] = tmp;
            }
        }
    }
}
```

### 通用实现

``` java
/** 冒泡排序的简单实现(支持泛型)
 * 
 * @param: <E>
 * @param: comparable
 * @return: void
 * @throws
 */
public static <E extends Comparable<? super E>> void bubbleSort(
            E[] comparable)
{
    E tmpE;
    for (int i = 0; i < comparable.length; i++) {
        for (int j = 0; j < comparable.length - i - 1; j++) {
            if (comparable[j].compareTo(comparable[j + 1]) > 0) {
                tmpE = comparable[j];
                comparable[j] = comparable[j + 1];
                comparable[j + 1] = tmpE;
            }
        }
    }
}
```

## 参考文章

- [排序算法](https://zh.wikipedia.org/wiki/排序算法)
- [冒泡排序](https://zh.wikipedia.org/wiki/冒泡排序#JAVA)
