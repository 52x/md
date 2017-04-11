title: leetcode-Best Time to Buy and Sell Stock 系列
date: 2015-02-23 10:29:11
categories: Leetcode
tags: [DP,贪心]
---
LeetCode OJ上面的炒股票系列，截止2015年2月，共收录四题。每题之间有相似性，现总结如下：

<!-- more -->

# [Best Time to Buy and Sell Stock](https://oj.leetcode.com/problems/best-time-to-buy-and-sell-stock/)
## 题意：
给出每天的股票价格，允许买入卖出恰好一次，求最大收益。
## 分析：
时间复杂度O(N),扫一遍，维护最低价格，最优解是（当天价格-之前最低价格）的最大值。
{% codeblock lang:cpp %}
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int minPrice = INT_MAX;
        int ret = 0;
        for(int i=0; i < prices.size(); ++i){
            minPrice = min(minPrice,prices[i]);
            ret = max(ret,prices[i]-minPrice);
        }
        return ret;
    }
};
{% endcodeblock %}

# [Best Time to Buy and Sell Stock II](https://oj.leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)
## 题意：
与上题的差异是可以无限次的买卖，但是再次买入前，必须要将之前持有的股票卖掉。
## 分析：
贪心解法，时间复杂度O(N),实质是求所有上升线段之和
{% codeblock lang:cpp %}
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int ret = 0;
        for(int i=1; i < prices.size(); ++i)
            if(prices[i]-prices[i-1]>0) ret += (prices[i]-prices[i-1]);
        return ret;
    }
};
{% endcodeblock %}

# [Best Time to Buy and Sell Stock III](https://oj.leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)
## 题意：
与上题的差异是恰好只能买卖2次，并且两次交易不能重叠。
## 分析：
时间复杂度O(N^2)的算法可以借用第一题，枚举分割点，分割点左右各调用一次第一题的函数即可。
时间复杂度O(N)DP思想；
l[i]:第0天至第i天，恰好只交易一次的最大收益，l[i]可由l[i-1]推出
r[i]:第i天至第n-1天，恰好只交易一次的最大收益，r[i]可由r[i+1]推出
然后O(N)枚举
{% codeblock lang:cpp %}
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int n = (int)prices.size();
        if(n==0) return 0;

        vector<int>l(n+1),r(n+1);
        int minPrice = prices[0];
        l[0] = 0;
        for(int i=1; i<n; ++i){
            minPrice = min(minPrice,prices[i]);
            l[i] = max(l[i-1],prices[i]-minPrice);
        }

        int maxPrice = prices[n-1];
        r[0] = 0;
        for(int i=n-2; i>=0; --i){
            maxPrice = max(maxPrice,prices[i]);
            r[i] = max(r[i+1],maxPrice-prices[i]);
        }

        int ret = l[n-1];
        for(int i=1; i<n; ++i)
            ret = max(ret, l[i-1]+r[i]);
        return ret;
    }
};
{% endcodeblock %}

# [Best Time to Buy and Sell Stock IV](https://oj.leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)
## 题意：
本题限制恰好买卖k次，并且交易之间不能有重叠。
## 分析：
当k >= N/2，实质上可以买卖无限多次，同第二题。
当k < N/2时，需要DP
设原始数据存于prices[1...n]数组中
f[i][k]表示用prices中的[1..i]天数据，交易k次的最大收益。
转移方程为：
f[i][k] = max(f[i][k-1],max(f[i-1][k],f[j-1][k-1] + prices[i] - prices[j]))
移项后得到：
f[i][k] = max(f[i][k-1],max(f[i-1][k],prices[i] + (f[j-1][k-1] - prices[j]) ))
(f[j-1][k-1]-prices[j])用变量max_cur代替，max_cur可以边计算边维护，所有时间算法复杂度为O(kN).
滚动数组节省空间，空间复杂度为O(N).
{% codeblock lang:cpp %}
class Solution {
public:
    int allSell(vector<int> &prices) {
        int ret = 0;
        for(int i=1; i < prices.size(); ++i)
            if(prices[i]-prices[i-1]>0) ret += (prices[i]-prices[i-1]);
        return ret;
    }
    int maxProfit(int k, vector<int> &prices) {
        int n = prices.size();
        if(k>=n/2){
            return allSell(prices);
        }

        prices.push_back(0);
        for(int i=n; i>=1; i--) prices[i] = prices[i-1]; //make sure prices's index start from 1

        int f[n+1][2];
        memset(f,0,sizeof(f));

        int now = 0;
        for(int j=1; j<=k; ++j){
            int max_cur = INT_MIN;
            f[0][now] = 0;
            for(int i=1; i<=n; ++i){
                f[i][now] = max(f[i][1-now],max(f[i-1][now],prices[i]+max_cur));
                max_cur = max(max_cur,f[i-1][1-now]-prices[i]);
            }
            now = 1-now;
        }
        return f[n][1-now];
    }
};
{% endcodeblock %}