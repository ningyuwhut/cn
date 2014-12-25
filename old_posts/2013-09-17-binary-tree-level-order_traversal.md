---
layout: post 
title: LeetCode binary tree level order traversal 
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Given a binary tree, return the level order traversal of its nodes' values. (ie, from left to right, level by level).
>
> For example:

> Given binary tree {3,9,20,#,#,15,7},
>
> return its level order traversal as:
>
> [

>  [3],

>  [9,20],

>  [15,7]

> ]

思路:

和 word ladder那道题一样,使用lev1和lev2标记层次,把我在另外一篇文章的理解转述过来

> 使用两个变量lev1和lev2,lev1表示当前层尚未访问到的元素个数,初始化为1,lev2表示下一层当前入队的元素个数,初始化为0.当队列元素出队时lev1减1,表示当前层尚未访问的元素个数又少了一个.有元素入队时lev2加1,说明下一层中进入队列的元素又多了一个.当lev1==0时,说明此时已经遍历完了当前层,可以进入下一层,注意,当遍历完当前层后,下一层的所有元素都已经进入队列,也就是说lev2就是下一层的元素个数.此时我们把lev2赋值给lev1,并把lev2置0,最后将路径长度+1.再次进入下一层的遍历.

在<编程之美>中也有这道题,除了递归外,里面也讲解了另外一种类似这里的方法.

使用两个变量cur和last,cur表示当前层已经访问的元素个数,初始化为0,last表示当前层的元素个数,初值为1. 当元素出队时cur+1,当cur与last相等时,说明该层已经访问完.此时可以进入下一层,再次把cur置0,last置为当前队列的大小.

代码:

    vector<vector<int> > levelOrder(TreeNode *root) {
        vector<vector<int>> result;
        if( root == NULL )
            return result;
        TreeNode* node = root;
        queue<TreeNode*> nodequeue;
        nodequeue.push(node);
        int lev1 = 1;
        int lev2 = 0;
        vector<int> levelNode;
        
        while( !nodequeue.empty()){
            node = nodequeue.front();
            nodequeue.pop();
            levelNode.push_back(node->val);
            --lev1;
            
            if( node->left != NULL ){
                nodequeue.push( node->left );
                ++lev2;
            }
            if( node->right != NULL ){
                nodequeue.push( node->right );
                ++lev2;
            }
                
             if( lev1 == 0 ){
                result.push_back( levelNode);
                levelNode.clear();
                lev1 = lev2;
                lev2 = 0;
            }
        }
        
        return result;
    }


下面的代码按照编程之美的思路实现

    vector<vector<int> > levelOrder(TreeNode *root) {
        vector<vector<int>> result;
        if( root == NULL )
            return result;
            
        TreeNode* node = root;
        queue<TreeNode*> nodequeue;
        nodequeue.push(node);
        int cur = 0;
        int last = 1;
        
        vector<int> levelNode;

        while( !nodequeue.empty()){
            node = nodequeue.front();
            nodequeue.pop();
            levelNode.push_back(node->val);
            ++cur;
            
            if( node->left != NULL ){
                nodequeue.push( node->left );
            }
            if( node->right != NULL ){
                nodequeue.push( node->right );
            }
                
            if( cur == last ){
                result.push_back( levelNode);
                levelNode.clear();
                cur = 0;
                last = nodequeue.size();
            }
        }
        return result;
    }
