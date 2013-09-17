---
layout: post 
title: LeetCode binary tree inorder traversal 
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:

> Given a binary tree, return the inorder traversal of its nodes' values.

> For example:

> Given binary tree {1,#,2,3},

> return [1,3,2].

> Note: Recursive solution is trivial, could you do it iteratively?

基础题


代码:

    vector<int> inorderTraversal(TreeNode *root) {
        vector<int> a;
        if( root == NULL )
            return a;
        TreeNode* node = root;
        
        stack<TreeNode*> nodestack;
        
        while( node != NULL || !nodestack.empty() ){
            
            if( node != NULL ){
                nodestack.push( node);
                node = node->left;
            }else{
                node = nodestack.top();
                nodestack.pop();
                a.push_back( node->val);
                node = node->right;
            }
        }
        return a;
    }

第一次提交时我先把根节点入栈,然后再进入循环.
此时,在`node!=NULL`时,要判断是否把node加入到栈中.因为此时node可能已经在栈中,所以要多几个判断,此时的代码显然
不如上面的简洁

代码:

     vector<int> inorderTraversal(TreeNode *root) {
        vector<int> a;
        if( root == NULL )
            return a;
        TreeNode* node = root;
        
        stack<TreeNode*> nodestack;
        
        nodestack.push( node );
        
        while( node != NULL || !nodestack.empty() ){
            
            if( node != NULL ){
                if( nodestack.empty() || node != nodestack.top() )//如果栈为空,则加入node,否则判断node是否在栈中,不在则加入
                    nodestack.push( node);
                else{
                    node = node->left;
                    if( node != NULL )
			nodestack.push( node);
                }
            }else{
                node = nodestack.top();
                nodestack.pop();
                a.push_back( node->val);
                node = node->right;
            }
        }
        return a;
    }
