title: Queue With MAX
date: 2016-06-02 20:03:53
categories: Algorithms
tags: [Stack, Queue]
---
实现一个带有max操作的队列

<!--more-->
队列的前后两端数据都在变化，直接添加max操作，很难维护最大值。用降维的思想来考虑，固定一端不变，另一端进出数据（其实就是栈stack）。stack with max是一个很经典的问题，具体解法是利用两个栈来维护，A栈负责push，B栈负责记录可以update最大值得元素。代码如下：
```cpp
class MaxStack {
    stack<int> S, M;
    
public:
    void push(int x) {
        S.push(x);
        if (M.empty() || M.top() <= x) M.push(x);
    }
    
    void pop() {
        if (S.top() == M.top()) M.pop();
        S.pop();
    }
    
    int top() { return S.top(); }
    
    int getMax() { return M.top(); }
};
```

如何将stack with max转成queue with max？显而易见，用stack实现queue的操作即可，思路还是两个栈，A栈负责push数据，pop的时候优先从B栈弹出元素，B栈为空时，将A栈的元素全部弹入置B栈。每个从B栈弹出的元素，都曾经从A栈弹出，负负得正，两次栈的操作保证了队列的有序性。pop的均摊复杂度O(1)。
```cpp
class Queue {
public:
    // Push element x to the back of queue.
    stack<int> in, out;
    void push(int x) { in.push(x); }
    
    // Removes the element from in front of queue.
    void pop(void) {
        if (!out.empty())
            out.pop();
        else {
            while (!in.empty()) {
                out.push(in.top());
                in.pop();
            }
            out.pop();
        }
    }
    
    // Get the front element.
    int peek(void) {
        if (!out.empty())
            return out.top();
        else {
            while (!in.empty()) {
                out.push(in.top());
                in.pop();
            }
            return out.top();
        }
    }
    
    // Return whether the queue is empty.
    bool empty(void) { return in.empty() && out.empty(); }
};
```
将以上两份代码略加综合，就是Queue With MAX的完整解法。
