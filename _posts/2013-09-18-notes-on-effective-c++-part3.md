---
layout: post 
title: effective c++ 笔记(3)
categories:
- C++
tags:

- reading notes
---

##条款13: 以对象管理资源

    class Investment{ };
    Investment* createInvestment();//返回动态分配的对象的指针

    void f(){
	Investment* pInv = createInvestment();
	...
	delete pInv;
	}

有时delete是执行不到的,比如过早的return或是异常.

许多资源动态分配后被用于单一区块或函数,它们应在控制流离开区块或函数时被释放(例如上面的createInvestment),标准库中的auto_ptr就是为上面的情形而设计.auto_ptr是一个智能指针,其析构函数自动对其所指对象调用delete.

    void f(){
	auto_ptr<Investment> pInv(createInvestment());
    }

上述例子示范了以对象管理资源的两个关键想法:

**获得资源后立即放进管理对象中**

上面createInvestment返回的资源被当做管理者auto_ptr的初值.事实上,以对象管理资源的观念常称为"Resource Acquisition is Initialization, RAII).因为我们总是在获得一笔资源后于同一语句内以它初始化某个管理对象.

**管理对象运用析构函数确保资源被释放**

auto_ptr销毁时会自动删除所指的对象,所以不要让多个auto_ptr指向同一个对象.否则,对象会被删除多次,而这是未定义的.

为预防这个问题,auto_ptr有一个性质:
> 若通过copy constructor或copy assignment复制它们,它们会变成 null,而复制所得的指针将取得资源的唯一拥有权

    auto_ptr<Investment> pInv(createInvestment());
    auto_ptr<Investment> pInv2(pInv);//pInv2指向对象,pInv为null
    pInv = pInv2;//pInv指向对象,pInv2为null


auto_ptr的替代方案是"reference-counting smart pointer,RCSP)
它追踪共有多少对象指向某个资源,并在没有指针指向它时删除该资源,RCSP类似垃圾回收,不同的是RCSP无法打破环状引用.

TR1的tr1::shared_ptr就是一个RCSP.

    void f(){
	tr1::shared_ptr<Investment> pInv(createInvestment());
    }


    void f(){
	tr1::shared_ptr<Investment> pInv(createInvestment());
	tr1::shared_ptr<Investment> pInv2(pInv);//pInv2和pInv指向同一个对象
	pInv = pInv2;
    }
由于tr1::shared_ptr的复制行为一如预期,它们可以被用于STL容器以及其他"auto_ptr之非正统复制行为并不适用"的语境上

##条款14:在资源管理类中小心copying行为

条款13中的两种智能指针适用与heap-base的资源.

对于non-heap-based资源,我们想自己写一个资源管理类.以Mutex为例.

Mutex对象有两个函数可用:

    void lock(Mutex* pm);
    void unlock(Mutex* pm);
为确保不会忘记将锁住的锁解锁,我们自己建立class来管理锁,其基本结构符合RAII守则:

> 资源在构造期间获得,在析构期间释放

    class Lock{
	public:
	    explicit Lock(Mutex* pm):mutexPtr(pm){
		lock(mutexPtr);
	    }
	    ~Lock(){ unlock( mutexPtr ); }
	private:
	    Mutex* mutexPtr;
    };

客户的用法也符合RAII方式:
    Mutex m;

    {
	...
	Lock m1(&m);
	...

    }

但是,如果Lock对象被复制怎么办?
    
    Lock ml1(&m);
    Lock ml2(ml1);

我们有两种选择

###禁止复制

    class Lock: private Uncopyable{
	public:
	    ...
    };

###使用引用计数
    
有时我们希望保持资源直到最后一个引用者销毁.这时复制RAII对象时应把该资源的被引用数+1.tr1::shared_ptr便是如此.

通常只要内含tr1::shared_ptr成员变量,便可实现出reference-counting copying行为.如果前面的Lock打算使用reference counting,那么我们可以把mutexPtr的类型从Mutex*改为tr1::shared_ptr<Mutex>.但是tr1::shared_ptr的缺省行为是引用计数为0时删除所指对象,这显然不是我们想要的.我们想要的是解锁.

但是tr1::shared_ptr可以指定删除器,这是一个函数或函数对象,引用计数为0时被调用.

    class Lock{
	public:
	    explicit Lock(Mutex* pm):mutexPtr(pm, unlock){//指定unlock为删除器
		lock(mutexPtr.get());
	    }
    //	    ~Lock(){ unlock( mutexPtr ); }
	private:
	    tr1::shared_ptr<Mutex> mutexPtr;
    };

Lock不再需要析构函数,因为class的析构函数会自动调用non-static成员的析构函数,而mutexPtr的析构函数会在引用计数为0时自动调用tr1::shared_ptr的删除器.

    

##条款15:在资源管理类中提供对原始资源的访问

考虑下面的函数
    
    int daysHeld( const Investment* pi );

    int days = daysHeld(pInv);

显然该语句无法通过编译,因为类型不符.
此时我们需要把RAII class对象转化为其内部所含的原始资源.
有两个方法:显示和隐式转换

tr1::shared_ptr和auto_ptr都提供get函数,用来执行显式转化,即返回智能指针内部的原始指针:

    int days = daysHeld(pInv.get());

像所有智能指针一样,shared_ptr和auto_ptr都重载了操作符`->`和`*`,它们允许转换至底部原始指针.

    class Investment{
	public:
	    bool isTaxFree() const;
    };

    Investment* createInvestment();

    tr1::shared_ptr<Investment> pInv(createInvestment());
    bool taxable1 = !(pInv->isTaxFree());

    auto_ptr<Investment> pInv2(createInvestment());
    bool taxable2 = !((*pInv2).isTaxFree());


##条款16:成对适用new和delete时使用相同形式

##条款17:以独立语句将newed对象置入智能指针

一个例子:

    int priority();
    void processWidget( std1::tr1::shared_ptr<Widget>pw, int prioriy);

    processWidget( std::tr1::shared_ptr<Widget>(new Widget), prioriy());

上述代码有可能会出现内存泄露

因为C++对函数参数的计算顺序是不确定的,可能如下顺序:

1. 执行new Widget
2. 调用prioriy
3. 调用tr1::shared_ptr构造函数

想想步骤2出现异常会怎么样,此时new Widget返回的指针可能会遗失.

避免该问题的办法是使用分离语句

    std::tr1::shared_ptr<Widget>(new Widget);
    processWidget( pw, prioriy() );

