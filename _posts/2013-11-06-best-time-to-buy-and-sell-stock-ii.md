---
layout: post 
title: LeetCode Best Time to Buy and Sell Stock II
categories:
- Online Judge
tags:
- LeetCode
- interview
---

>Say you have an array for which the ith element is the price of a given stock on day i.
>
>Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).
>

思路:

其实,这道题要分析出来一个规律,只要第i天的价格大于(i-1)天的,那么就可以看做最大收益的一部分.
如果认识到这个规律,问题就很简单了.就怕没看出来啊.

代码:

    int maxProfit(vector<int> &prices) {
        // IMPORTANT: Please reset any member data you declared, as
        // the same Solution instance will be reused for each test case.
        
        int maxPro = 0;
        for( int i = 1; i < prices.size(); ++i ){
            if( prices[i] > prices[i-1] )
                maxPro += prices[i] - prices[i-1];
        }
        
        return maxPro;
    }
