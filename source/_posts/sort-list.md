title: 单链表排序 Sort List
date: 2015-02-23 11:45:31
categories: Leetcode
tags: [排序,链表]
---
LeetCode OJ中的[Sort List](https://oj.leetcode.com/problems/sort-list/)要求使用O(nlogn)的时间复杂度，常数空间对单链表排序。
<!-- more -->
之前面试也遇到过实现链表快排，本文介绍下如何在链表中应用几种常见的排序算法。以快速排序和归并排序为例。
在链表中快速排序和归并排序的均摊复杂度都是O(nlogn)，空间复杂度也都是常数。但是对于某些数据，快排的最坏时间复杂度会退化至O(N^2)。

## 快速排序 Quick Sort
快排的核心是partition函数，选择某个元素作为支点，O(N)扫一遍，使数组中比支点小的元素都在支点左边，比支点大的元素都在支点右边。

对于数组的快排代码如下：
{% codeblock lang:cpp%}
void quick_sort(int s[], int l, int r) {
    if (l < r) {
        //Swap(s[l], s[(l + r) / 2]); //将中间的这个数和第一个数交换 参见注1
        int i = l, j = r, x = s[l];
        while (i < j) {
            while(i < j && s[j] >= x) // 从右向左找第一个小于x的数
                j--;
            if(i < j)
                s[i++] = s[j];

            while(i < j && s[i] < x) // 从左向右找第一个大于等于x的数
                i++;
            if(i < j)
                s[j--] = s[i];
        }
        s[i] = x;
        quick_sort(s, l, i - 1); // 递归调用
        quick_sort(s, i + 1, r);
    }
}
{% endcodeblock %}

如果是单链表排序，由于只有next指针，将不能像数组从后往前遍历元素，因此上面的实现不再适用于单链表。但是核心思想不便，链表的优势是可以O(1)的向链表后边插入节点。因此partition的时候，新建两个指针，*lp保存比支点小的节点，*rp保存比支点大的节点。O(N)遍历链表，节点比支点小时插入*lp链表，反之插入*rp链表。遍历结束后，按照lp->支点->rp的顺序，将链表进行组装，即可完成partition需要的操作,时间复杂度也是O(N)。

leetcode的测试数据，有一组[2,2,3,3,1,3,2,1...]的数据，只有1-3三个整数，出现65536次。快排面对这组数据，复杂度会降为O(N^2),导致TLE。使用了剪枝技巧，当子数组中的元素全部相等，即最大值等于最小值，就不继续递归。
完整代码如下：
{% codeblock lang:cpp%}
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *sortList(ListNode *head) {
        if(head==NULL) return head;
        ListNode *lRoot = new ListNode(-1);
        ListNode *rRoot = new ListNode(-1);
        ListNode *piot = head;

        ListNode *lp = lRoot,*rp = rRoot;
        ListNode *lastLeft = NULL;
        int minL=INT_MAX,minR=INT_MAX,maxL=INT_MIN,maxR=INT_MIN;
        head = head->next;
        while(head!=NULL){
            if(head->val <= piot->val){
                minL = min(minL,head->val);
                maxL = max(maxL,head->val);
                lp->next = head;
                lp = lp->next;
                head = head->next;
                lp->next = NULL;
            }
            else{
                minR = min(minR,head->val);
                maxR = max(maxR,head->val);
                rp->next = head;
                rp = rp->next;
                head = head->next;
                rp->next = NULL;
            }
        }
        if(minL==maxL)
            lp = lRoot->next;
        else
            lp = sortList(lRoot->next);

        if(minR==maxR)
            rp = rRoot->next;
        else
            rp = sortList(rRoot->next);

        lastLeft = lp;
        while(lastLeft!=NULL && lastLeft->next!=NULL) lastLeft = lastLeft->next;

        //merge
        if(lp==NULL){
            piot->next = rp;
            return piot;
        }
        else{
            lastLeft->next = piot;
            piot->next = rp;
            return lp;
        }
    }
};
{% endcodeblock %}

## 归并排序 Merge Sort
归并排序使用分治思想，单链表实现与数组极其类似。MergeList()函数将两个有序链表合并为一个有序链表，具体代码如下：
{% codeblock lang:cpp%}
class Solution {
public:
    ListNode *mergeList(ListNode *l1, ListNode *l2){
        ListNode *vRoot = new ListNode(-1), *cur = vRoot;
        while(l1!=NULL && l2!=NULL){
            if(l1->val<=l2->val){
                cur->next = l1;
                l1 = l1->next;
            }
            else{
                cur->next = l2;
                l2 = l2->next;
            }
            cur = cur->next;
        }
        while(l1!=NULL){
            cur->next = l1;
            l1 = l1->next;
            cur = cur->next;
        }
        while(l2!=NULL){
            cur->next = l2;
            l2 = l2->next;
            cur = cur->next;
        }
        cur->next = NULL;

        cur = vRoot->next;
        return cur;
    }
    ListNode *sortList(ListNode *head) {
        if(head==NULL || head->next==NULL) return head;

        int cnt = 0;
        ListNode *mid,*cur,*l1,*l2;
        cur = head;
        while(cur!=NULL){
            cnt++;
            cur = cur->next;
        }
        mid = head;
        for(int i=1; i<cnt/2; ++i) mid = mid->next;

        cur = mid; mid = mid->next; cur->next = NULL;
        l1 = sortList(head);
        l2 = sortList(mid);
        head = mergeList(l1,l2);
        return head;
    }
};
{% endcodeblock %}