---
layout: post 
title: LeetCode Recover Binary Search Tree
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Two elements of a binary search tree (BST) are swapped by mistake.
>
>Recover the tree without changing its structure.
>Note:
>A solution using O(n) space is pretty straight forward. Could you devise a constant space solution? 

思路:

刚开始做的时候想简单了,结果写出来发现根本不对.没有想到什么好的O(1)的方法,参考[这篇文章](http://jane4532.blogspot.com/2013/07/recover-binary-search-treeleetcode.html)的思路.

其实**二叉搜索树的中序遍历序列是一个递增序列**,可以利用这一性质找到被交换的两个节点,然后交换它们就可以了.

以下面这棵树为例.

	    2

       / \

     4    1

首先需要两个变量保存被交换的两个节点,分别记为n1,n2,还要有一个节点保存当前节点的前一个节点,记为prev.初始化均为空.

中序遍历,首先找到4和2,4>2,违反增序,那么置n1=4,n2=2.再往右子树遍历,2>1,违反增序,那么置n2=1,因为此时n1不为空,所以不用再调整.

最后,交换4和1的值.


代码:

     void recoverTree(TreeNode *root) {
        TreeNode* n1 = NULL;
        TreeNode* n2 = NULL;
        TreeNode* prev = NULL;
        
        recoverTree( root, n1, n2, prev );
        
        if( !n1 || !n2 )
            return;
        if( n1 != n2 ){
            int tmp = n1->val;
            n1->val = n2->val;
            n2->val = tmp;
        }
    }
    
    void recoverTree( TreeNode* root, TreeNode*& n1, TreeNode*& n2, TreeNode*& prev ){
        
        if( !root )
            return;
        
        recoverTree( root->left, n1, n2, prev );
        
        if( prev && prev->val > root->val ){
            if( !n1 ){
                n1 = prev;
            }
            n2 = root;
        }
        
        prev = root;
        recoverTree( root->right, n1, n2, prev );
    }

其实,判断一颗二叉树是否是一个合法的二叉树时,也可以利用上面的性质.

leetcode上面也有这道题,之前做的时候并没有用过这个思路,下面是把基于这个思路的代码

代码:

    bool isValidBST(TreeNode *root) {
        
        TreeNode* prev = NULL;
        return isValidBST( root, prev );
    }
    
    bool isValidBST( TreeNode* root, TreeNode*& prev ){
        if( !root )
            return true;
        
        bool isValid = isValidBST( root->left, prev );
        if( isValid ){
            if( prev && prev->val >= root->val ){
                return false;
            }
            
            prev = root;
            
            isValid = isValidBST( root->right, prev );
        }
        
        return isValid;
    }

递归遍历二叉树时需要注意的是prev指针,作为参数传递时一定要传引用!因为在函数中会被修改.

下面是非递归的代码:

    bool isValidBST(TreeNode *root) {
        TreeNode* node = root;
        TreeNode* prev = NULL;//前一个节点
        TreeNode* cur = NULL;//当前节点
        
        stack<TreeNode*> s;
        
        while( node != NULL || !s.empty() ){
            if( node != NULL ){
                s.push(node);
                node = node->left;
            }else if( !s.empty() ){
                TreeNode* curNode = s.top();
                s.pop();
                
                cur = curNode;
        
                if( prev && prev->val >= cur->val ){
                    return false;
                }
                prev = cur;
                
                node = curNode->right;
            }
        }
        return true;
    }
