---
layout: post
title: "C++ 模板（Template）"
date: 2014-11-18 17:58:41 +0800
comments: true
categories: 
---

最初,模板设计的目的是用来解决参数类型的问题。<br>即,对于一定范围的若干类型的对象都可以用一种结构来处理。

以取最大值函数为例,以下使用函数重载的方式对多个不同类型的相同功能函数进行重载

	int max(int x,int y){
		return(x>y)?x:y ;
	}
	
	float max(float x,float y){
		return (x>y)? x:y ;
	}
	
	double max(double x,double y){
		return (c>y)? x:y ;
	}
	
函数重载是一种静态的多态实现,在编译时就确定函数入口地址。

也可以使用一个函数模板实现上面的功能:

	template <class T>
	T max(T x,T y){
    	return (x>y)? x:y ;
	}
	
	
#写一个函数模板
	
C++模板分为<font color='#bd260d'>函数模板</font>和<font color='#bd260d'>类模板</font>。上面用到的是函数模板。

写一个函数模板:

一. 对于普通函数,数据类型采用具体的数据类型。仍以取最大值函数为例,定义这样一个普通的取最大值函数:

	int max(int x,int y){
		return(x>y)?x:y ;
	}

二.将数据类型参数化。将其中的数据类型名(int)全部替换成自己定义的抽象类型参数名(T)	

	T max(T x,T y){
		return(x>y)?x:y ;
	}

三.在函数头前面用关键字template引出对类型参数名的声明。这样就把一个具体的函数改造成一个通用的函数模板：

	template <class T>
	T max(T x,T y){
		return(x>y)?x:y ;
	}

#类模板

类模板的结构和函数模板类似。前缀<font color='#bd260d'>template \<class T></font>表明这里声明了一个模板,并且一个类型为T的参数类型将在模板的作用域中使用。

	template <class T>
	class ClassA
	{
	private:
	    T m;
	public:
	    ClassA(T t){
	        m = t;
	    }
	    
	    void show(){
	        printf("%d\n",m);
	    }
	};
	


模板可以像下面这样使用:
	
	ClassA<int> a(10);
	a.show();


#参考:

Effect.C++

The Design And Evolution Of C++

模板:<br><http://jpk.sdju.edu.cn/cplus/kejian/content/chapter9/chapter9_1_1.htm>

函数模板:<br><http://jpk.sdju.edu.cn/cplus/kejian/content/chapter9/chapter9_2_1.htm>

类模板:<br><http://jpk.sdju.edu.cn/cplus/kejian/content/chapter9/chapter9_3_1.htm>