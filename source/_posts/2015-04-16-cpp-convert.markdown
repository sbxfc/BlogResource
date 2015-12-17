---
layout: post
title: "C/C++ 类型转化"
date: 2015-04-16 20:05:03 +0800
comments: true
categories: 
---


#<font color='#bd260d'>const_cast</font>

const_cast,移除对象的常量性,一般用于移除指针或引用的常量性,不适用于对象。const_cast去除常量性不是为了修改常量所指向的内容,而是为了使函数能接受实参	


**const_cast**作用于指针,转化后的指针pval和常量val的地址相同。但是操作\*pval不会影响常量val。<font color='#bd260d'>**\*pval修改的是一个临时对象**</font>

	const int val = 17;
    int* pval = const_cast<int*>(&val);
    *pval = 200;	
    std::cout << "*pval="<<*pval<<std::endl;//*pval = 200
    std::cout << "pval 地址="<<pval<<std::endl;	 
    std::cout << "&val 地址="<<&val<<std::endl;
    std::cout << "val="<<val<<std::endl;//常量val值不变

const_cast作用于引用,转化后的引用refval并未指向常量val,而是指向一个临时对象。此时,修改引用refval不会对val造成影响

	const int val = 17;
    int refval = const_cast<int&>(val);
    refval = 100;
    std::cout << "refval="<<refval<<std::endl;	//refval = 100
    std::cout << "val="<<val<<std::endl;		//val = 17
  
const_cast不能用于对象
  
  	const int val = 17;
    int num = const_cast<int>(val);//错误,不能通过编译
      
<!-- more -->
      
#<font color='#bd260d'>static_cast</font>
  

static_cast 运算符可用于将指向基类的指针转换为指向派生类的指针等操作。此类转换并非始终安全。

通常使用static_cast转换数值数据类型，例如将枚举型转换为整型或将整型转换为浮点型，而且你能确定参与转换的数据类型。static_cast 转换安全性不如 dynamic_cast 转换，因为 static_cast 不执行运行时类型检查，而 dynamic_cast 执行该检查。 对不明确的指针的 dynamic_cast 将失败，而 static_cast 的返回结果看似没有问题，这是危险的。 尽管 dynamic_cast 转换更加安全，但是 dynamic_cast 只适用于指针或引用，而且运行时类型检查也是一项开销。

	//转换数值数据类型
	int val = static_cast<int>(3.141592654);
    
    //将无类型指针转化为某一类型指针
    void* p = &val;
    int* pval = static_cast<int*>(p);
    
#<font color='#bd260d'>reinterpret_cast</font>


“通常为操作数的位模式提供较低层的重新解释”也就是说将数据以二进制存在形式的重新解释。

	int *ip;
	char *pc = reinterpret_cast<char*>(ip);
	// 需要记得pc所指向的真实对象是int型,并⾮字符串。
	// 如果将pc当作字符指针进⾏行操作,可能会造成运⾏行时错误
	// 如int len = strlen(pc);