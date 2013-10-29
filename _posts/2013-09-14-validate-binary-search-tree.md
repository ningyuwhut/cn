---
layout: post 
title: LeetCode validate binary search tree
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Given a binary tree, determine if it is a valid binary search tree (BST).

> Assume a BST is defined as follows:

>  The left subtree of a node contains only nodes with keys less than the node's key.

>  The right subtree of a node contains only nodes with keys greater than the node's key.

>  Both the left and right subtrees must also be binary search trees.

一个比较丑陋的解法: brute force
代码:

     bool preOrder( TreeNode *node ){
        if( node == NULL )
            return true;
            
        if( preOrder( node, node->left, true ) && preOrder( node, node->right, false) ){
            return preOrder( node->left ) && preOrder( node->right );
        }else{
            return false;
        }
    }
    
    bool preOrder( TreeNode *node, TreeNode* child, bool left ){
        if( node == NULL || child == NULL )
            return true;
        
        bool isBST = true;

        if( left ){//左子树
            if( child->val >= node->val ){
                isBST = false;
            }else{
                isBST = preOrder( node, child->left, true ) && preOrder( node, child->right, true );
            }
        }else{
            if( child->val <= node->val ){
                isBST = false;
            }else{
                isBST = preOrder( node, child->left, false ) && preOrder( node, child->right, false );
            }
        }
        return isBST;
    }
    
    bool isValidBST(TreeNode *root) {
        if( root == NULL )
            return true;
        
        return preOrder( root );
    }

另外,在leetcode论坛上看到一种比较巧妙的办法,下面抄录过来:


    10
   /  \
  5   15     -------- binary tree (1)
     /  \
    6   20

以上面的树为例:

> As we traverse down the tree from node (10) to right node (15), we know for sure that the right node's value fall between 10 and +INFINITY. Then, as we traverse further down from node (15) to left node (6), we know for sure that the left node's value fall between 10 and 15. And since (6) does not satisfy the above requirement, we can quickly determine it is not a valid BST. All we need to do is to pass down the low and high limits from node to node!

代码:

    bool isBSTHelper(BinaryTree *p, int low, int high) {
	if (!p) return true;
	if (low < p->data && p->data < high)
	    return isBSTHelper(p->left, low, p->data) &&  isBSTHelper(p->right, p->data, high);
	else
	    return false;
    }

    bool isBST(BinaryTree *root) {
	// INT_MIN and INT_MAX are defined in C++'s <climits> library
	return isBSTHelper(root, INT_MIN, INT_MAX);
    }


