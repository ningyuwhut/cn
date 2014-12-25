---
layout: post 
title: effective c++ 笔记(2)
categories:
- C++
tags:
- reading notes
---

## 条款5:了解C++默默编写并调用哪些函数

如果你没有自己声明任何构造函数,编译器会为你声明default构造函数(**public且inline的哦**).但你一旦声明了自己的构造函数,编译器就不会再给你构造一个default了.

内含reference成员和const成员的类,编译器不会自动生成copy constructor和copy assignment constructor,你必须自己定义

如果base class将copy assigment声明为private,编译器也会拒绝为其derived class 生成 copy assigment 

## 条款6:若不想使用编译器自动生成的函数,就明确拒绝

为了阻止复制,我们可以把copy constructor and copy assigment constructor 声明为private,既阻止了编译器自动创建,又阻止了复制和赋值操作
但是此时member函数和friend函数还是可以调用private的啊....怎么办呢?

不实现它!! 将成员函数声明为private且不实现它!!

这样调用这两个函数时就会出现链接错误.

iostream中就是用这个trick阻止copying行为.

另外一种方法如下:

    class Uncopyable{
	protected:
	    Uncopyable(){};
	    ~Uncopyable(){};
	private:
	    Uncopyable(const Uncopyable&);
	    Uncopyable& operator=(const Uncopyable&);
    };

    class HomeForSale: private Uncopyable{
	...
    };


## 条款7:为多态基类声明virtual析构函数

当derived class对象经由一个base class指针被删除,而该base class带有一个non-virtual析构函数,其结果是未定义的,实际执行时通常是对象的derived 成分未被销毁.

** 任何class只要带有virtal函数都几乎确定应该也有一个virtual 函数 **

如果class没有virtual函数,通常表示它并不会作为一个base class,此时再将其析构函数设为virtual往往是个馊主意.

pure virtual 函数表示class为abstract class,也就是不会被实体化(instantiated)的class.亦即你不能创建该类型的对象.

析构函数的调用顺序是先调用derived class的析构函数,然后是base class的析构函数

给base class 定义virtual 析构函数只适用于polymorphic base class,这种base class的目的就是通过base class处理derived class.但并非所有的base class的设计目的就是为了多态,比如string和STL容器都不被设计用于base class.

## 条款8:别让异常逃离析构函数

## 条款9:绝不在构造和析构过程中调用virtual函数

    class Transaction{
	public:
	    Transaction();
	    virtual void logTransaction() const = 0;

    };
    Transaction::Transaction(){
	logTransaction();
    }

    class BuyTransaction: public Transaction{
	public:
	    virtual void logTransaction() const;
	    ...
    };

考虑`BuyTransaction b;`会发生什么事情

首先调用Transaction的构造函数,该函数最后一行会调用logTransaction函数.问题就在这里.

这里调用的logTransaction函数是Transaction类里的版本,而不是BuyTransaction的版本.
这是因为base class构造函数在执行期间,derived class的成员变量尚未初始化.

更根本的原因是**在derived class对象的baseclass构造期间,对象的类型是base class类型而不是derived class.**
即对象在derived class构造函数开始执行前不会称为一个derived class对象.

## 条款10: 令operator= 返回一个reference to *this

不仅适用与赋值运算符,也适用于所有赋值相关运算(+=,-=等)

## 条款11:在operator=中处理自我赋值

    class Bitmap{ };
    class Widget{
	private:
	    Bitmap* pb;
    };

    Widget& Widget::operator=(const Widget& rhs){
	delete pb;
	pb = new Bitmap(*rhs.pb);
	return *this;
    }

想想如果*this和rhs是同一个对象会怎样.delete pb把rhs的Bitmap也给删除了!!返回的Widget持有一个指向已经被删除的对象!

    class Bitmap{ };
    class Widget{
	private:
	    Bitmap* pb;
    };

    Widget& Widget::operator=(const Widget& rhs){
	if( this == &rhs) return *this;
	delete pb;
	pb = new Bitmap(*rhs.pb);
	return *this;
    }

跳过了一部分内容

## 条款12: 复制对象时勿忘其每一个成分

    class Date { };
    class Customer{
	public:
	    ...
	private:
	    string name;
	    Date lastTransaction;
    };

    class PriorityCustomer: public Customer{
	public:
	   PriorityCustomer( const PriorityCustomer& rhs );
	   PriorityCustomer& operator=(const PriorityCustomer& rhs);
	   
	private:
	    int priority; 
    };

    PriorityCustomer::PriorityCustomer( const PriorityCustomer& rhs )
     : priority(rhs.priority){
	logcall("...");
     }

    PriorityCustomer& PriorityCustomer::PriorityCustomer& operator=(const PriorityCustomer& rhs)
    {
	priority = rhs.priority;
	return *this;
    }

上面的copying函数只复制了自己的成员变量,但是没有复制继承的Customer,所以编译器会调用Customer的default constructor对其进行初始化.

改成这样:

    PriorityCustomer::PriorityCustomer( const PriorityCustomer& rhs )
     : Customer(rhs), priority(rhs.priority){
	logcall("...");
     }

    PriorityCustomer& PriorityCustomer::PriorityCustomer& operator=(const PriorityCustomer& rhs)
    {
	Customer::operator=(rhs);
	priority = rhs.priority;
	return *this;
    }
    
如果两个copying函数有相似的代码你有可能想让一个函数调用另外一个以避免代码重复,但是这是不合理的!!!
正确的方法是定义一个新的成员函数给这两个函数调用
