---
layout: post 
title: LeetCode Swap Nodes in Pairs
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

> Given a linked list, swap every two adjacent nodes and return its head.
>
> For example,

> Given 1->2->3->4, you should return the list as 2->1->4->3.
> 
> Your algorithm should use only constant space. You may not modify the values in the list, only nodes itself can be changed. 

题目较为简单,不过还是没有一遍写对.....

代码:

    ListNode *swapPairs(ListNode *head) {
        if( head == NULL || head->next == NULL )
            return head;
            
        ListNode* newHead = head->next;
        
        for( ListNode* cur = head, *prev = NULL; cur != NULL && cur->next != NULL; cur = cur->next ){
            ListNode* nxt = cur->next;
            cur->next = nxt->next;
            nxt->next = cur;
            
            if( prev != NULL ){
                prev->next = nxt;
            } 
            prev = cur;
        }
        
        return newHead;
    }
