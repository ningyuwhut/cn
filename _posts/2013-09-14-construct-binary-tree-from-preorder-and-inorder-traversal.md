---
layout: post 
title: LeetCode construct binary tree from preorder and inorder traversal 
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:

> Given preorder and inorder traversal of a tree, construct the binary tree.

> Note:
> You may assume that duplicates do not exist in the tree.


思路和代码都和给定中序和后序遍历序列时相同.

先序遍历序列的第一个元素是根节点,然后在中序遍历序列中找到这个元素,那么序列被分成两个部分,根据两个部分长度确定先序遍历序列中相应的两部分

代码:


    TreeNode *buildChildTree( vector<int> &preorder, int begin1, int end1,
                              vector<int> &inorder, int begin2, int end2 ){
        
        if( begin1 > end1 ){
            return NULL;
        }
        
        TreeNode* node = new TreeNode(preorder[begin1]);
        node->left = node->right = NULL;
        
        int nodeIndexInorder = find( inorder.begin(), inorder.end(), preorder[begin1]) - inorder.begin();
        
        int leftChildTreeNodeNumber = nodeIndexInorder - begin2 ;
        
        node->left = buildChildTree( preorder, begin1+1, begin1+ leftChildTreeNodeNumber,
                                     inorder, begin2, nodeIndexInorder-1);
        node->right = buildChildTree( preorder, begin1+leftChildTreeNodeNumber+1, end1,
                                      inorder, nodeIndexInorder+1, end2 );
        return node;
    }

    TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder) {
        TreeNode* root = buildChildTree(preorder, 0, preorder.size()-1, inorder, 0, inorder.size()-1 );
        
        return root;
    }
