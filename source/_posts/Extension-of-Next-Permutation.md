title: Next Permutation的扩展
date: 2016-06-08 10:37:07
categories: Algorithms
tags: [Math, Greedy]
---

写一个函数生成满足下面三个条件的整数1. 非负2. 不能有重复数字3. 递增，既后面产生的比前面产生的要大
<!--more-->
# 解法一、递归生成全部数据
一个类似排列数的递归函数即可实现
# 解法二、nextNum
实现nextNum()函数，对于给定一个符合条件的数字，如318，返回下一个（比输入大但是最小的）数字，319。如果能高效的实现这个函数，不断调用，就可递增的生成满足条件的整数。
首先考虑一种特殊情况，当输入数字是10位数，那么0-9每个数字恰好出现一次，此时的nextNum()就是next_permutation。简单复习下next_permutation的算法：
## 2.1 next_permutation```cppvoid nextPermutation(vector<int>& nums) {
    int n = nums.size();
    int p = -1;

    //from end,find first increase,nums[p]<nums[p+1]
    for (int i = n - 2; i >= 0; --i) {
        if (nums[i] < nums[i+1]) {
            p = i;
            break;
        }
    }
    if (p == -1) { // all increase
        sort(nums.begin(),nums.end());
        return;
    }

    //find the min value at nums[pos] which bigger than nums[p]
    //have the same value,get the rightest
    int value = nums[p+1], pos = p+1;
    for (int i = p + 2; i < n; ++i){
        if (nums[i] > nums[p] && nums[i] <= value) {
            value = nums[i];
            pos = i;
        }
    }

    swap(nums[p], nums[pos]);

    //reverse [p+1,end)
    int l = p + 1, r = n - 1;
    while (l <= r) {
        swap(nums[l], nums[r]);
        l++; r--;
    }
}```
算法大概分为三步1. 从后向前，找到第一个递增的位置p。无递增则无下一个排列数，例如54321。
2. 找到该p后，大于等于nums[p]的最小值，交换这两个数
3. 将p+1之后的元素翻转
例如，5 2 4 3 1
第一个递增位置是 2 4中的2
2之后，大于等于2的最小值是3，交换2 3变为5 3 4 2 1
翻转3之后的元素为，5 3 1 2 4

## 2.2 next_permutation的扩展
回到nextNum问题找几个例子就会看出一定的规律，`括号中是未出现的数字`

68543192 (07)
68543197 (02)
一个贪心策略是在未出现的数字中挑选出x来替换末位数字，x > 末位数字

但是如果未使用数字均小于末位数字呢？例如
38219607 （45）
38219640 （57）
那么可以稍稍扩展一下，当无法替换末位时，那就逐位向前比较，直到找比未使用数字小的那位。38219607倒数第二位0比未使用的4小，于是用4替换0，之后将未使用（0，5）由小到大的填入后面。

换组数据检测下当前贪心策略
2341987 （056）
2345016 （789）
2341987中的倒数第四位的1才小于未使用数字（056），因此用未使用的5替换1，变为2345xxx(016789)，将未使用的0 1 6 7 8 9递增填入后面的空位得到2345016，貌似是对的。

但是！
1573698（024）按照上面算法计算的结果是1574026，然而正确答案是1573802！
之前算法替换的是倒数第四位的3，而正确答案替换的是倒数第三位的6。为什么6比（024）小却可以替换呢？其实应该将比较失败的末尾数字加入到未使用集合，如此当比较到6的时候，未使用序列就是（02489），显然8比6大，因此用8替换6。

那么整理下目前的贪心策略
1.将未出现的数字加入候选集合R
2.从末尾向前扫描，
如果x > R中的全部数字，则将x加入R，继续向前扫描
如果x < R中的某个数字，则从R中选取恰当数字替换x（大于x的最小值）
3.将R中的元素，递增的填入x之后的空位

## 2.3 nextNum与next_permutation比较
这个算法本质是next_permutation的一般化，当R初始为空集时，就是next_permutation。第二步其实就对应next_permutation的寻找第一个递增位置，并swap元素；第三步对应next_permutation的reverse，因为next_permutation可以保证swap之后的元素是递减的，因此可以直接reverse，这里需要将R中元素排序后依次填入。

`注：需要考虑进位的边界情况, 9876->10234`

getNext()代码如下:
```cpp
int getNext(int n) {
    vector<int> num, R; //num存储当前，R存储未使用数字
    bool h[10] = {0};
    while (n) {
        num.push_back(n % 10);
        h[n % 10] = true;
        n /= 10;
    }
    
    //将未使用的数字加入R集合，R集合有序递增
    for (int i = 0; i < 10; ++i)
        if (h[i] == false) R.push_back(i);
    
    //由于R有序，只需比较R中最大的R.back()
    int p = 0;
    while (R.empty() || (p < num.size() && num[p] > R.back())) {
        //当前位的数字num[p]大于R中所有元素，则将num[p]加入R集合
        R.push_back(num[p++]);
    }
    
    if (p == num.size()) {
        //需要进位 9876->10234
        int res = 10;
        for (int i = 2; i <= num.size(); ++i) {
            res = res * 10 + i;
        }
        return res;
    }
    
    //在R中寻找最小的>=num[p]的位置pos
    auto pos = lower_bound(R.begin(), R.end(), num[p]);
    swap(*pos, num[p]);
    sort(R.begin(), R.end()); //swap后R无序，调整为有序
    //将R中元素按顺序加入空位
    int k = 0;
    for (int i = p - 1; i >= 0 && k < R.size(); --i) {
        num[i] = R[k++];
    }
    
    int res = 0;
    for (int i = (int)num.size() - 1; i >= 0; --i) {
        res = res * 10 + num[i];
    }
    return res;
}
```