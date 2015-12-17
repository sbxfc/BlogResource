---
layout: post
title: "C/C++ 关键字"
date: 2014-11-18 12:17:10 +0800
comments: true
categories: 
---

#const

定义有类型常量,该常量在编译时分配内存。

#\#define
	
定义无类型常量,在预编译时进行替换,不分配内存。
	
#extern

extern置于变量或函数前，表示变量或函数定义在其他文件中。

例如,<font color='#bd260d'>a.cpp</font> 和 <font color='#bd260d'>b.cpp</font> 在同一项目中,a.cpp中定义了一个变量 <font color='#bd260d'>int x; </font> b.cpp想调用这个变量x,不需要包含a.cpp,直接使用extern int x;就可以在b.cpp中使用了。

按照C语言的方式进行函数编译

	extern "C" fun();

#static

静态变量和全局变量一样都储存在静态储存区,生命期一直到程序运行结束。

类里声明的static变量在类外是不可见的；函数内部的声明的static变量在函数外是不可见的。
	