---
layout: post 
title: LeetCode two sum (1)
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

解法一:

穷举,时间复杂度为O(n^2 )

代码:

    vector<int> twosum(vector<int> &numbers, int target){
	vector<int> result;
	for( int i = 0; i < numbers.size(); ++i ){
	    for(int j = i+1; j < numbers.size(); ++j  ){
		if( numbers[i] + numbers[j] == target ){
		    result.push_back(i+1);
		    result.push_back(j+1);
		    return result;
		}
	    }
	}
	return result;
    }


解法二:

先排序,再二分查找,时间复杂度为O(nlgn)

这里使用快排

代码:

    void quick_sort( vector<int>& numbers, vector<int>& index, int begin, int end ){
	if( begin >= end-1 )
	    return;
	int k = partition( numbers, index, begin, end);
	quick_sort( numbers, index, begin, k);
	quick_sort( numbers, index, k+1, end );
    }

    int partition(vector<int>& numbers, vector<int>& index, int begin, int end){
	int pivot = numbers[begin];
	int pivotIndex = index[begin];
	int i = begin;
	int j = end;

	while( i < j ){
	    while( numbers[++i] < pivot && i < end );
	    while( numbers[--j] > pivot && j > begin );

	    if( i < j ){
		int tmp = numbers[i ];
		numbers[i] = numbers[j];
		numbers[j] = tmp;

		int tmpIndex = index[i];
		index[i] = index[j];
		index[j] = tmpIndex;
	    }
       }

	numbers[begin] = numbers[j];
	numbers[j] = pivot;
	index[begin] = index[j];
	index[j] = pivotIndex;
	return j;
    }

    vector<int> twoSum(vector<int> &numbers, int target) {
	vector<int> index;
	for( int i = 0; i < numbers.size(); ++i )//保存排序后元素的下标,从1开始
	      index.push_back(i+1);
	
	vector<int> result;
	quick_sort(numbers, index, 0, numbers.size() );//升序排列
	
	for( int i = 0; i < numbers.size(); ++i )
	{
	    int target_minus_i = target - numbers[i];
	    
	    int low = 0;
	    int high = numbers.size() -1;
	    int mid ;
	    
	    while( low <= high ){
		mid = (low+high)/2;
		if( numbers[mid] == target_minus_i ){
		    if( mid != i ){
			result.push_back(index[i]);
			result.push_back(index[mid]);
			sort( result.begin(), result.end() );
			return result;
		    }else if( mid >0 && numbers[mid-1] == target_minus_i ){
			result.push_back(index[i]);
			result.push_back(index[mid-1]);
			sort( result.begin(), result.end() );
		       return result;
		    }else if( mid < numbers.size()-1 && numbers[mid+1] == target_minus_i ){
			result.push_back(index[i]);
			result.push_back(index[mid+1]);
			sort( result.begin(), result.end() );
		      return result;
		    }
		}else if( numbers[mid] > target_minus_i ){
		    high = mid-1;
		}else{
		    low = mid+1;
		}
	    }
	}
    }

