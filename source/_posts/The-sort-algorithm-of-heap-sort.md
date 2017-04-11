---
title: 常用排序算法总结7一一堆排序
date: 2016-09-04 20:28:15
tags: [sort, algorithm]
toc: true
categories: 算法
---

在了解堆排序之前，我们有必要清楚“什么是堆呢？”。

> 堆（英语：Heap）是计算机科学中一类特殊的数据结构的统称。***堆通常是一个可以被看做一棵树的数组对象。***在队列中，调度程序反复提取队列中第一个作业并运行，因为实际情况中某些时间较短的任务将等待很长时间才能结束，或者某些不短小，但具有重要性的作业，同样应当具有优先权。堆即为解决此类问题设计的一种数据结构。

堆的逻辑定义：
![堆的逻辑定义](http://7xsd89.com1.z0.glb.clouddn.com/heap-defination.png)

堆的实现通过构造二叉堆（英语：binary heap），实为二叉树的一种；由于其应用的普遍性，当不加限定时，均指该数据结构的这种实现。这种数据结构具有以下性质。

- 任意节点小于（或大于）它的所有后裔，最小元（或最大元）在堆的根上（堆序性）。
- 堆总是一棵完全树。即除了最底层，其他层的节点都被元素填满，且最底层尽可能地从左到右填入。

将根节点最大的堆叫做***最大堆***或大根堆，根节点最小的堆叫做***最小堆***或小根堆。常见的堆有***二叉堆***、斐波那契堆等。

## 定义

> 堆排序（英语：Heap Sort）是指利用***堆***这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

![堆排序演示动画](http://7xsd89.com1.z0.glb.clouddn.com/sort_heapsort_animate.gif)

<!--more-->

## 算法步骤

堆排序的根本是进行一次堆的构建过程。

- 得到当前序列的最小(大)的元素 
- 把这个元素和最后一个元素进行交换,这样当前的最小(大)的元素就放在了序列的最后,而原先的最后一个元素放到了序列的最前面 
- 这交换可能会破坏堆序列的性质(注意此时的序列是除去已经放在最后面的元素),因此需要对序列进行调整,使之满足于上面堆的性质
- 重复上面的过程,直到序列调整完毕为止

### 堆的操作
在堆的数据结构中，堆中的最大值总是位于根节点。堆中定义以下几种操作：

- 最大堆调整（Max_Heapify）：将堆的末端子节点作调整，使得子节点永远小于父节点
- 创建最大堆（Build_Max_Heap）：将堆所有数据重新排序
- 堆排序（HeapSort）：移除位在第一个数据的根节点，并做最大堆调整的递归运算

### 堆节点的访问

通常堆是通过一维数组来实现的。在数组起始位置为0的情形中：

- 父节点i的左子节点在位置(2*i+1);
- 父节点i的右子节点在位置(2*i+2);
- 子节点i的父节点在位置floor((i-1)/2);

## 代码实现（java）

``` java
/** 堆排序的简单实现
 *
 * @param: a
 * @return: void
 * @throws
 */
public static void heapSort(int[] a)
{
    int count = a.length;

    // first place a in max-heap order
    heapify(a, count);

    int end = count - 1;
    while (end > 0) {
        // swap the root(maximum value) of the heap with the
        // last element of the heap
        int tmp = a[end];
        a[end] = a[0];
        a[0] = tmp;
        // put the heap back in max-heap order
        siftDown(a, 0, end - 1);
        // decrement the size of the heap so that the previous
        // max value will stay in its proper place
        end--;
    }
}

private static void heapify(int[] a, int count)
{
    // start is assigned the index in a of the last parent node
    int start = (count - 2) / 2; // binary heap

    while (start >= 0) {
        // sift down the node at index start to the proper place
        // such that all nodes below the start index are in heap
        // order
        siftDown(a, start, count - 1);
        start--;
    }
    // after sifting down the root all nodes/elements are in heap order
}

private static void siftDown(int[] a, int start, int end)
{
    // end represents the limit of how far down the heap to sift
    int root = start;

    while ((root * 2 + 1) <= end) { // While the root has at least one child
        int child = root * 2 + 1; // root*2+1 points to the left child
        // if the child has a sibling and the child's value is less than its
        // sibling's...
        if (child + 1 <= end && a[child] < a[child + 1])
            child = child + 1; // ... then point to the right child instead
        if (a[root] < a[child]) { // out of max-heap order
            int tmp = a[root];
            a[root] = a[child];
            a[child] = tmp;
            root = child; // repeat to continue sifting down the child now
        } else
            return;
    }
}
```

## 参考文章

- [堆](https://zh.wikipedia.org/wiki/堆_%28数据结构%29)
- [堆排序](https://zh.wikipedia.org/wiki/堆排序)
