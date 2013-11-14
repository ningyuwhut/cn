---
layout: post 
title: Longest Increasing Subsequence
categories:
- Online Judge
tags:
- interview
---

题目:

给定一个数组,找出它的最长递增子序列的长度.

这道题编程之美上有,动态规划的O(n^2)的解法比较好理解,但是上面的解法二很费解.不过felix(http://www.felix021.com/blog/read.php?1587)给出的解法很好理解.

基本思路就是使用一个数组B,B[i]表示所有长度为i的LIS中的最小元素.什么意思呢?最长递增子序列可能不只一个,对于长度相同的LIS,我们只需要记录这些序列的最大元素的最小元素就可以.(有点拗口,好好想一下)

对于一个新的元素a,我们在B中查找它的位置,因为B是有序的,所以可以二分查找.(B有序也是显然的),为什么要查找a在B的位置(记该位置为i)呢?因为查到a的位置i后,我们可以更新该位置上的元素,因为出现了长度为i的LIS且该LIS的最大元素比当前值更小.如果没有找到,那么说明LIS的长度可以加1了.

代码:

    int longestIncreasingSubsequence(vector<int>& array, vector<int>& lis ){
	lis[0] = array[0];//长度为1的LIS的最小元素为array[0]
	int len = 1;//当前LIS的长度为1.
	int left;
	int right;
	int mid;

	for( int i = 1; i < array.size(); ++i ){

	    left = 0;
	    right = len;
	    while( left < right ){
		mid = (left+right)/2;
		if( lis[mid] > array[i] )
		    right = mid-1;
		else if( lis[mid] < array[i] )
		    left = mid+1;
		else{
		    left = mid;
		    break;
		}
	    }

	    lis[left] = array[i];
	    if( left == len ){
		++len;
	    }
	}
	return len;
    }



