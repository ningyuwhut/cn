---
layout: post 
title: LeetCode Reverse Linked List II
categories:
- Online Judge
tags:
- LeetCode
- interview
---

题目描述:

>Reverse a linked list from position m to n. Do it in-place and in one-pass.
>
>For example:

>Given 1->2->3->4->5->NULL, m = 2 and n = 4,
>
>return 1->4->3->2->5->NULL.
>
>Note:

>Given m, n satisfy the following condition:

>1 ≤ m ≤ n ≤ length of list. 

思路:

比较繁琐,代码还需要简化.

代码:

     ListNode *reverseBetween(ListNode *head, int m, int n) {
        if( !head )
            return NULL;
        if( m == n )
            return head;
        ListNode* n1 = head;
        ListNode *beforeBegin = NULL, *begin=NULL, *end=NULL,*afterEnd=NULL;
        int i = 1;
        while( n1 != NULL  ){
            if( i+1 == m )
                beforeBegin = n1;
            if( i == m )
                begin = n1;
            if( i == n )
                end = n1;
            if( i == n+1 ){
                afterEnd = n1;
                break;
            }
            n1 = n1->next;
            i++;
        }
        if( beforeBegin )
            beforeBegin->next = end;
        else
            head = end;
        ListNode* n2, *n3;
        n1 = begin;
        n2 = n1->next;
        while( n1 != NULL ){
            n3 = n2->next;
            n2->next = n1;
            if( n2 == end )
                break;
            n1 = n2;
            n2 = n3;
        }
        if( begin )
            begin->next = afterEnd;
        return head;       
    }
