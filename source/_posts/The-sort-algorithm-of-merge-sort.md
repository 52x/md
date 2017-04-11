---
title: 常用排序算法总结4一一归并排序
date: 2016-08-30 20:48:22
tags: [sort, algorithm]
toc: true
categories: 算法
---

## 概念

> 归并排序（英语：Merge sort），是创建在归并操作上的一种有效的排序算法，效率为O(n log n)。
**归并操作**（merge），也叫归并算法，指的是将两个已经排序的序列合并成一个序列的操作。归并排序算法依赖归并操作。

![归并排序算法演示动画](http://7xsd89.com1.z0.glb.clouddn.com/sort_merge_animate.gif)

<!--more-->

## 递归法

原理如下（假设序列共有n个元素）：
 1. 将序列每相邻两个数字进行归并操作，形成floor(n/2)个序列，排序后每个序列包含两个元素
 2. 将上述序列再次归并，形成floor(n/4)个序列，每个序列包含四个元素
 3. 重复步骤2，直到所有元素排序完毕

## 迭代法

 1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
 2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
 3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
 4. 重复步骤3直到某一指针到达序列尾
 5. 将另一序列剩下的所有元素直接复制到合并序列尾
 
## 代码实现（java）

### 递归版

``` java
/** 归并排序，递归版
 *
 * @param: <E>
 * @param: m
 * @return: List<E>
 * @throws
 */
public static <E extends Comparable<? super E>> List<E> mergeSort(List<E> m) {
	if (m.size() <= 1)
		return m;

	int middle = m.size() / 2;
	List<E> left = m.subList(0, middle);
	List<E> right = m.subList(middle, m.size());

	right = mergeSort(right);
	left = mergeSort(left);
	List<E> result = merge(left, right);

	return result;
}

public static <E extends Comparable<? super E>> List<E> merge(List<E> left, List<E> right) {
	List<E> result = new ArrayList<E>();
	Iterator<E> it1 = left.iterator();
	Iterator<E> it2 = right.iterator();

	E x = it1.next();
	E y = it2.next();
	while (true) {
		// change the direction of this comparison to change the direction
		// of the sort
		if (x.compareTo(y) <= 0) {
			result.add(x);
			if (it1.hasNext()) {
				x = it1.next();
			} else {
				result.add(y);
				while (it2.hasNext()) {
					result.add(it2.next());
				}
				break;
			}
		} else {
			result.add(y);
			if (it2.hasNext()) {
				y = it2.next();
			} else {
				result.add(x);
				while (it1.hasNext()) {
					result.add(it1.next());
				}
				break;
			}
		}
	}
	return result;
}
```

### 迭代版

``` java
/** 归并排序，迭代版
 *
 * @param: nums
 * @return: void
 * @throws
 */
public static void mergeSort(int[] nums) {
	int len = nums.length;
	int[] result = new int[len];
	int block, start;

	for (block = 1; block < len*2; block *= 2) {
		for (start = 0; start < len; start += 2 * block) {
			int low = start;
			int mid = (start + block) < len ? (start + block) : len;
			int high = (start + 2 * block) < len ? (start + 2 * block)
					: len;
			// 两个块的起始下标及结束下标
			int start1 = low, end1 = mid;
			int start2 = mid, end2 = high;
			// 开始对两个block进行归并排序
			while (start1 < end1 && start2 < end2) {
				result[low++] = nums[start1] < nums[start2] ? nums[start1++]
						: nums[start2++];
			}
			while (start1 < end1) {
				result[low++] = nums[start1++];
			}
			while (start2 < end2) {
				result[low++] = nums[start2++];
			}
		}
		int[] temp = nums;
		nums = result;
		result = temp;
	}
	result = nums;
}
```

## 算法复杂度分析

比较操作的次数介于(n\*logn)/2(n log n)/2和n\*logn?n+1。 赋值操作的次数是 n\*logn。归并算法的空间复杂度为：O(n)。

## 参考文章

- [归并排序](https://zh.wikipedia.org/wiki/归并排序)
