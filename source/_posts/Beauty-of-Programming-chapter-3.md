title: 《编程之美读书笔记——第三章结构之法》
date: 2015-10-06 22:33:29
categories:
tags:
---
# 3.1 字符旋转包含的问题
给定两个字符串s1和s2，要求判断s2是否可以透过s1作旋转而获得。
s1作旋转所得到的字符都是s1s1的子串，如果s2可以透过s1作旋转而获得，那么s2一定是s1s1的子串。
<!-- more -->

# 3.2 电话号码对应英语单字
简单的回溯搜索

# 3.3 计算字符串的相似度
求两个字符串的编辑距离，可以修改一个字元；增加一个字元；删除一个字元。
动态规划，六种操作，整理之后是三种转移。
如果s1[i]!=s2[j]
f[i][j] = min(f[i+1][j],f[i][j+1],f[i+1][j+1])+1;

# 3.4 从无头单向Linked List中删除节点
狸猫换太子，将next节点信息拷贝到本节点，之后删除next节点。

# 3.5 最短摘要生成*
暂时没看到它要干嘛

# 3.6 写程式判断两个Linked List是否相交
两个无循环单向Linked List判断是否相交。
解法一：由于无循环，将第一个Linked List接在第一个后面，转换为判断Linked List是否有循环。

解法二：考虑两个Linked List的相交情况一定是类似与Y这种形态，那么尾节点必然一样，判断两个Linked List最后一个节点是否相同即可。

# 3.7 队列中取最大值操作问题*
设计一个队列，满足以下三个操作：
1.EnQueue(v)：将v加入队列中
2.DeQueue：将队列中的首元素删除并传出此元素
3.MaxElement：取出队列中的最大元素

解法一：构建大根堆，用链表维护进队序列。出入队复杂度O(logN),取最大值复杂度O(1)。
解法二：分解为两个子问题。
1.支持取最大值操作的栈
2.用栈模拟队
子问题1可以用两个栈来实现，每个操作复杂度O(1)
子问题2可以用两个栈来模拟堆，插入一律放到B堆，弹出时如果A为空，将B中元素pop到A中，再从A中弹出。每个操作复杂度O(1)。
总时间复杂度O(1)

# 3.8 求二元树中节点的最大距离*
树的直径问题，经典作法是两遍BFS(DFS)。对于二元树也可以用DP，一遍dfs即可。

# 3.9 重建二元树*
前序、中序、后序三种遍历顺序，已知其中两个，求另一个并构建出二元树。
如果是二叉搜索树（BST），有没有更快地方法？

# 3.10 分层走遍二元树
DFS打印每层level会有重复调用，直接BFS即可。

# 3.11 程式改错
给段二分查找代码，找错。
m = (l+r)/2，假设是32位元程式，32位int的范围是-2147483648到+2147483647,如果l+r恰好超过+2147483647，就会向上溢出，比较保险的写法是 m = l + (r-l)/2
```cpp
while(l<=r){
	if(check()) l++;
	else r--;
}
```


