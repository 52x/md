title: 判断序列是否为BST的先序遍历
categories: Algorithms
date: 2016-06-10 22:57:52
tags: [Stack, BST]
---
给定一个整数序列，判断这个序列是否可以构成某颗搜索二叉树（BST）的前序遍历结果。朴素的做法是O(N^2)，2014年面试的时候遇到过此题，当时只给出了O（NlogN）的解法，现在给出O(N)解法。

<!--more-->

根据BST和前序遍历性质，可以直接递归判断，该算法的最坏时间复杂度为O(N^2)，加入二分的思想，维护区间max和min可以优化到O(NlogN)。


此题还有O(N)解法，主要思想是使用栈，思路类似[Next Greater Element](http://www.geeksforgeeks.org/next-greater-element/)，也是寻找Next Greater Elemen，并维护下界low，如果之后找到小于下界low的值，则返回false。

算法流程如下：
1. 创建空栈
2. 初始化根的下界low = INI_MIN
3. 对于数组中每个数pre[i]
	a）如果pre[i] 小于当前下界low，return false
	b)如果pre[i] > 栈顶元素则持续弹出栈顶元素，最后一个弹出的为当前子树的根，因此将low更新为pre[i]。
	c)将pre[i]进栈（栈中元素保持递减性质）
	
具体代码如下:
```cpp
bool verifyPreorder(vector<int>& preorder) {
    int low = INT_MIN;
    stack<int> path;
    for (auto& p : preorder) {
        if (p < low) {
            return false;
        }
        while (!path.empty() && p > path.top()) {
            // Traverse to its right subtree now.
            // Use the popped values as a lower bound because
            // we shouldn't come across a smaller number anymore.
            low = path.top();
            path.pop();
        }
        path.emplace(p);
    }
    return true;
}
```
此方法的空间、时间复杂度均为O(N)。还有一种不使用额外空间的改进方法，将原数组当做栈。
```cpp
bool verifyPreorder(vector<int>& preorder) {
    int low = INT_MIN, i = -1;
    for (auto& p : preorder) {
        if (p < low) {
            return false;
        }
        while (i >= 0 && p > preorder[i]) {
            low = preorder[i--];
        }
        preorder[++i] = p;
    }
    return true;
}
```