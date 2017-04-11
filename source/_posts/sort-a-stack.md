title: Sort a Stack
date: 2016-06-06 22:24:02
categories: Algorithms
tags: [Stack]
---
将一个栈中的数据排序，只能使用push pop top empty四种操作，并且不能手动申请额外的内存。

<!--more-->
用栈实现选择排序，关键的步骤是insert函数：向一个已经有序的栈插入元素e，并保持有序性。由于不能手动申请新内存，可以用系统栈的递归调用实现。
```cpp
template <typename T>

void insert(stack<T> &S, const T &e) {
	if (S.empty() || S.top() <= e) {
		S.push(e);
	} else {
		T f = S.top();
		S.pop();
		insert(S, e);
		S.push(f);
	}

}
```

排序函数也同样是递归实现，将e插入栈底已经排好的序列中。
```cpp
template <typename T>

void sort(stack<T> &S) {
	if (!S.empty()) {
		T e = S.top();
		S.pop();
		sort(S);
		insert(S, e);
	}
}
```
以上两个函数均使用了O(N)的系统栈内存，总时间复杂度为O(N^2)。
