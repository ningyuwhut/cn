---
layout: post 
title: effective c++ 笔记(5)
categories:
- C++
tags:
- reading notes
---

##条款26:尽可能延后变量定义式的出现时间

看代码:

    string encryptPassword( const string& password ){
	string encrypted;
	if( password.length() < MinimumPasswordLength){
	    throw logic_error("...");
	}
	...
	return encrypted;
    }
//由于if判断可能会有异常,那么encrypted可能并没有用到但我们却付出了构造和析构的成本,为避免这种情况,延后其定义

    string encryptPassword( const string& password ){
	if( password.length() < MinimumPasswordLength){
	    throw logic_error("...");
	}
	string encrypted;
	...
	return encrypted;
    }

但是一般我们在构造一个对象时通常会给予一个初值,我们可以这么做:

    string encryptPassword( const string& password ){
	if( password.length() < MinimumPasswordLength){
	    throw logic_error("...");
	}
	string encrypted;
	encrypted = password;
	...
	return encrypted;
    }
但是这样我们会付出一个默认构造函数和一个copy assignment的代价,其实我们可以直接指定初值,这样就不用default constructor了.

    string encryptPassword( const string& password ){
	if( password.length() < MinimumPasswordLength){
	    throw logic_error("...");
	}
	string encrypted(password);
	...
	return encrypted;
    }

但是循环时又该如何?

A:                    
    
    Widget w;                               
    for( int i = 0; i < n; ++i ){    
	w = 取决于i的某个值;	
	...		
    }                  

B:

    for( int i = 0; i < n; ++i ){    
	Widget w(取决于i的某个值);
	...		
    } 

A和B选哪个呢?

A的成本:1个构造,1个析构,n个赋值
B的成本:n个构造,n个析构

如果class的一个赋值成本低于一组构造和析构成本,那么A会比较高效.否则B高效些.
但是A造成w的作用域比做法B更大,有时会对程序的可理解性和可维护性造成冲突,所以除非(1)你知道赋值成本比"构造+析构"成本低,(2)你正在处理代码中效率高度敏感的部分,否则你应该使用做法B


##条款27: 尽量少做转型动作

先跳过

##条款28: 避免返回handles指向对象内部成分

    class Point{
	public:
	Point(in x, int y);
	void setX(int newVal);
	void setY(int newVal);
    };
    struct RectData{
	Point ulhc;
	Point lrhc;
    };
    class Rectangle{
	...
	Point& upperLeft() const { return pData->ulhc; }
	Point& lowerRight() const { return pData->lrhc; };
	private:
	tr1::shared_ptr<RectData> pData;
    };

上面的设计有什么问题呢?

upperLeft和lowerRight声明为const函数,因为其目的是客户只能得到坐标点,但是不能修改过它.可是函数返回的reference却使客户可以修改内部数据.
    
    Point p1(0,0);
    Point p2(100,100);
    const Rectangle rec(p1,p2);
    rec.upperLeft().setX(50);

此时,ulhc和lrhc其实就是public了

这种情形对于指针和迭代器同样成立.

references,pointer,iterators统统是所谓的handles,而返回一个代表对象内部数据的handle,随之而来的便是降低对象封装性的风险

怎么办呢?

    class Rectangle{
	...
	const Point& upperLeft() const { return pData->ulhc; }
	const Point& lowerRight() const { return pData->lrhc; };
	...
    };

此时,用户可以读取,但是不能修改,这正是我们想要的.

这里还是返回了指向对象内部成分的handles,这有时 会导致dangling handles.


    class GUIObject{ ...};
    const Rectangle boundingBox(const GUIObject& obj);

    GUIObject* pgo;
    const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());

上面的代码有什么问题呢?

boundingBox返回一个新的临时Rectangle对象,该对象没有名称,权且称为temp.随后在temp上调用upperLeft,返回一个指向temp内部成分的reference.然后pUpperLeft指向这个内部对象(也就是一个Point).

接下来呢?这条语句结束后,temp就被销毁了,此时pUpperLeft指向了一个不再存在的对象!!

