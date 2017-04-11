---
title: 常用排序算法总结6一一快速排序
date: 2016-09-03 21:06:22
tags: [sort, algorithm]
toc: true
categories: 算法
---

## 定义

>快速排序（英语：Quick Sort），又称划分交换排序（partition-exchange sort），一种排序算法，最早由东尼·霍尔提出。在平均状况下，排序n个项目要Ο(n log n)次比较。在最坏状况下则需要Ο(n2)次比较，但这种状况并不常见。事实上，快速排序通常明显比其他Ο(n log n)算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地被实现出来。

![快速排序演示动画1](http://7xsd89.com1.z0.glb.clouddn.com/sort_quicksort_animate.gif)

<!--more-->

## 算法步骤

快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为两个子序列（sub-lists）。

步骤为：

- 从数列中挑出一个元素，称为"基准"（pivot），
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

递归的最底部情形，是数列的大小是零或一，也就是永远都已经被排序好了。虽然一直递归下去，但是这个算法总会结束，因为在每次的迭代（iteration）中，它至少会把一个元素摆到它最后的位置去。

伪代码如下：

```
function quicksort(q)
     var list less, pivotList, greater
     if length(q) ≤ 1 {
         return q
     } else {
         select a pivot value pivot from q
         for each x in q except the pivot element
             if x < pivot then add x to less
             if x ≥ pivot then add x to greater
         add pivot to pivotList
         return concatenate(quicksort(less), pivotList, quicksort(greater))
     }
```

![快速排序演示动画2](http://7xsd89.com1.z0.glb.clouddn.com/quicksort_partition_example.png)

## 代码实现（java）

``` java
/** 快速排序，递归版
 *
 * @param: <E>
 * @param: arr
 * @return: List<E>
 * @throws
 */
public static <E extends Comparable<? super E>> List<E> quickSort( List<E> arr) {
    if (!arr.isEmpty()) {
        E pivot = arr.get(0); // This pivot can change to get faster results

        List<E> less = new LinkedList<E>();
        List<E> pivotList = new LinkedList<E>();
        List<E> more = new LinkedList<E>();

        // Partition
        for (E i : arr) {
            if (i.compareTo(pivot) < 0)
                less.add(i);
            else if (i.compareTo(pivot) > 0)
                more.add(i);
            else
                pivotList.add(i);
        }

        // Recursively sort sublists
        less = quickSort(less);
        more = quickSort(more);

        // Concatenate results
        less.addAll(pivotList);
        less.addAll(more);
        return less;
    }
    return arr;
}
```

## 参考文章

- [快速排序](https://zh.wikipedia.org/wiki/快速排序)
