---
layout: post 
title: LeetCode Next Permutation
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.

> If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).

> The replacement must be in-place, do not allocate extra memory.

> Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.

> 1,2,3 → 1,3,2

> 3,2,1 → 1,2,3

> 1,1,5 → 1,5,1

思路:

没有想出来做法,参考[这篇文章]( http://fisherlei.blogspot.com/2012/12/leetcode-next-permutation.html )完成了代码


代码:

    void nextPermutation(vector<int> &num) {
        if( num.size() < 2 )
            return;
            
        vector<int>::iterator i = num.end()-2;
        vector<int>::iterator j;// = i-1;
        
        while( i >= num.begin()  && *i >= *(i+1) ){
            --i;
        }
        
        if( i < num.begin() ){
             reverse( num.begin(), num.end() );
        }else{
            j = num.end() -1;
            while( j >= num.begin() ){
                if( *j > *i ){
                    int tmp = *i;
                    *i = *j;
                    *j = tmp;
                    reverse( i+1, num.end() );
                    break;
                }
                --j;
            }
        }
    }