##条款29: 为异常安全而努力是值得的

    class PrettyMenu{
	public:
	    void changeBackground( istreams& imgSrc);

	private:
	    Mutex mutex;
	    Image* bgImage;
	    int imageChanges;
    };

    void PrettyMenu::changeBackground(istreams& imgSrc){
	lock(&mutex);
	delete bgImage;
	++imageChanges;
	bgImage = new Image(imgSrc);
	unlock(&mutex);
    }

从异常安全性来看,上面的代码很糟.异常安全有两个条件,该函数一个也不满足.

    当异常被抛出时,具有异常安全性的函数会:

    **不泄露任何资源**
    **不允许数据破坏**

上面的代码在new Image(imgSrc)抛出异常时,对unlock的调用就绝不会执行,因为互斥器得不到释放.不尽如此,成员变量bgImage和imageChanges都已经被修改了.

我们可以使用之前提到的Lock类确保互斥器及时释放

    void PrettyMenu::changeBackground(istreams& imgSrc){
	Lock m1(&mutex);
	delete bgImage;
	++imageChanges;
	bgImage = new Image(imgSrc);
    }

现在来解决数据破坏

异常安全函数(Exception-safe functions)提供3个保证:

###基本承诺

    异常被抛出时没有任何对象或数据结构遭到破坏,所有对象都处于一种内部前后一致的状态.

###强烈保证

    如果函数成功,就完全成功,如果函数失败,程序会回到调用函数之前的状态.

###不抛异常(nothrow)保证
    绝不会抛出异常.
    作用域内置类型的所有操作都提供nothrow保证.

Exception-safe code 必须提供上述三种保证之一.否则就不具备异常安全性.
如果可能我们应提供nothrow保证,但对大多数函数而言,抉择往往落在基本保证和强烈保证之间.

对于changeBackground函数,我们可以改变两点以提供异常安全性:
1.把bgImage改为智能指针类型
2.改变语句次序,不要为了某件事情发生而改变对象状态,除非它真的发生了.

    class PrettyMenu{
	public:
	    void changeBackground( istreams& imgSrc);
	private:
	    tr1::shared_ptr<Image> bgImage;
    };

    void PrettyMenu::changeBackground(istreams& imgSrc){
	Lock m1(&mutex);
	bgImage.reset(new Image(imgSrc) );
	++imageChanges;
    }
这里不再需要手动删除旧图像,因为智能指针负责处理它.另外,删除只发生在新图像成功创建之后.更确切地说,tr1::shared_ptr::reset函数只有在参数被成功生成后才会被调用.delete只在reset函数内使用,所以如果从未进入哪个函数也就绝对不会使用delete.

如果Image构造函数出了异常,这个问题没有解决,changeBackground函数就只提供了基本的异常安全保证.
如何提供强烈保证呢?
有个一般会的设计策略很典型的会导致强烈保证,那就是**copy and swap**.

> 为你打算修改的对象(原件)做出一份副本,然后在副本上做一切修改.若有任何动作抛出异常,原对象仍保持不变.待所有改变都成功后,再将修改过的那个副本和原对象在一个不抛出异常的操作中置换.

实现上通常将所有隶属对象的数据从原对象放进另一个对象,然后赋予原对象一个指针,指向那个所谓的实现对象(即副本).这种手法常被称为pimpl idiom(pimpl是pointer to implementation的缩写),典型写法如下:

    struct PMImpl{
	tr1::shared_ptr<Image> bgImage;
	int imageChanges;
    };
    class PrettyMenu{
	...
	private:
	    Mutex mutex;
	    tr1::shared_ptr<PMImpl> pImpl;
    };
    void PrettyMenu::changeBackground(istreams& imgSrc){
	using std::swap;
	Lock m1(&mutex);
	tr1::shared_ptr<PMImpl> pNew( new PMImpl(*pImpl));
	pNew->bgImage.reset(new Image(imgSrc) );
	++pNew->imageChanges;
	swap(pImpl,pNew);
    }

