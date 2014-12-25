---
layout: post 
title: LeetCode construct binary tree from inorder and postorder traversal 
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:

> Given inorder and postorder traversal of a tree, construct the binary tree.

> Note:
> You may assume that duplicates do not exist in the tree.

思路:

后续遍历的最后一个元素肯定是根,然后可以根据这个元素把中序遍历序列分成左右两部分,
而后续遍历序列也可以根据中序遍历序列中左右子树的个数来划分边界

代码:

    TreeNode* buildChildTree( vector<int>& inorder, int begin1, int end1,
                         vector<int>& postorder, int begin2, int end2){
        if( begin1 > end1 ){
            return NULL;
        }
            
        TreeNode* node = new TreeNode(postorder[end2]); 
        node->left = node->right = NULL;
        
        int nodeIndexInorder = find( inorder.begin(),inorder.end(),postorder[end2] ) - inorder.begin();
       
     /*
      * nodeIndexInorder-1-begin1+1为以node为根节点的左子树的节点数目,
      * 与该该子树对应的后序遍历序列为[begin2,begin2+ nodeIndexInorder-1-begin1],
      * 而右子树的后序遍历的起始节点下标就是begin2+ nodeIndexInorder-begin1.
      */ 
        node->left = buildChildTree(inorder, begin1, nodeIndexInorder-1, postorder, begin2,begin2+ nodeIndexInorder-1-begin1  );
        node->right = buildChildTree(inorder, nodeIndexInorder+1, end1, postorder, begin2+ nodeIndexInorder-begin1, end2-1 );
        
        return node;
    }
    
    TreeNode *buildTree(vector<int> &inorder, vector<int> &postorder) {
        if( inorder.size() == 0 || postorder.size() == 0 )
	      return NULL;
        TreeNode* root = buildChildTree( inorder, 0, inorder.size()-1, postorder, 0, postorder.size()-1);
        return root;
    }
