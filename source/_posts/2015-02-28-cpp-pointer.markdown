---
layout: post
title: "C/C++ 认识指针"
date: 2015-02-28 13:47:25 +0800
comments: true
categories: 
---

#管理内存

C/C++语言和其他语言相比最大的区别是需要自己管理内存,但也不是所有内存都需要开发者自己手动去管理。应当了解一下C/C++程序编译后的内存。

编译后的程序内存主要分为几部分:

1）栈区(stack) : 用于存放函数的参数和局部变量的值等。这一部分内存,在程序运行中由编译器自动分配,并且当程序结束时由编译器自动释放,不需要开发者参与管理。例如,在下面的代码中,val值是函数func里的一个局部变量,在函数结束后val自动释放:

	void func(){
		int val;
	}

2）堆区(heap) :这一部分内存开发者自己主动分配的,需要开发者手动去释放。在C/C++程序中,我们主要用到两种主动申请内存的函数 <font color="#bd260d">**malloc**</font>和<font color="#bd260d">**new**</font>,前者来源于C,后者是C++特有的。

在myMallow函数里,指针pVal主动创建了一块int类型大小的内存,因为该内存创建于堆区,编译器不会帮我们释放掉,需要我们自己手动释放。

	void myMallow(){
		int* pVal = (int*)malloc(sizeof(int));
    	free(pVal);//释放内存
    }

3）全局区(静态区)(static),全局变量和静态变量存储于此,编译器在编译时就为这些变量分配内存。
	
	void foo(){
		static int val;//(val == 0),静态变量有默认值,动态变量没有,值是随机的。
	}

	
4）其他区 : 剩下的部分是文字常量区和程序代码区,前者存放常量字符串后者用于储存函数体的二进制代码。


#指针的使用

指针是C/C++里进行内存管理的途径。在C/C++里,所有变量、函数等,本质上是一个存在于内存区上的一块内存,并且每个内存都有一个地址。

对于一个int类型的变量val来说,其存在形式是一个大小为4个字节并且连续的内存。首字节的内存地址，即是该变量的内存地址。


	//指针pval的内存地址,并且记录下val类型
	int val = 17;
	int* p = &val;//&是取地址运算符

我们使用取地址运算符(&)来获取变量val的内存地址,对于指针变量p,如果我们想获取其指向的目标对象,我们可以用解引用运算符(*p)来得到它:

	int reVal = *p;  //使用*解引用,获取目标对象。

在程序中,如果指针暂时没有一个需要指向的目标对象,我们可以声明一个空指针:
	
	int* p;
	
但是这样声明是不安全的,最好为其初始化。未初始化的指针,可能包含一个垃圾值,其可能覆盖内存上的某个随机内存区域。可以用等于0来初始化指针,这个声明确保这个指针不指向任何实体。<font color='#bd260d'>**虽然符号NULL在标准库中也定义为0，它常常用于初始化空指针。但是，NULL仅同C兼容,在C++中最好使用0。**</font>
	
    int* p = 0;


#函数参数

在函数调用时,参数是按值传递的,即传入的参数只是这个参数值的副本。我们在函数体内,修改这个参数并不会对原参数对象造成影响。有时候,我们需要去修改原参数本身,这时候我们可以传入原参数的地址。

	void inc(int *p){
    	(*p)++;
	}
	
	int main()
	{
	    int a=3;
	    inc(&a);
	}

在main函数里,我们将变量a的地址传入inc函数时,在函数调用的一刹那发生了一次赋值操作。这时新建的指针p(形参)指向变量a

	int* p = &a;
	
#二级指针
	
在使用变量地址作为参数传入时,通过解引用可以获取原变量,并修改原变量的值。但是如果我们要修改引用变量的指针,用传入原变量地址的方式就不行了。

这时我们可以传入指针的地址,<font color='#bd260d'>**指针作为一个对象也是有地址的。**</font>
对一个普通变量,通过取地址运算符(&)获取该变量的地址。同样地,对一个指针使用取地址运算符(&)获取指针的地址。


	void myMalloc(int** p){
		//申请一块内存,并赋值给指针对象*p
	    *p = malloc(sizeof(int));
	}
	
	int main()
	{
	    int *p = 0;
    	myMalloc(&p);
	    return 0;
	}

#指针常量

用const修饰的变量为常量,常量的值不能被修改。当const符号修饰指针时,分为两种情况。

第一,当const修饰符在*左侧时,表示\*p部分为常量不能被修改:
	
	const int* pVal = &val;

第二,当const修饰符在*右侧时,pVal部分为常量不能被修改:

	int* const pVal = &val;

当\*pVal作为常量部分时:

    int val = 17;
    const int* pVal = &val;
    
    /**
    * val不受影响
    * 但常量*pVal不能被重新赋值
    */
    val++;		  
    //*pVal = 18;
    
    /**
    * 可以修改指针引用的对象
    */
    int num = 999;
    pVal = &num;

当pVal作为常量部分时:

    int val = 17;
    const int* pVal = &val;
    
    //val值不受影响
    val++;
    
    /**
    * 不可以修改指针引用的对象
    */
    int num = 999;
    //pVal = &num;
  
当指针指向常量时,因为引用对象为常量，所以这个对象值不能被改变:
	
	const int val = 17;
	const int* pval = &val;
	
	const char* pStr = "sbxfc";

#引用

引用实际上是一个常量指针。表达式 <font color='#bd260d'>**int &i = j;**</font>在编译时会被 <font color='#bd260d'>**int *const i = &j;**</font> 替换掉。所以,引用在初始化时必须为其赋值。

	int& ref = val;

#字符串数组

在C语言中,可以将字符串常量当做字符数组来使用。因为C语言中,系统会对字符串常量后面加一个'\0'来作为结束符。有了结束标志'\0'后，字符数组的长度就显得不那么重要了，在程序中往往依靠检测'\0'的位置来判定字符串是否结束，而不是根据数组的长度来决定字符串长度。当然，在定义字符数组时应估计实际字符串长度，保证数组长度始终大于字符串实际长度。（在实际字符串定义中，常常并不指定数组长度，如char str[]）

	char str[] = "Hello Sbxfc";
    printf("%zu \n",sizeof(str));//12字节
    
#参考:

More Effective C++

[维基百科-指针](http://zh.wikipedia.org/wiki/%E6%8C%87%E6%A8%99_(%E9%9B%BB%E8%85%A6%E7%A7%91%E5%AD%B8\))

[维基百科-Reference](http://en.wikipedia.org/wiki/Reference_%28computer_science%29)

[维基百科-Dereference operator](http://en.wikipedia.org/wiki/Dereference_operator)

[指针和引用的区别](http://wenku.baidu.com/link?url=aw5yRsRMPGXrTDsBoP4MNSi7iTZ94ogX-b0GSaOYvFx7XyWC7071EOg_ehZrw4LuqBvfhml5wfUy-TUCBm5QZXksYcMSrK30FsDpSarMUcO)

[c++之指针作为函数参数传递的问题](http://blog.csdn.net/fjb2080/article/details/5623427)

[重新认识二级指针](http://www.fenesky.com/blog/2014/07/03/pointers-to-pointers.html)
 
[深入分析C++引用](http://blog.csdn.net/webscaler/article/details/6577429)
 