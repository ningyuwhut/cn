---
layout: post 
title: LeetCode Balanced Binary Tree
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

>Given a binary tree, determine if it is height-balanced.
>
>For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1. 

思路:

对每个节点,判断左右子树的深度,如果深度之差超过1,则返回false,否则,继续判断左右子树

这个思路会对同一个节点重复判断多次,所以效率较低

另外一种思路是使用后序遍历,在遍历子树时记录子树的高度.

代码:

    bool isBalanced(TreeNode *root) {
        if( root == NULL )
            return true;
        
        int ldepth = depth( root->left );
        int rdepth = depth( root->right );
        if( ldepth - rdepth >1 || ldepth - rdepth < -1 )
            return false;
        else
            return isBalanced( root->left ) && isBalanced( root->right );
    }
    
    int depth( TreeNode* node ){
        if( node == NULL )
            return 0;
        int ldepth = 1 + depth( node->left );
        int rdepth = 1 + depth( node->right );
        return ldepth > rdepth ? ldepth : rdepth;
    }


思路二:

    bool isBalanced(TreeNode *root) {
        int depth;
        return isBalanced( root, depth );
    }
    
    bool isBalanced( TreeNode* node, int& depth ){
        if( node == NULL ){
            depth = 0;
            return true;
        }

        int leftDepth, rightDepth;
        bool left = isBalanced( node->left, leftDepth );
        bool right = isBalanced( node->right, rightDepth );
        if( left != true || right != true )
            return false;
        else{
            if( leftDepth - rightDepth > 1 || leftDepth - rightDepth < -1 )
                return false;
            else
                depth = leftDepth-rightDepth>0 ? leftDepth+1:rightDepth+1;
        }
        return true;
    }

