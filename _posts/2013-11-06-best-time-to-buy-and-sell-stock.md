---
layout: post 
title: LeetCode Best Time to Buy and Sell Stock
categories:
- Online Judge
tags:
- LeetCode
- interview
---

>Say you have an array for which the ith element is the price of a given stock on day i.
>
>If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.

思路:

其实只需要求出数组中差最大的两个数且较小者位于数组前面.

代码:

    int maxProfit(vector<int> &prices) {
        if( prices.size() < 2 )
            return 0;
        int minIndex = 0;
        int maxDiff = 0;
        int buy  = 0;
        int sell = 0;
        
        for( int i = 1; i < prices.size(); ++i ){
            if( prices[i] < prices[minIndex] ){
                minIndex = i;
            }
            
            if( maxDiff < ( prices[i] - prices[minIndex] ) ){
                maxDiff = prices[i] - prices[minIndex];
                buy = minIndex;
                sell = i;
            }
        }
        return maxDiff;
    }
