---
layout: post 
title: LeetCode Merge Intervals 
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:
>Given a collection of intervals, merge all overlapping intervals.
>
>For example,

>Given [1,3],[2,6],[8,10],[15,18],

>return [1,6],[8,10],[15,18]. 

思路:
排序后再合并,

需要注意的问题

1. vector在删除一个元素后后面的迭代器都会失效,所以应该使用erase返回的迭代器.
2. 在合并区间时,如果下一个区间的start和当前区间的end相等,也可以合并,这种情况在第一次提交时忽略了.

代码:

    /**
     * Definition for an interval.
     * struct Interval {
     *     int start;
     *     int end;
     *     Interval() : start(0), end(0) {}
     *     Interval(int s, int e) : start(s), end(e) {}
     * };
     */
    int partition( vector<Interval>& intervals, int begin, int end ){
	if( begin < end ){
	    Interval pivot_ele = intervals[begin];
	    
	    while( begin < end ){
		
		while( begin < end && intervals[end].start >= pivot_ele.start ) --end;
		intervals[begin] = intervals[end];
		while( begin < end && intervals[begin].start <= pivot_ele.start) ++begin;
		intervals[end] = intervals[begin];
	    }
	    intervals[begin] = pivot_ele;
	}
	return begin;
    }
    
    void quick_sort( vector<Interval>& intervals, int begin, int end ){
	if( begin < end ){
	    int pivot = partition( intervals, begin, end );
	    quick_sort( intervals, begin, pivot-1);
	    quick_sort( intervals, pivot+1, end);
	}
    }

    vector<Interval> merge(vector<Interval> &intervals) {
	int size = intervals.size();
	quick_sort( intervals, 0, size-1);
	
	vector<Interval> mergedIntervals;
	vector<Interval>::iterator iter = intervals.begin();
	
	while( iter < intervals.end() ){
	    if( (iter+1) < intervals.end() && (iter+1)->start <= iter->end){//相等时也可以合并
		if((iter+1)->end <= iter->end ){//下一个区间完全被当前区间包含
		    iter = intervals.erase((iter+1));
		    --iter;
		}else{
		    Interval new_interval( iter->start, (iter+1)->end );
		    
		    iter = intervals.erase( iter );
		    iter = intervals.erase( iter );
		    iter = intervals.insert( iter, new_interval );
		}
	    }else{
		mergedIntervals.push_back( *iter);
		++iter;
	    }
	}//while
	return mergedIntervals;
    }

