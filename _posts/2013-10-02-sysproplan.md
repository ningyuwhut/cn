---
layout: post 
title: 系统程序员成长计划笔记
categories:
- c
---

今天看了之前读过的<系统程序员成长计划>,有些地方值得记一下.

隐藏数据结构

如果是内部数据结构,外面完全不会引用,则直接放在c文件中,千万不要放在头文件中.

如果该数据结构在内外都要使用,则可以对外暴露结构的名字,而封装结构的实现细节,做法如下:

1. 在头文件中声明该数据结构
2. 在C文件中定义该数据结构

dlist.h

    struct _DList;
    typedef struct _DList DList;  

dlist.c

    //节点类在外部不会用到,定义在c文件中
    typedef struct _DListNode{
	struct _DListNode* prev;
	struct _DListNode* next;
	void* data;
    }DListNode;

    struct _DList{
	DListNode* first;
    };

隐藏内部函数

1. 在头文件中,只放最少的接口函数的声明
2. 在c文件中,所有内部函数都加上static关键字

以前写c程序时并不经常用static,但是书中在实现双向链表时,把所有节点相关的函数都声明为static,这样这些函数在外部就是不可见的.

拥抱变化

实现一个通用的链表,以打印为例.既然通用,那么就应该这个打印函数就应该可以应对不同的数据类型.怎么办呢?

答案就是#回调函数#

回调函数就是实现者调用调用者提供的函数,把变化的部分留给调用者来实现.

dlist.h

    typdef DListRet (*DListDataPrintFunc)(void* data);
    DListRet dlist_print(DList* thiz, DListDataPrintFunc print );

dlist.c
    
    DListRet dlist_print( DList* thiz, DListDataPrintFunc pritn ){
	DListRet ret = DLIST_RET_OK;
	DListNode* iter = thiz->first;

	while( iter != NULL ){
	    print( iter->data );
	    iter = iter->next;
	}

	return ret;
    }

现在我们开始调用它:

    static DListRet print_int( void* data ){

	printf("%d", (int)data);
	return DLIST_RET_OK;
    }
    ...
    dlist_print(dlist, print_int );

打印函数的主体就是遍历一遍链表,其实还有很多功能需要遍历链表,这样我们可以提炼出一个遍历函数,通过回调函数完成具体的功能.另外,有些功能需要一些参数信息,怎么办呢?全局变量无疑是一个比较糟糕的选择, 我们可以将这些参数传递给遍历函数.如果参数有多个,则可以把这些参数封装成一个结构体.

    DListRet dlist_foreach( DList* thiz, DListVisitFunc visit, void* ctx ){
	DListRet ret = DLIST_RET_OK;
	DListNode* iter = thiz->first;
	while( iter != NULL && ret != DLIST_RET_STOP ){
	    ret = visit( ctx, iter->data );
	    iter = iter->next;
	}

	return ret;
    }

visit是回调函数,ctx就是函数的上下文

    static DListRet sum_cb( void* ctx, void* dataa ){
	long long* result = ctx;
	*result += (int)data;
	return DLIST_RET_OK;
    }

    long long sum = 0;
    dlist_foreach( thiz, sum_cb, &sum );

这中方法在APUE中经常看到,比如pthread_create函数.


