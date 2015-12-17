---
layout: post
title: "C/C++ 结构体"
date: 2015-04-14 17:35:22 +0800
comments: true
categories: 
---

结构体的声明规则,struct为结构体关键字,tag为结构体标志,member-list为结构体成员列表,variable-list为此结构体声明的变量:

	struct tag { 
		member-list 
	} variable-list 

在一般情况下，tag、member-list、variable-list这3部分至少要出现2个。以下为示例：

	//此声明声明了拥有3个成员的结构体，分别为整型的a，字符型的b和双精度的c
	//同时又声明了结构体变量s1
	//这个结构体并没有标明其标签
	struct 
	{
	    int a;
	    char b;
	    double c;
	} s1;
	 
	//此声明声明了拥有3个成员的结构体，分别为整型的a，字符型的b和双精度的c
	//结构体的标签被命名为SIMPLE,没有声明变量
	struct SIMPLE
	{
	    int a;
	    char b;
	    double c;
	};
	//用SIMPLE标签的结构体，另外声明了变量t1、t2、t3
	struct SIMPLE t1, t2[20], *t3;
	 
	//也可以用typedef创建新类型
	typedef struct
	{
	    int a;
	    char b;
	    double c; 
	} Simple2;
	//现在可以用Simple2作为类型声明新的结构体变量
	Simple2 u1, u2[20], *u3;

结构体的成员可以包含其他结构体，也可以包含指向自己结构体类型的指针，而通常这种指针的应用是为了实现一些更高级的数据结构如链表和树等。
	
	//此结构体的声明包含了其他的结构体
	struct COMPLEX
	{
	    char string[100];
	    struct SIMPLE a;
	};
	 
	//此结构体的声明包含了指向自己类型的指针
	struct NODE
	{
	    char string[100];
	    struct NODE *next_node;
	};

如果两个结构体互相包含，则需要对其中一个结构体进行不完整声明，如下所示：

	struct B;    //对结构体B进行不完整声明
 
	//结构体A中包含指向结构体B的指针
	struct A
	{
	    struct B *partner;
	    //other members;
	};
	 
	//结构体B中包含指向结构体A的指针，在A声明完后，B也随之进行声明
	struct B
	{
	    struct A *partner;
	    //other members;
	};

#结构体成员访问
	
结构体成员依据结构体变量类型的不同，一般有2种访问方式，一种为直接访问，一种为间接访问。直接访问应用于普通的结构体变量，间接访问应用于指向结构体变量的指针。直接访问使用结构体变量名.成员名，间接访问使用(*结构体指针名).成员名或者使用结构体指针名->成员名。相同的成员名称依靠不同的变量前缀区分。
		
		struct SIMPLE
		{
		    int a;
		    char b;
		};
		 
		//声明结构体变量s1和指向结构体变量的指针s2
		struct SIMPLE s1, *s2;
		 
		//给变量s1和s2的成员赋值,注意s1.a和s2->a并不是同一成员
		s1.a = 5;
		s1.b = 6;
		s2->a = 3;
		s2->b = 4;

#结构体变量存储

在内存里,编译器根据成员变量在结构体里的排列顺序为其分配内存。第一个成员变量的内存地址相对结构体内存地址的偏移量为0,若单个成员变量所占内存小于对齐参数时,会空出额外内存单元。

在下图中,第一个成员变量a占一个内存单元（1字节）,由于最小对齐参数为8,a后面有7个内存单元被空出:

![](/images/2015/4/struct.png)


在结构体里,对齐数取结构体里最大的内存变量和预设对齐数之间的最小值,结构体的总大小为最大对齐数的整数倍。

使用关键字sizeof可以获取结构体所占内存,使用offsetof宏可以查看结构体的某个特定成员在结构体的位置:

	struct SIMPLE
	{
		char a;
		double b;
	};
	
	//获得SIMPLE类型结构体所占内存大小
    int size_simple = sizeof( struct SIMPLE );
    
    //获得成员b相对于SIMPLE储存地址的偏移量
    int offset_b = offsetof( struct SIMPLE, b );

使用代码修改结构体的预设对齐数:

	#pragma pack(8)
	struct SIMPLE
	{
	    int a;
	    char b;
	};
	#pragma pack()

---

<font color='#bd260d'>*VS可以在设置里修改预设对齐数,Xcode不能,而且Xcode下使用宏修改预设对齐数无效。<br>ps. Xcode里的可以按照预设对齐数为1来计算<br>难道是由于编译器用的是apple家的LLVM 6.0的关系？ 0.0*</font>

---

#参见 


结构体 (C语言):<br><http://zh.wikipedia.org/zh-cn/%E7%BB%93%E6%9E%84%E4%BD%93_(C%E8%AF%AD%E8%A8%80)>