---
layout: post 
title: effective c++ 笔记
categories:
- C++
tags:
- reading notes
---

## 条款2:尽量以const,enum,inline替换 #define

宏是在预处理器中处理的,而其他都是在编译器中处理的,也就是宏不会进入符号表.
如果出现编译错误,可能不利于调试.解决办法是用常量代替.

class专属常量

    class GamePlayer{
	private:
	    static const int NumTurns = 5;
	    int scores[NumTurns];
	    ...
    };

上面的代码中是NumTurns的声明式而非定义式.

> 通常c++要求你对所使用的任何东西提供定义式,但是如果是class专属常量又是static且为integral type(ints,chars,bools),则可以特殊处理.
只要不取它们的地址,那么我们可以声明并使用它们而无须提供定义式.但如果取某个class专属常量的地址,则必须提供定义式

    const int GamePlayer::NumTurns;//放到实现文件中

上述的"in-class初值设定"只允许对整数常量进行.但是如果编译器不允许,还有下述的"enum hack".

    class GamePlayer{

	private:
	    enum {NumTurns = 5 };
	    int scores[NumTurns];
	    ...
    };

**形似函数的宏最好改用inline函数替换**

## 条款3: 尽可能使用const

STL迭代器以指针为根据塑模出来,所以迭代器的作用就像个T*指针.

**声明迭代器为const就像声明指针为const一样(T* const),此时迭代器不能指向其他对象,但可以通过迭代器改变它所指的对象.**
如果希望迭代器所指的东西不可被修改,那么我们需要使用const_iterator.
这其实就是T* const 和const T*的区别.

对于函数返回的const类型对象,我们是无法修改的.

    class TextBlock{
	public:
	    ...
	    const char& operator[](size_t position) const{
		return text[position];
	    }

	private:
	    string text;

    };

    TextBlock tb("Hello");
    const TextBlock ctb("World");
    cout << ctb[0];
    ctb[0] = 'x'; //错误,此时调用的是const TextBlock::operator[],返回的是const 对象,无法修改

另外,当non-const operator[] 返回 char,而不是char&时,下述语句同样无法编译:

   tb[0] = "x"

   因为
   > 如果函数返回类型为内置类型,那么改动函数返回值从来就不合法

对于const成员函数有两种观点:bitwise constness和logical constness.
前者认为const成员函数不应该改变对象的任何一个bit,而后者认为const成员函数可以修改对象的某些bit,但只有在客户端侦测不出的情况下才合法.

当我们需要修改const对象但是编译器又坚持bitwise constness时该怎么办呢?

办法就是使用 mutable.
**mutable 释放了non-static成员变量的bitwise constness约束.**即mutable即使在const成员函数内也可以修改

在const和non-const成员函数中避免重复

书中以operator[]为例,当const和non-const版本代码几乎相同时,为避免代码重复,我们令non-const版本调用const版本,代码如下:

    class TextBlock{
	public:
	    const char& operator[](size_t position) const{

		...
		return text[position];
	    }

	    char& operator[](size_t position) {
		return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);
	    }

	...
    };

在non-const中,包含两次转型,一是用static_const把*this从TextBlock&转化为const TextBlock&,二是使用const_cast把返回类型从const char&转化为char&

这里,令non-const调用const是可以的,但是令const调用non-const是不可以的.

## 条款4: 确保对象被使用前已先被初始化

在c part of c++中,初始化可能会有运行时成本,所以并不保证初始化,但是在non-C parts of C++中规则略有不同.
这也就是为什么array不保证内容被初始化,但是vector却有此保证

此时,**最佳办法是:永远在使用对象前将其初始化**

内置类型你就手动初始化,自定义类型就要依靠构造函数:确保每个构造函数把每个成员都初始化

c++规定,**对象的成员变量的初始化发生在进入构造函数本体之前 **,所以在构造函数体中的操作是赋值而非初始化,这时应该使用成员初始化列表(member initialization lis).

为避免记住成员变量何时必须在成员初始化列中初始化,何时不需要,最简单的做法是:总是使用成员初始化列表.

C++成员初始化次序:base-class->derived class,且按照声明次序被初始化

不同编译单元内定义的non-local static对象

non-local static 对象是只除了在函数中定义的static对象(称为local static)之外的其他所有static对象
所谓编译单元,指的是产生单一目标文件的源码,基本上是源文件加上其包含的头文件

此时的情形是多个源文件中每个都至少包含一个non-local static对象,而且某个编译单元中的non-local static使用另外一个单元的non-local static进行初始化,而它用到的那个对象可能尚未初始化,因为C++对**"定义于不同编译单元的non-local static对象的初始化次序无明确定义"**

    class FileSystem{
	public :
	...
	    size_t numDisks() const;
	    ...
    };
    extern FileSystem tfs;

    class Directory{
	public:
	    Directory(params);
	    ...
    };

    Directory::Directory(params){
	size_t disks = tfs.numDisks();
	...
    }

    Directory tempDir(params);

此时,除非tfs在tempDir之前初始化,否则tempDir的构造函数会用到尚未初始化的tfs

解决该问题的办法是将non-local static放进函数并且声明为static,函数返回一个指向它的reference.

即用local static代替 non-local static.说这是Singleton模式,没看过,跳过先.

原因在于:

> c++保证,函数内的local static对象会在"该函数被调用期间""首次遇上该对象之定义式"时被初始化

前面的程序变为

    class FileSystem{};//同前

    FileSystem& tfs{
	static FileSystem fs;
	return fs;
    }

    class Directory{};//同前

    Directory::Directory(params){
	size_t disks = tfs().numDisks();
	...
    }

    Directory tempDir(){
	static Directory td;
	return td;
    }

任何non-const static 对象(local+non-local)在多线程环境下"等待某事发生"都会有麻烦

体会一下
