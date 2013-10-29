---
layout: post 
title: 深度探索c++对象模型(2)
categories:
- c++
---

copy constructor的构建操作

三种情况会调用copy constructor

1.

    class X { ... };
    X x;
    X xx = x;
2.

    extern void foo(X x);

    void bar(){
	X xx;
	foo( xx );
    }

3. 

    X foo_bar(){
	X xx;
	return xx;
    }

如果class没有定义copy constructor,当需要时,编译器以所谓的default memberwise initialization 手法完成.就是把每一个内建或派生的data member从源对象拷贝过来.不过它并不拷贝member class object,而是以递归的方式对这个member class object实施memberwise initialization.

如果没有声明copy constructor,那么编译器会生成隐含的声明或定义.C++同样把copy constructor分为trivial和nontrivial的,只有nontrivial的才会被合成.决定一个copy constructor是否trivial的标准在于class是否展现出所谓的"bitwise copy semantics"

Bitwise Copy Semantics
