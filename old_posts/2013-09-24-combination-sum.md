---
layout: post 
title: LeetCode Combination Sum
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

>Given a set of candidate numbers (C) and a target number (T), find all unique combinations in C where the candidate numbers sums to T.

>The same repeated number may be chosen from C unlimited number of times.

>Note:
>
>    All numbers (including target) will be positive integers.
>    Elements in a combination (a1, a2, … , ak) must be in non-descending order. (ie, a1 ≤ a2 ≤ … ≤ ak).
>    The solution set must not contain duplicate combinations.
>
>For example, given candidate set 2,3,6,7 and target 7,
>A solution set is:
>[7]
>[2, 2, 3] 

思路:

和全排列,八皇后,四色问题,还原IP地址的思路一样,都是使用回溯法,或者说深搜+剪枝.

题目要求组合中的数字以非降序排列,所以首先要对C进行排序.

代码:

    vector<vector<int> > combinationSum(vector<int> &candidates, int target) {
        vector<vector<int> > result;
        if( target <= 0 ){
            return result;
        }

        sort( candidates.begin(), candidates.end() );

        vector<int> ele;
        combinationSum( candidates, target, result, ele, 0, 0  );
        return result;
    }
   
    /*
     *tmp 保存当前ele中的元素和
     *index 表示当前已经搜索到的层次,与candidates中元素下标相对应
     *每次从index开始遍历,而不是从零开始,可以保证加入到ele的元素不会程序降序的情况
     */ 
    void combinationSum( vector<int>& candidates, int target, vector<vector<int>>& result, vector<int> ele, int tmp, int index ){
         if( tmp == target){
	     result.push_back( ele );
             return;
         }
        
        for( int i = index; i < candidates.size(); ++i ){
           if( tmp + candidates[i] <= target ){//注意,不要漏掉相等时的情况
               ele.push_back( candidates[i] );
               tmp += candidates[i];
               combinationSum( candidates, target, result, ele, tmp, i );
               
               ele.erase(ele.end()-1 );//恢复ele和tmp,相当于回溯
               tmp -= candidates[i];
           }
        }
    }
