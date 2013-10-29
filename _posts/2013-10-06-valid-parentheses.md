---
layout: post 
title: LeetCode Valid Parentheses
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

> The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.

思路:

使用栈,如果栈空,则直接入栈,否则进行如下判断:

出现左括号,入栈;出现右括号,则判断是否与当前栈顶匹配,如果匹配则弹出栈顶,否则,返回false

最后,判断栈是否为空,不为空则返回false

代码:

    bool isValid(string s) {
        stack<int> a;
        
        for( int i  = 0; i < s.size(); ++i ){
            if( a.empty() ){
                a.push( s[i] );
            }else{
                switch (s[i] ){
                    case '(':
                       a.push( s[i] );
                       break;
                     case '[':
                       a.push( s[i] );
                       break;
                     case '{':
                       a.push( s[i] );
                       break;
                     case ')':
                       if( a.top() != '(')
                           return false;
                       else
                          a.pop();
                     break;
                     case ']':
                       if( a.top() != '[')
                           return false;
                       else
                          a.pop();
                     break;
                     case '}':
                       if( a.top() != '{')
                           return false;
                       else
                          a.pop();
                     break;
                }
            }
        }
       if( !a.empty() ){
           return false;
       }
       return true;
    }
