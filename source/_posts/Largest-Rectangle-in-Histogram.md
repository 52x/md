title: Largest Rectangle in Histogram
categories: Algorithms
date: 2016-06-10 20:28:45
tags: [Stack, LeetCode]
---

一道Leetcode原题，[Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)
给定n个非负整数，代表代表柱状图的高度（每个柱体宽度为1），找到柱状图中面积最大的矩形。

![摘自GeeksForGeeks](http://d1gjlxt8vb0knt.cloudfront.net//wp-content/uploads/histogram1.png)

<!--more-->
这里有分治+RMQ的解法[GeeksForGeeks分治解法](http://www.geeksforgeeks.org/largest-rectangular-area-in-a-histogram-set-1/)，时间复杂度为O(NlogN)，下面介绍一种O(N)的解法。
# 算法思路
对于一个bar x，我们计算以x为最低点的面积最大的矩形，把全部bar都计算一遍，就可以找到柱状图中面积最大的矩形了。关键是如何找到以x为最低点的矩形？我们需要知道x左右两侧，第一个比x低的bar的位置，分别记做left index和right index。
从左至右遍历所有的bar，用栈来维护这些bar。保证栈中的bar是递增的，即新遇到的bar比栈顶元素低的时候，就弹出栈顶元素。当一个bar弹出栈的时候，这个bar就是x，新遇到的bar的坐标就是right index，栈中上一个元素坐标就是left index，具体代码如下：

# 详细解法
1. 创建一个空栈
2. 从第一个bar开始扫描
   a)如果栈空或者height[i] >= 栈顶元素高度，则将i压入栈中
   b)如果height[i] < 栈顶高度，不断弹出栈顶元素，直到height[i] >= 栈顶元素高度或栈空。令弹出栈的的元素高度是height[tp],计算以height[tp]为最低点的矩形面积。对于height[tp]这个bar，left index是栈中前一个元素，right index是i。由于每个Bar入栈出栈各一次，因此总时间复杂度为O(N)。
   
`具体实现时，再尾部加入height = 0的bar，确保之前所有bar都能出栈`

代码如下：
```cpp
int largestRectangleArea(vector<int> &height) {
   int res = 0;
   stack<int>index;
   height.push_back(0);
   
   int i = 0, n = (int)height.size();
   while (i < n) {
       while (!index.empty() && height[index.top()] >= height[i]) {
           int h = height[index.top()];
           index.pop();
           int idx = index.empty() ? -1 : index.top();
           res = max(res, h * (i - idx - 1));
       }
       index.push(i++);
   }
   return res;
}
```

# 进阶
LeetCode上面还有一道此题的进阶版，[Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)，题意为：给定一个01矩阵，找出面积最大的全1子矩阵。
降维之后其实就是Largest Rectangle in Histogram，假设矩阵有N行M列，则做了N次Largest Rectangle in Histogram，因此复杂度为O(N*M)，代码如下：
```cpp
int getRectangle(vector<int>& height) {
    int n = height.size();
    int res = 0, i = 0;
    stack<int> index;
    while (i < n) {
        while (!index.empty() && height[index.top()] >= height[i]) {
            int h = height[index.top()];
            index.pop();
            int idx = index.empty() ? -1 : index.top();
            res = max(res, (i - idx - 1) * h);
        }
        index.push(i++);
    }
    return res;
}
int maximalRectangle(vector<vector<char>>& matrix) {
    int row = matrix.size();
    if (row == 0) return 0;
    int n = matrix[0].size();
    vector<int> height;
    for (int i = 0; i < n; ++i) height.push_back(matrix[0][i] - '0');
    height.push_back(0);
    int res = 0;
    for (int r = 1; r < row; ++r) {
        res = max(res, getRectangle(height));
        for (int i = 0; i < n; ++i) {
            if (matrix[r][i] == '1')
                height[i]++;
            else
                height[i] = 0;
        }
    }
    res = max(res, getRectangle(height));
    return res;
}
```

类似的题目还有：[The Stock Span Problem](http://www.geeksforgeeks.org/the-stock-span-problem/)

