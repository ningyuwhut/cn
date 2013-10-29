---
layout: post 
title: 深度探索c++对象模型
categories:
- c++
---

当类没有定义任何构造函数,那么编译器会自动合成一个trivial constructor.trivial意思是无用的,编译器实际上并不产生它们.但是在四种情况下,声明出来的constructor是nontrivial的.

1. 带有default constructor的member class object.

如果类中有个成员对象,该对象所属类具有default constructor,那么编译器合成的默认构造函数就是nontrivial的.注意,这个合成操作只在constructor真正被调用时才会发生.

    class Foo { public: Foo(), Foo(int) ... };
    class Bar { public: Foo foo; char* str; };

被合成的default constructor可能像这样:

    inline Bar::Bar(){
	foo.Foo::Foo();
    }
合成的constructor中并没有初始化Bar::str.

假设我们定义了一个Bar的default constructor:

    Bar::Bar() { str = 0; }

在这个constructor中没有初始化member object foo.此时,由于已经有了default constructor,编译器不再合成第二个,那怎么办呢?

编译器的做法是:如果class A内含一个或一个以上的member class object,那么class A的每一个constructor必须调用每一个member class的default constructor"就是说,编译器会在已存在的constructor中加入一些代码,调用member object的default constructor.

扩张后的constructor可能像这样:

    Bar::Bar(){
	foo.Foo::Foo();
	str = 0;
    }

如果有多个class member objects,c++语言要求以"member objects 在class中声明次序"来调用各自的constructor.

2. "带有default constructor"的base class

如果派生类没有任何constructor,而它又继承自一个带有default constructor的base class,那么这个派生类的default constructor会被视为nontrivial,并被合成出来,它将调用父类的default constructor.如果派生类提供了多个constructor,但是又没有default constructor,编译器会扩张现有的每一个constructor,将调用必要的default constructor的代码加进去.如果同时存在带有default constructor的member class objects,那些default constructor也会被调用,在所有base class constructor被调用之后.

3.带有virtual function的class

 1. class 声明一个virtual function
 2. class 派生自一个继承串链,其中有一个或更多的virtual base classes.

在以上两种情况中,如果没有自定义constructor,那么编译器会合成一个default constructor.

编译器会完成两个扩张操作:

1. 产生一个virtual function table(vtbl),存放class的virtual function地址.
2. 在每一个class object中,存放一个额外的pointer member(vptr),指向虚函数表.

为此,编译器为每个对象的vptr设置初值,存放virtual table地址.对于class定义的每个constructor,编译器会扩张它们.对于没有定义任何constructor的类,编译器会合成一个default constructor,以便正确初始化每个class object的vptr.

4. 带有一个virtual base class 的 class.

总结:

在合成的default constructor中,只有base class subobjects和member class object会被初始化,所有其他的nonstatic data member(整数,指针,整数数组)等等都不会被初始化.

