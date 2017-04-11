title: 非递归遍历二叉树
date: 2016-06-02 16:16:37
categories: Algorithms
tags: [Stack, Binary Tree]

---

使用非递归的方法实现二叉树的中序、前序序、后序遍历
<!-- more -->

用栈（stack）模拟递归是很自然的想法
![二叉树](/images/BinaryTreeTraversal/binary_tree.jpg)

# 中序遍历
上图的中序遍历为：ADEFGHMZ
```cpp
vector<int> inorderTraversal(TreeNode* root) {
  vector<int> res;
  stack<TreeNode*> S;
  TreeNode *cur = root;
  while(!S.empty() || cur) {
      if (cur) {
          S.push(cur);
          cur = cur->left;
      } else {
          TreeNode *temp = S.top();
          S.pop();
          res.push_back(temp->val);
          cur = temp->right;
      }
  }
  return res;
}
```
将cur节点入栈，用cur的左儿子代替cur向下迭代遍历。cur走到NULL节点时，从栈中取出之前保存的parent（temp）节点，将parent节点出栈，继续遍历parent的右子树。

# 前序遍历
上图的中序遍历为：GDAFEMHZ
前序遍历的思路与中序遍历完全相同，只是记录答案的时机略有区别。
```cpp
vector<int> preorderTraversal(TreeNode* root) {
    TreeNode* cur = root;
    stack<TreeNode*> S;
    vector<int> res;
    
    while (!S.empty() || cur) {
        if (cur) {
            res.push_back(cur->val);s
            S.push(cur);
            cur = cur->left;
        } else {
            TreeNode *temp = S.top();
            S.pop();
            cur = temp->right;
        }
    }
    return res;
}
```

# 后序遍历
上图的后序遍历为：AEFDHZMG
后序遍历与前两种遍历略有不同，难点在于遍历parent的右儿子之前，已经将parent节点出栈。
```cpp
TreeNode *temp = S.top();
S.pop();
cur = temp->right;
```
后序遍历的顺序是：左子树->右子树->根，但是当访问完右子树之后，已经不能得到parent节点。

`如何解决这个问题？之前的思路还可以用吗？`

我们从前序遍历入手，前序遍历的顺序是:根->左子树->右子树
如果以根->右子树->左子树的顺序遍历二叉树，会得到什么结果呢？ GMZHDFEA，恰好是后序遍历的逆序。因此可以对前序遍历稍作改造，以根->右->左的方式遍历二叉树，得到的结果就是后序遍历的reverse。代码如下:
```cpp
vector<int> postorderTraversal(TreeNode* root) {
    TreeNode* cur = root;
    stack<TreeNode*> S;
    vector<int> res;

    while (!S.empty() || cur) {
        if (cur) {
            res.push_back(cur->val);
            S.push(cur);
            cur = cur->right;
        }
        else {
            TreeNode *temp = S.top();
            S.pop();
            cur = temp->left;
        }
    }
    reverse(res.begin(), res.end());
    return res;
}
```
上面这种做法略微有些trick，不过代码思路与前序、中序保持了一致性。后序遍历还有很多其他解法，如下：
```cpp
vector<int> postorderTraversal(TreeNode* root) {
    TreeNode* pre = root;
    stack<TreeNode*> S;
    vector<int> res;
    
    if (root == NULL) return res;
    S.push(root);
    while (!S.empty()) {
        TreeNode *cur = S.top();
        if (cur->left == pre || cur->right == pre || (!cur->left && !cur->right)) {
            res.push_back(cur->val);
            pre = cur;
            S.pop();
        }
        else {
            if (cur->right)
                S.push(cur->right);
            if (cur->left)
                S.push(cur->left);
        }
    }
    return res;
}
```
该方法的思路是考查输出节点的必要条件
1.叶子节点
2.左右儿子已经被访问过
基于此，用pre记录上次输出的节点，如果`cur->left == pre || cur->right == pre`则说明cur节点的左右子树已经遍历完毕，可以输出cur节点。

# 扩展
Morri算法可以用O(1)的空间，O(N)的时间遍历二叉树。
[Morris Traversal方法遍历二叉树（非递归，不用栈，O(1)空间)](http://www.cnblogs.com/AnnieKim/archive/2013/06/15/morristraversal.html)


