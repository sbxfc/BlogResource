---
layout: post
title: "C/C++  析构函数"
date: 2015-03-03 19:34:22 +0800
comments: true
categories: 
---

1. 为基类声明虚构析构函数
				
	当一个类作为基类(base class)被另一个类继承时(derived class),基类里的析构函数需要声明为virtual:
	
		virtual ~BaseClass();
	
	假若,基类中的析构函数被声明为non-virtual,而对象经由基类(base class)类型的指针指向继承类(derived class)时。<br>在删基类时,基类部分被销毁,而继承类成分没有被销毁,造成资源泄露的情况:
	
		/**
		 * 若BaseClass为声明virtual析构函数,在删除时会造成内存泄露
		*/
		BaseClass* base = new DerivedClass();
		...
	    delete base;
    
	为创建基类(base class)创建一个virtual析构函数,在删除基类(base class)指针时就会摧毁整个对象,包括继承类(derived class)成分。
  
	相同的道理,对于一些non-virtual析构函数的类。比如string 、vector、list、set等,这些标准容器。其设计的目的不是作为base classes使用,如果错误地继承会发生跟上面相同的错误。

	<!--more-->

		class SpecialString : public std:string{..};
	
2. 为基类声明纯虚构析构函数。


	为基类声明纯虚构(pure virtual)函数会使基类变为抽象(abstract)类,也就是不能被实例化的类。<br>有时,基类里没有能声明的pure virtual的函数。我们可以声明一个pure virtual析构函数,这时基类就会变成抽象类。

		class BaseClass{
		public:
		    BaseClass();
		    virtual ~BaseClass() = 0;
		};
		
		
		//new BaseClass 初始化会出错

---

实例:<https://github.com/sbxCPP/cppVirtual>