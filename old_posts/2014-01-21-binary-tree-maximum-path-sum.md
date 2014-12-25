---
layout: post 
title: LeetCode Binary Tree Maximum Path Sum
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

  Given a binary tree, find the maximum path sum.
  
  The path may start and end at any node in the tree.
  
  For example:
  Given the below binary tree,
  
         1
        / \
       2   3
  Return 6.

思路：

对二叉树进行后序遍历，在遍历过程中维护全局最大路径长度,记为m，同时计算以左孩子为终止节点的路径的最大长度,记为l, 计算以右孩子伟终止节点的路径的最大长度，记为r。假设当前节点值为r，如果l或r大于零，那么将当前节点加入路径后,如果路径长度大于m,那么更新m。最后利用l和r计算以当前节点为终止节点的路径的最大长度。

这道题目的思路和求二叉树节点的最大距离类似，二叉树中节点的距离定义为两个节点之间的边数，而二叉树节点的最大距离即树中相距最远的两个节点之间的距离。

分析之后可以看出，最大距离分为两种情况，一种是两个节点分别在左右子树中，一种是仅存在于左子树或仅存在于右子树中。

在该问题中，同样需要维护一个全局最大距离d，同时需要计算左子树的最大深度l和右子树的最大深度r，如果l+r大于d,则更新d。最后返回以当前节点为根节点的子树的最大深度。

代码：

    class Solution {
    public:
        int maxPathSum(TreeNode *root) {
            if( root == NULL )
                return INT_MIN;
            int maxSum = INT_MIN;
            maxPathSum( root, maxSum );
            
            return maxSum;
        }
        
        int maxPathSum( TreeNode *root, int& maxSum ){
            
            if( root->left == NULL && root->right == NULL ){
                if( root->val > maxSum )
                    maxSum = root->val;
                return root->val;
            }
                
            int leftMaxSum = INT_MIN;
            int rightMaxSum = INT_MIN;
            
            if( root->left != NULL ){
                leftMaxSum = maxPathSum(root->left, maxSum );
            }
            if( root->right != NULL ){
                rightMaxSum = maxPathSum(root->right, maxSum );
            }
            
            int tmpMaxSum = root->val;
            if( leftMaxSum > 0 )
                tmpMaxSum = leftMaxSum + tmpMaxSum;
            if( rightMaxSum > 0 )
                tmpMaxSum = rightMaxSum + tmpMaxSum;
            if( tmpMaxSum > maxSum )
                maxSum = tmpMaxSum;
            
            return std::max( leftMaxSum , rightMaxSum ) > 0 ? max(leftMaxSum, rightMaxSum ) + root->val : root->val;
        }
    };

最大距离问题代码：

    int MaximumDistance( Node* root, int& maxD ){
	if( root == NULL )
	    return 0;
	
	int leftMaxDepth = 0, rightMaxDepth = 0;
	if( root->left != NULL )
	    leftMaxDepth = MaximumDistance( root->left, MaxD );
	if( root->right != NULL )
	    rightMaxDepth = MaximumDistance( root->right, MaxD );
	if( leftMaxDepth + rightMaxDepth > maxD )
	    maxD = leftMaxDepth + rightMaxDepth + 2;
	return leftMaxDepth > rightMaxDepth ? (leftMaxDepth +1) : (rightMaxDepth + 1) ;
    }

