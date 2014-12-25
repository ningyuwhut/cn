---
layout: post 
title: LeetCode Path Sum II
categories:
- Online Judge
tags:
- LeetCode
- interview
---

> Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum.

代码:

    vetor<vector<int> > pathSum(TreeNode *root, int sum) {
        vector<vector<int> > result;
        if( root == NULL )
            return result;
        
        int currentSum = 0;
        vector<int> path;
        
        pathSum( root, sum, currentSum, result, path );
        
        return result;
    }
    
    /*
     *需要注意的地方,
     *1. 参数path不能是引用,否则result中只有一个元素了.
     *2. currentSum可以是引用,也可以不是引用,如果是引用的话,在找到符合条件的路径时,要记得减去当前节点的值.
     *
     */
    void pathSum(TreeNode* root, int sum ,int& currentSum, vector<vector<int> >& result, vector<int> path ){
        
        path.push_back( root->val );
        currentSum += root->val;
        if( currentSum == sum && root->left == NULL && root->right == NULL ){
         
            currentSum -= root->val;//不要忘了.
            result.push_back(path);
            return;
        }
        
        if( root->left != NULL ){
            pathSum ( root->left, sum, currentSum, result, path );
        }
        if( root->right != NULL ){
            pathSum( root->right, sum, currentSum, result, path );
        }
        
       // path.pop_back();
        currentSum -= root->val;
    }
