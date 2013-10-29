---
layout: post 
title: LeetCode Generate Parentheses
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:

> Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.
>
> For example, given n = 3, a solution set is:
>
> "((()))", "(()())", "(())()", "()(())", "()()()" 

思路:

深度优先搜索

问题需要考虑的是如何产生"well-formed"的字符串.

我的思路是使用两个变量needToMatch和leftCount.
needToMatch表示字符串中现在需要匹配的左括号个数.
leftCount表示字符串中已经出现的左括号个数.
当needToMatch为0时,表示此时只能使用左括号
当leftCount小于n时,此时还能使用左括号,否则只能使用右括号.

代码:

    vector<string> generateParenthesis(int n) {
       vector<string> result;
       string tmp;
       generateParenthesis( n, result, tmp, 1, 0,0);
       return result;
    }
    
    void generateParenthesis( int n, vector<string>& result, string tmp, int step, int needToMatch, int leftCount ){
        if( step > 2*n ){
            result.push_back( tmp );
            return;
        }
        
        if( needToMatch == 0 ){
            tmp += '(';
            generateParenthesis( n, result, tmp, step+1, needToMatch+1,leftCount+1);
        }else{
            if( leftCount < n ){
                tmp += '(';
                generateParenthesis( n, result, tmp, step+1, needToMatch+1, leftCount+1);
                tmp.erase(tmp.size()-1);
            }
            tmp += ')';
            generateParenthesis( n, result, tmp, step+1, needToMatch-1, leftCount);
        }
    }
