title: 只用O(1)额外空间的Min Stack
categories: Algorithms
date: 2016-06-08 14:02:21
tags: [Stack]
---
`Min Stack`：支持`getMin()`操作的栈，前不久刚写过O(1)时间，O(N)额外空间的解法，并扩展到了Min Queue。详见[Queue With MAX](http://devhui.com/2016/06/02/Queue-With-MAX/)。

昨晚在GeeksforGeeks上面看到了一篇文章[Design a stack that supports getMin() in O(1) time and O(1) extra space](http://www.geeksforgeeks.org/design-a-stack-that-supports-getmin-in-o1-time-and-o1-extra-space/)，可以仅用O(1)时间、`O(1)额外空间`解决此问题。这个解法非常的genius，在此介绍这个方法。

<!--more-->

# 算法思路
定义变量minEle，代表当前栈中的最小元素。那么关键问题就是，如何在最小元素出栈后，维护minEle的值。为了维护minEle，入栈push(x)的时候，并不是将x入栈，而是将`2x - minEle`入栈，这样就可以根据当前的minEle获取到之前的minEle。下面是算法那的细节：

# 详细步骤
`Push(x)`:

如果栈空，则将x入栈，并将minEle = x
如果栈非空，则比较x和minEle
  （1）如果x >= minEle，则直接将x入栈
  （2）如果x < minEle,将`2*x - minEle`入栈，并将minEle = x，例如之前的minEle = 3，现在push(2)，则将minEle更新为2，并讲2*2 - 3 = 1入栈。
  
`Pop()`:
将栈顶元素出栈，令出栈的元素为y
如果y >= minEle，则最小元素依旧在栈中，minEle不变
如果y <= minEle,则最小元素变为2*minEle - y，所以将minEle = 2 * minEle - y。在这里我们通过当前的minEle，得到了之前的minEle。

`注意`
如果当前值是栈中最小值，那么栈中存储的并不是它的真实值
minEle永远是栈中最小值的真实值。

# 实例

## Push(x)
![Push(x)](http://d1gjlxt8vb0knt.cloudfront.net//wp-content/uploads/stack_insert.png)
1. Number to be Inserted: 3, Stack is empty, so insert 3 into stack and minEle = 3.
2. Number to be Inserted: 5, Stack is not empty, 5> minEle, insert 5 into stack and minEle = 3.
3. Number to be Inserted: 2, Stack is not empty, 2< minEle, insert (2 * 2-3 = 1) into stack and minEle = 2.
4. Number to be Inserted: 1, Stack is not empty, 1< minEle, insert (2 * 1-2 = 0) into stack and minEle = 1.
5. Number to be Inserted: 1, Stack is not empty, 1 = minEle, insert 1 into stack and minEle = 1.
6. Number to be Inserted: -1, Stack is not empty, -1 < minEle, insert (2 * -1 – 1 = -3) into stack and minEle = -1.

## Pop()
![Pop()](http://d1gjlxt8vb0knt.cloudfront.net//wp-content/uploads/stack_removal.png)
1. Initially the minimum element minEle in the stack is -1.
2. Number removed: -3, Since -3 is less than the minimum element the original number being removed is minEle which is -1, and the new minEle = 2 * -1 – (-3) = 1
3. Number removed: 1, 1 == minEle, so number removed is 1 and minEle is still equal to 1.
4. Number removed: 0, 0< minEle, original number is minEle which is 1 and new minEle = 2 * 1 – 0 = 2.
5. Number removed: 1, 1< minEle, original number is minEle which is 2 and new minEle = 2 * 2 – 1 = 3.
6. Number removed: 5, 5> minEle, original number is 5 and minEle is still 3

# 源码
源码摘自GeeksforGeeks

```cpp
// C++ program to implement a stack that supports
// getMinimum() in O(1) time and O(1) extra space.
#include <bits/stdc++.h>
using namespace std;
 
// A user defined stack that supports getMin() in
// addition to push() and pop()
struct MyStack
{
    stack<int> s;
    int minEle;
 
    // Prints minimum element of MyStack
    void getMin()
    {
        if (s.empty())
            cout << "Stack is empty\n";
 
        // variable minEle stores the minimum element
        // in the stack.
        else
            cout <<"Minimum Element in the stack is: "
                 << minEle << "\n";
    }
 
    // Prints top element of MyStack
    void peek()
    {
        if (s.empty())
        {
            cout << "Stack is empty ";
            return;
        }
 
        int t = s.top(); // Top element.
 
        cout << "Top Most Element is: ";
 
        // If t < minEle means minEle stores
        // value of t.
        (t < minEle)? cout << minEle: cout << t;
    }
 
    // Remove the top element from MyStack
    void pop()
    {
        if (s.empty())
        {
            cout << "Stack is empty\n";
            return;
        }
 
        cout << "Top Most Element Removed: ";
        int t = s.top();
        s.pop();
 
        // Minimum will change as the minimum element
        // of the stack is being removed.
        if (t < minEle)
        {
            cout << minEle << "\n";
            minEle = 2*minEle - t;
        }
 
        else
            cout << t << "\n";
    }
 
    // Removes top element from MyStack
    void push(int x)
    {
        // Insert new number into the stack
        if (s.empty())
        {
            minEle = x;
            s.push(x);
            cout <<  "Number Inserted: " << x << "\n";
            return;
        }
 
        // If new number is less than minEle
        if (x < minEle)
        {
            s.push(2*x - minEle);
            minEle = x;
        }
 
        else
           s.push(x);
 
        cout <<  "Number Inserted: " << x << "\n";
    }
};
```

# 为什么这个方法是正确的？
当入栈元素x小于minEle，我们是将`2x - minEle`入栈，重点就在这里，易证2x - minEle必定小于x。当出栈的时候，如果发现出栈元素y小于当前的minEle时，就用2 * minEle - y维护minEle。
```
How previous minimum element, prevMinEle is, 2 * minEle - y
in pop() is y the popped element?

 // We pushed y as 2x - prevMinEle. Here 
 // prevMinEle is minEle before y was inserted
 y = 2*x - prevMinEle  

 // Value of minEle was made equal to x
 minEle = x .
	
 new minEle = 2 * minEle - y 
            = 2*x - (2*x - prevMinEle)
            = prevMinEle // This is what we wanted
```