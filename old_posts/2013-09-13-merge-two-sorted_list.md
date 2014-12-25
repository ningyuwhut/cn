---
layout: post 
title: LeetCode merge two sorted list
categories:
- Online Judge
tags:
- LeetCode
- interview
---


题目描述:

> Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

题目比较简单

代码:

    ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
        if( l1 == NULL )
            return l2;
        if( l2 == NULL )
            return l1;
            
        ListNode* l3;
        ListNode* l4;
        
        if( l1->val < l2->val ){
                l3 = l1;
                l1 = l1->next;
        }else{
            l3 = l2;
            l2 = l2->next;
        }
        
        l4 = l3;
        while( l1 != NULL && l2 != NULL ){
            if(l1->val < l2->val ){
                l4->next = l1;
                l4 = l4->next;
                l1 = l1->next;
            }else{
                l4->next = l2;
                l4 = l4->next;
                l2 = l2->next;
            }
        }
           while( l1 != NULL ){
            l4->next = l1;
            l4 = l4->next;
            l1 = l1->next;
        }
        while( l2 != NULL ){
            l4->next = l2;
            l4 = l4->next;
            l2 = l2->next;
        }
        return l3;
    }

[这里](http://discuss.leetcode.com/questions/202/merge-two-sorted-lists) 有各种版本的程序可供参考.
