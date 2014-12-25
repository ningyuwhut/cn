---
layout: post 
title: LeetCode Permutation Sequence
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> The set [1,2,3,…,n] contains a total of n! unique permutations.

> By listing and labeling all of the permutations in order,
> We get the following sequence (ie, for n = 3):

>    "123"

>    "132"

>    "213"

>    "231"

>    "312"

>    "321"

> Given n and k, return the kth permutation sequence.

> Note: Given n will be between 1 and 9 inclusive.

思路:

这道题做的比较曲折,刚开始时生成了所有的K个排列,还用vector保存起来了.运行时提示内存超了,然后就不保存排列,
当生成第K个排列时就将其返回.这时运行提示时间超了.这时才想到应该是可以不生成前面的排列直接计算出第K个排列是
什么样的.

这个算法想了很久才明白,太弱了....之后按着自己的理解写下来,幸好对了,也验证了我的理解没错.

假设排列为a1,a2,...,an,那么a1的值可以这样来看:
 
a1前面肯定还有0...(a1-1),而以1,...,(a1-1)开头的排列分别有(n-1)!个( n-1个元素的排列数为(n-1)!个 ),
这样a1 = (K-1) / (n-1)! 注意,a1并不是排列中第一位数的值,而是第一位值在[1,n]区间的下标(从0开始).就是a1为2时,排列的第一位数
为3....
同理,我们可以计算剩下的所有位数.

计算a2时,K已经不是最初的K了,因为计算a1时已经计算掉了a1*(n-1)!个排列了,还剩下K%(n-1)!个排列,即K = K %(n-1)!,
此时a2 = K %(n-2)!.和a1一样,a2也是下标,但是此时区间[1,n]中在第一位出现的数字必须排除掉,因为排列不能有重复数字.

感觉这个方法还是很绕的,先举个简单的例子吧

以题目中的例子为例,此时n=3,我们看第3个排列是如何计算的,即K=3时排列是什么.

首先计算第1位,此时n=3,K=3, a1 = K/(3-1)! = 1,即第1位应该为2,(区间[1,3]中下标为1的元素)
计算第2位,此时K=1, a2 = K/(3-2)! = 1, 即第2位应该为3,(区间[1,3]中下标为1的元素,因为2已经出现过了,所以区间中还剩下1,3,下标为1的元素自然是3.
计算第3为,此时K=0, a3 = K/(3-3)! = 0,即第3位应该为1.

这样第3个排列应该是"231",和题目中第4个排列一样,也就是说我们想要第K个排列时其实计算K-1时的排列是多少就ok.至于原因我也说不太清楚....

后记:

看了8数码,原来这个问题就是康托展开的逆运算.

代码

    string getPermutation(int n, int k) {
	//计算n!
        int factorial=1;
        for( int i = 1; i <=n; ++i ){
            factorial *= i;
        }
        
        string result="";
        int tmp;
        vector<int> vec;
        k-=1;
        
        for( int i = 1; i <= n; ++i){
            factorial /= (n-i+1);
            
            tmp = k /factorial;
            
            int m = -1;
            
            for( int j = 0; j < n; ++j ){
                if( find(vec.begin(), vec.end(), j+1) == vec.end() ){//没找到
                    ++m;
                    if( m == tmp ){
                        vec.push_back(j+1);
                        result += (j+1+'0');
                        break;
                    }
                }
            }
            
            k %= factorial;
        }
        return result;
    }
下面是最初的超时做法

    int number = 0;//全局计数器
    string result;
    string getPermutation(int n, int k) {
        number = 0;
        string result;
        string tmp;
        generatePermutation( n, 0, tmp, result, k);
        return result;
    }
    
    void generatePermutation( int n, int step, string tmp, string& result, int k ){
        if( step > n  )
            return;
        if( step == n || number > k ){
            ++number;
            if( number == k)
                result = tmp;
            return;
        }
        
        string old_tmp = tmp;
        for( int i = 1; i <=n; ++i ){
            if( tmp.find( to_string(i) ) == string::npos  ){//i不在tmp中
                tmp += to_string(i);
                generatePermutation(n , step+1, tmp, result, k );
                tmp = old_tmp;
            }
        }
    }
