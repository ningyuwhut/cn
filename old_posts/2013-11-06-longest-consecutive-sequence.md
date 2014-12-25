---
layout: post 
title: LeetCode Longest Consecutive Sequence
categories:
- Online Judge
tags:
- LeetCode
- interview
---

>Given an unsorted array of integers, find the length of the longest consecutive elements sequence.
>
>For example,
>Given [100, 4, 200, 1, 3, 2],
>The longest consecutive elements sequence is [1, 2, 3, 4]. Return its length: 4.
>
>Your algorithm should run in O(n) complexity. 

思路:

使用哈希表,将数组的元素映射到哈希表中.然后在遍历数组时查找当前元素的相邻数字是否存在于哈希表中,注意,要分两个方向进行查找.

另外,为了避免重复查找,我们需要在查找哈希表时把找过的数字删除掉.

代码:

    int longestConsecutive(vector<int> &num) {
        if( num.size() == 0 )
            return 0;
            
        unordered_set<int> Set;
        for( int i = 0; i < num.size(); ++i )
            Set.insert(num[i]);
            
        int longest = 0;
        
        for( int i = 0; i < num.size(); ++i ){
            int leftCount = longestConsecutive( num[i], Set, 0 );
            int rightCount = longestConsecutive( num[i]+1, Set, 1 );
            
            if( leftCount + rightCount > longest )
                longest = leftCount + rightCount;
        }
        return longest;
    }
    
    int longestConsecutive( int target, unordered_set<int>& Set, int direction ){
        int count = 0;
        while( Set.find(target) != Set.end() ){
            count++;
            Set.erase( target );
            if( direction == 0 )
                --target;
            else
               ++target;
        }
        return count;
    }

