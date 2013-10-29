---
layout: post 
title: LeetCode Maximum Depth of Binary Tree
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Given a binary tree, find its maximum depth.
>
> The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.


代码:

    int maxDepth(TreeNode *root) {
        if( root == NULL )
            return 0;
        
        int lMaxDpt = maxDepth( root->left );
        
        int rMaxDpt = maxDepth( root->right );
        
        if( lMaxDpt > rMaxDpt ){
            return lMaxDpt + 1;
        }else{
            return rMaxDpt + 1;
        }
    }
