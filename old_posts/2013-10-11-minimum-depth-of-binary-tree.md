---
layout: post 
title: LeetCode Minimum Depth of Binary Tree
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Given a binary tree, find its minimum depth.
>
> The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.

思路:

这道题和求最大深度那题思路一样,但是要考虑到子树为空的情况.举个最简单的例子,如果根节点的右子树为空,左子树有一个孩子,那么此时最小深度应该是2,而不是0

代码:

    int minDepth(TreeNode *root) {
        
        if( root == NULL )
            return 0;
        
        if( root->left == NULL && root->right == NULL )
            return 1;
        
        int lminDpt = INT_MAX;
        int rminDpt = INT_MAX;
        if( root->left != NULL )
            lminDpt = minDepth( root->left );
        if( root->right != NULL )
            rminDpt = minDepth( root->right );
        
        if( lminDpt < rminDpt )
            return lminDpt + 1;
        else
            return rminDpt + 1;
    }
