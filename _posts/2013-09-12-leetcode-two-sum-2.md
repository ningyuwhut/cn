---
layout: post 
title: LeetCode two sum (2)
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:

>Given an array of integers, find two numbers such that they add up to a specific target number.

>The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.

>You may assume that each input would have exactly one solution

>Input: numbers={2, 7, 11, 15}, target=9

>Output: index1=1, index2=2

解法三:

排序后从两端往开始查找,时间复杂度O(nlgn)

代码:

    vector<int> twoSum(vector<int> &numbers, int target) {
	vector<int> index;
	for( int i = 0; i < numbers.size(); ++i )
	      index.push_back(i+1);

	vector<int> result;
	quick_sort(numbers, index, 0, numbers.size() );//升序排列

	int i = 0; 
	int j = numbers.size()-1;

	while( i < j ){
	    int sum = numbers[i] + numbers[j];
	    if( sum  == target ){
		result.push_back( index[i] );
		result.push_back( index[j] );
		sort( result.begin(), result.end() );
		return result;
		
	    }else if( sum > target ){
		--j;
	    }else{
		++i;
	    }
	}
    }

解法四:

使用哈希表

    vector<int> twoSum(vector<int> &numbers, int target) {
	vector< int> result;
	map< int, int> hashmap;
	
	for( int i = 0; i < numbers.size(); ++i ){
	    if( hashmap.count( target - numbers[i] ) ){//显然, hashmap[ numbers[i] 先加入的map,下标更小
		result.push_back( hashmap[ target - numbers[i] ]);
		result.push_back( i+1 );
		return result;
	    }else{
		hashmap[ numbers[i] ] = i+1;
	    }
	}
    }
