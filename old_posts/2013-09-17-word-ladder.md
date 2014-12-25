---
layout: post 
title: LeetCode word ladder
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:
> Given two words (start and end), and a dictionary, find the length of shortest transformation sequence from start to end, such that:

>    Only one letter can be changed at a time
>    Each intermediate word must exist in the dictionary

> For example,

> Given:

> start = "hit"

> end = "cog"

> dict = ["hot","dot","dog","lot","log"]

> As one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog",
 return its length 5.

> Note:

>    Return 0 if there is no such transformation sequence.

>    All words have the same length.

>    All words contain only lowercase alphabetic characters.

没有做出来,看了一下[这里](http://discuss.leetcode.com/questions/1108/word-ladder).

说一下我的理解.

使用广度优先搜索(BFS),把dict中的每个单词当做图中的节点(可以并不显式构造图),当然源节点和目的节点也要
加入到图中.然后从源点start出发,进行BFS遍历,首先找到end时的路径长度就是最后的最短路径

在实现时,有两种思路:

1. 使用`unordered_set`保存已经在最短路径上的节点(单词),这样做一来可以避免把已经在路径中的节点再加入到路径
中形成环路,二来可以以O(1)的时间判断一个单词是否已经在集合中.

2.不使用额外的数据结构,而是修改dict.如果一个单词已经被访问,那么把它从dict中删除,这样同样可以避免回路

另外,还有一个问题就是何时把路径长度加一呢?因为是BFS,所以应该在进入下一层时把路径长度加1,那么如何判断是否进入下一层呢?

使用两个变量lev1和lev2,lev1表示当前层尚未访问到的元素个数,初始化为1,lev2表示下一层当前入队的元素个数,初始化为0.当队列元素出队时lev1减1,表示当前层尚未访问的元素个数又少了一个.有元素入队时lev2加1,说明下一层中进入队列的元素又多了一个.当lev1==0时,说明此时已经遍历完了当前层,可以进入下一层,注意,当遍历完当前层后,下一层的所有元素都已经进入队列,也就是说lev2就是下一层的元素个数.此时我们把lev2赋值给lev1,并把lev2置0,最后将路径长度+1.再次进入下一层的遍历.


代码:

    list<string> matched_list(string &str, unordered_set<string> &dict) {
        list<string> l;
        for (size_t s = 0; s < str.length(); s++) {
            string temp = str;
            for (char c = 'a'; c <= 'z'; c++) {
                temp[s] = c;
                if (temp != str && dict.find(temp) != dict.end()) {
                    l.push_back(temp); 
                    dict.erase(temp);
                }
            }
        }
        return l;
    }

    class Solution {
    public:
        int ladderLength(string start, string end, unordered_set<string> &dict) {
            dict.insert(end);
            queue<string> q;
            list<string> l;
            list<string>::iterator lit;
            int ladderLength = 0;
            int lev1 = 1; 
            int lev2 = 0;
            q.push(start);
            while (!q.empty()) {
                string str = q.front();
                q.pop();
                lev1--;
                if (str == end) {
                    ladderLength++; 
                    break;
                }
                l = matched_list(str, dict);
                for (lit = l.begin(); lit != l.end(); lit++) {
                    q.push(*lit);
                    lev2++;
                }
                if(lev1 == 0) {
                    ladderLength++;
                    lev1 = lev2;
                    lev2 = 0;
                }
            }
            if (ladderLength==1) ladderLength=0;
            return ladderLength;
        }
    };

