---
layout: post 
title: LeetCode Median of Two Sorted Arrays 
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> There are two sorted arrays A and B of size m and n respectively. Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

这道题没有做出来,参考别人的答案给出解法,还要再理解

解法:

这道题可以转化为求解第K大(小)元素.

思路如下(这里使用求解第K小元素的思路):

比较A中第k/2小的元素(A[k/2-1])和B中第k/2小的元素(B[k/2-1]),有如下结果:

    A[k/2-1] = B[ k/2-1]
    A[k/2-1] < B[ k/2-1]
    A[k/2-1] > B[ k/2-1]

如果 A[k/2-1] < B[k/2-1] ,那么A[k/2-1]之前的所有元素(包含自身),即A[0...k/2-1]不可能含有第k小的元素.

因为此时A中比A[k/2-1]小的元素为k/2-1,而B中比A[k/2-1]小的元素最多有k/2-1个,所以,A和B中比A[k/2-1]小的元素最多为 k/2-1 + k/2-1 = k -2 个,那么A[k/2-1]最大也只能是第k-1小的元素.

所以,我们可以只需要考虑A[k/2-1]后半部分的元素即可.

同理,当` A[k/2-1] > B[ k/2-1] `时,只需要考虑B[k/2-1]后半部分的元素即可.

当两者相等时,说明第k小元素已经找到,此时,A和B中分别有k/2-1个元素小于A[k/2-1] (或B[k/2-1]),返回两者之一.


代码:

    double min( int lhs, int rhs ){
        if( lhs < rhs )
            return lhs;
        else
            return rhs;
    }
    
    double findKth( int A[], int m, int B[], int n, int k ){
        if( m > n )
           return findKth( B, n, A, m , k );
        if( m == 0 )
            return B[k-1];
        if( k == 1 )
            return min( A[0], B[0] );
            
        int pa = min(k/2, m ); 
        int pb = k - pa;
            
        if( A[pa-1]  < B[pb-1] ){
            return findKth( A+ pa, m -pa, B, n, k-pa );
        }else if( A[pa-1] > B[pb-1] ){
            return findKth( A, m, B+ pb, n-pb, k-pb );
        }else{
            return A[pa-1];
        }      
    }
   
    double findMedianSortedArrays(int A[], int m, int B[], int n) {
        int total = m+n;
        if( total & 0x1 ){//奇数
            return findKth(A, m, B, n, total/2+1);
        }else{//偶数
            return ( findKth(A,m, B,n, total/2) + findKth(A,m,B,n, total/2+1) ) /2;
        }
    }

参考:

1. <http://blog.csdn.net/zxzxy1988/article/details/8587244>
