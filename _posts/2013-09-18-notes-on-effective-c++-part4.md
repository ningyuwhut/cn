---
layout: post 
title: effective c++ 笔记(4)
categories:
- C++
tags:
- reading notes
---

## 条款21: 必须返回对象时,别妄想返回其reference

书中以operator*为例,实现了几个函数返回对象的方法
    
    class Rational{
	public:
	    Rational(int numerator = 0, int denominator = 1);

	    int numerator() const;
	    int denominator() const;
	private:
	    int n,d;
	    friend const Rational operator*(const Rational&lhs,
					    const Rational&rhs);
    };

operator* 可以实现成这样:

    const Rational& operator*(const Rational& lhs, const Rational& rhs){
	Rational result( lhs.n*rhs.n, lhs.d*rhs.d);
	return result;
    }

这显然是错误的,函数返回时result已经不存在了.

    const Rational& operator*(const Rational& lhs, const Rational& rhs){
	Rational* result = new Rational( lhs.n*rhs.n, lhs.d*rhs.d);
	return *result;
    }

这种方法也是有问题的,比如:

    Rational w,x,y,z;
    w = x*y*z;

最后一句连乘生成的对象有一个是获得不到的,这时肯定会有泄露
还有最后一种:

    const Rational& operator*(const Rational& lhs, const Rational& rhs){
	static Rational result ;
	result = ...;
	return *result;
    }

假设有以下代码:

    bool operator==(const Rational&rhs, const Rational&rhs);
    Rational a,b,c,d;

    if( (a*b) ==(c*d)){
	...
    }else{
	...
    }

这时会有什么问题呢?(a*b)==(c*d)总是true,因为它们使用的是同一个对象!!

正确的做法是什么呢?既然想返回对象,那就返回对象就可以了呗.

inline const Rational operator*(const Rational& lhs, const Rational& rhs){
    return Rational(lhs.n*rhs.n, lhs.d*rhs.d);
}

记住:
    绝不要返回pointer或reference指向一个local stack对象,或返回reference指向一个heap-allocated对象,或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象
    
##条款23: 宁以non-member,non-friend替换member函数

一个例子:

    class WebBrowser{
	public:
	    void clearCache();
	    void clearHistory();
	    void removeCookies();
    };
    假设又有一个执行上面全部功能的函数

    class WebBrowser{
	public:
	    void clearEverything();//调用上面三个函数
    };
    我们还有另外一个选择:non-member函数
    void clearEverything(WebBrowser& wb)
    {
	wb.clearCache();
	wb.clearHistory();
	wb.removeCookies();
    }

这两种实现哪一种更好呢?即此时在member函数和non-member函数间选择哪一个呢?

答案是选择封装性更大的那一个!但是哪个是封装性更大呢?non-member!为什么?因为member函数可以访问class的private成员.

> 如果你要在member函数和一个non-member,non-friend函数之间选择,而且两者提供相同功能,那么,导致更大封装性的是non-member,non-friend函数,因为它并不增加能够访问class内的private成分的函数数量

##条款24: 若所有参数皆需类型转换,请为此采用non-member函数

还是条款21(上面)的Rational的operator*为例,假设你打算想让以下语句通过:

    result = oneHalf*2;
    result = 2*oneHalf;//result和oneHalf都是Rational对象.

第2句是不能通过编译的,因为

    result = oneHalf.operator*(2);
    result = 2.operator*(oneHalf);
编译器还试图查找这样的函数:
    
    result = operator*(2, oneHalf);

但是显然又没有.
第一个调用能成功的原因是发生了隐式类型转换,like this:

    const Rational temp(2);
    result = oneHalf.temp;

隐式类型转换可行的原因在于Rational的构造函数不是explicit的!

这时可行的方法是运行编译器在每个实参身上运行隐式类型转换,怎么办呢?
non-member函数

    const Rational& operator*(const Rational& lhs, const Rational& rhs){
	return Rational(lhs.numerator()*rhs.numerator(),
			 lhs.denominator()*rhs.denominator());
    }

此时,上面的代码可以编译通过了.

##条款25:考虑写出一个不抛异常的swap函数