copy and swap 策略是对对象状态做出全有或全无改变的一个很好办法,但是它并不保证整个函数有强烈的异常安全性.解释如下:

假设有一个函数someFunc,它使用copy-and-swap策略,但函数包含两个函数调用:

    void someFunc(){
	...	
	f1();
	f2();
	...
    }

显然,如果f1或f2的异常安全性比强烈保证低,那么someFunc也就无法提供强烈保证.即使f1和f2都提供强烈保证,someFunc也不见得就有强烈保证.如果f1成功执行,但是f2出现异常,那么程序状态和someFunc调用之前有所不同,即使f2没有改变任何东西.

这些问题都会阻止你提供强烈保证,还有一个原因是效率.copy-and-swap显然效率低下.所以强烈保证有时不切实际,因为实现成本巨大,此时你应该提供基本保证.


##条款30: 透彻了解inlining的里里外外

inline函数是函数但是又没有函数调用的开销.编译器最优化机制被设计用来浓缩那些不含函数调用的代码,因此编译器有能力对inline函数执行语境相关最优化.

inline函数背后的整体观念是:将对该函数的每一个调用都以函数本体替换.这样会增加目标文件的大小.这种代码膨胀可能导致额外的换页行为(paging),降低指令cache的命中率,以及伴随而来的效率损失.

但是如果inline函数本来就很小,那么编译器对此生成的代码可能比通过函数调用产生的代码更小,此时inline一个函数确实可能产生更小的目标代码和更高的指令cache命中率.

inline一个函数有两种方式:隐式和显式.
隐式就是直接定义在class中,显式是在定义函数时加上inline关键字.

但是记住,inline只是一个申请,编译器可以拒绝将一个太过复杂的函数inlining,而所有对virtual函数的调用(除非是最平淡无奇的??)也都会使inlining落空.因为virtual意味着"直到运行期才确定调用哪个函数",而inline意味着"执行前先将调用动作替换为被调函数的代码".

编译器通常不会为"通过函数指针而进行的调用"实施inling,因为**inling该函数之后,你去哪得到这个函数的地址呢.** 也就是说,对inline函数的调用有可能被inlined,也可能不被inlined,取决于调用方式:

    inline void f() { ... }
    void (*pf) () = f;
    f();//会被inlined
    pf(); 不会被inlined

构造函数和析构函数通常是inlining的糟糕候选人.考虑以下代码:

    class Base{
	public:
	    ...
	private:
	    string bm1, bm2;
    };

    class Derived: public Base{
	public:
	    Derived() {}
	    ...
	private:
	    string dm1, dm2, dm3;
    };

这个构造函数看起来什么都没有,但是编译器却可能将其扩展成这样:

    Derived::Derived(){
	Base::Base();

	try{ dm1.string::string(); }
	catch(...){
	    Base::~Base();
	    throw;
	}
	try{ dm2.string::string(); }
	catch(...){
	    dm1.string::~string();
	    Base::~Base();
	    throw;
	}
	try{ dm1.string::string(); }
	catch(...){
	    dm1.string::~string();
	    dm2.string::~string();
	    Base::~Base();
	    throw;
	}
    }

编译器真正产生的代码可能笔者还要复杂,但是这已经能说明问题,Derived构造函数至少一定会陆续调用成员变量和base class的构造函数,而这些调用会影响编译器是否对此空白函数inling.

相同理由使用于Base构造函数和string构造函数,因为它们被inlined,那么这些代码都会插入到Derived构造函数中.Derived析构函数也适用.

而且inline函数无法随着程序库升级而升级.就是说如果f是库中的一个inline函数,客户将其编进程序中,一旦库作者改变f,那么所有用到f的客户端程序都必须重新编译.而如果f是non-inline函数,一旦它有修改,客户端只需重新连接,远比重新编译代价小.如果程序库采用动态链接,那么升级库甚至不知不觉地被应用程序吸纳.

还有一个事实:大部分调试器都对inline函数束手无策,毕竟无法在不存在的函数中设置断点.

##条款31: 将文件间的编译依存关系降至最低



