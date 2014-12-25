---
layout: post 
title: LeetCode Sum Root to Leaf Numbers
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

>Given a binary tree containing digits from 0-9 only, each root-to-leaf path could represent a number.
>
>An example is the root-to-leaf path 1->2->3 which represents the number 123.
>
>Find the total sum of all root-to-leaf numbers.
>
>For example,
>
>    1
>   / \
>  2   3
>
>The root-to-leaf path 1->2 represents the number 12.
>The root-to-leaf path 1->3 represents the number 13.
>
>Return the sum = 12 + 13 = 25. 

代码:

    int sumNumbers(TreeNode *root) {
        
        if( !root )
            return 0;
            
        int result = 0;
        
        vector<int> path;
        
        sumNumbers( root, path, result );
        
        return result;
    }
    
    void sumNumbers( TreeNode *node, vector<int> path, int& result ){
        if( !node )
            return;
            
        path.push_back( node->val );
        
        if( !node->left && !node->right ){
            int number = 0;
            
            for( vector<int>::iterator iter = path.begin(); iter < path.end(); ++iter ){
                number = 10*number + *iter;
            }
            result += number;
            return;
        }
        
        if( node->left != NULL )
            sumNumbers( node->left, path, result );
        if( node->right != NULL )
            sumNumbers( node->right, path, result );
    }
