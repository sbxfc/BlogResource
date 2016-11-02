---
layout: post
title: "GNU C 中的 __attribute__ 机制"
date: 2016-09-21 18:31:09 +0800
comments: true
categories: 
---

#\__attribute__ 机制


\_\_attribute\_\_ 是 GCC 提供的一种语法,可以帮助我们在编译时对声明的函数、变量和类型做一些特殊处理或者是检查操作。

\_\_attribute\_\_ 的语法格式为: <font color='#bd260d'>**\_\_attribute\_\_ ((attribute-list))**</font> , attribute-list 是指令集 , \_\_attribute\_\_ 出现在函数、变量和类型声明的 <font color='#bd260d'>**";"**</font> 前。

\_\_attribute\_\_ 有三类,分别为函数属性(Function Attribute) 、变量属性(Variable  Attribute) 和类型属性(Type  Attribute)

#一,函数属性

<font color='#bd260d'>**1. format (archetype, string-index, first-to-check)**</font> 

format 属性通过指定 *printf, scanf, strftime* 或 *strfmon* 等方法来检测函数的参数是否同样适用于这些指定的格式化字符串方法,如果不适用,编译器在编译时的就会发出警告,从而发现错误。

	extern int
	my_printf (int value, const char *my_format, ...)
	__attribute__ ((format (printf, 2, 3)));
	
	void foo()
	{
	    my_printf(0, "age = %d\n",17);
	    my_printf(0, "age = %d\n","17");
	    my_printf(0, "age = %d name = %s\n",17,"sbxfc");
	}

上面示例中, format 属性的第一个参数指定了一个 printf 方法,第二个参数 string-index  表示函数 my_printf 里格式化参数是总参数的第几个,这里我们的格式化参数 <font color='#bd260d'>**my_format**</font> 是第2个参数,format 属性的第三个参数表示,参数集合 (<font color='#bd260d'>**...**</font>) 从函数 my_printf 的第几个参数开始出现。

如无意外,上述示例在 gcc 编译时会提示以下警告信息:

	$ gcc -c main.c
	main.c:12:31: warning: format specifies type 'int' but the argument has type
      'char *' [-Wformat]
    my_printf(0, "age = %d\n","17");

去掉 \_\_attribute\_\_ 属性,该示例则不会提示错误,但运行时会出错。

<font color='#bd260d'>**2.  noreturn **</font> 

noreturn 属性表示其指定的函数没有返回值,当编译器执行到这时,要面对现实,不要大惊小怪(~慌忙报错~)。下面是 fatal 的部分代码,在程序出错的位置调用 fatal 函数打印信息,然后直接退出程序,不需要 return。

	void fatal () __attribute__ ((noreturn));
	
	void
	fatal (/* ... */)
	{
	  /* ... */ /* Print error message. */ /* ... */
	  exit (1);
	}

<font color='#bd260d'>**3.  deprecated **</font> 

deprecated 属性可以用来标识一个预计将会被弃用的函数,如果开发者使用该函数,编译时就会发出警告,并提示出错的行数。警告信息只会在开发者调用该函数时才会提示:

	int old_fn () __attribute__ ((deprecated));
	int old_fn ();
	int (*fn_ptr)() = old_fn;

在上面示例中,只会在第3行提出警告:

	main.c:7:19: warning: 'old_fn' is deprecated [-Wdeprecated-declarations]
	int (*fn_ptr)() = old_fn;
						^
	main.c:6:5: note: 'old_fn' has been explicitly marked deprecated here
	int old_fn ();

deprecated 也可以用于[变量](https://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Variable-Attributes.html#Variable-Attributes)和[类型](https://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Type-Attributes.html#Type-Attributes)。


<font color='#bd260d'>** 4. constructor & destructor**</font>
	
设置 constructor 属性可以使函数在 main 方法之前执行,而设置 destructor 可以使函数在 main 方法之后执行。 
	
	#include <stdio.h>
	#include <stdlib.h>
	
	__attribute__((constructor)) void before_func (){
	  printf("before \n");
	}
	
	__attribute__((destructor)) void after_func (){
	  printf("after \n");
	}
	
	int main(){
	  printf("main func \n");
	  return 0;
	}
	
不出意外,会看到以下输出:

	$ gcc main.c
	$ ./a.out 
	before 
	main func 
	after 	

constructor 、 destructor 函数也可以设置执行的优先级:

	__attribute__((constructor(PRIORITY)))
	__attribute__((destructor(PRIORITY)))

#二,变量属性

<font color='#bd260d'>** 1.  aligned (alignment) **</font>

aligned 属性让其指定的变量或结构体成员按 alignment 字节大小对齐。如果其中对齐长度有长度大于 alignment的,则按照最大对齐长度来对齐。

	#include <stdio.h>
	#include <stdlib.h>
	
	//结构体的对齐值为8
	struct foo {
	  char a;
	  int x[2] __attribute__ ((aligned (8)));
	};
	
	int main(){
	  int s0 = sizeof(struct foo);
	  printf("s0 = %d\n",s0);//print s0 = 16
	  return 0;
	}



<font color='#bd260d'>** 2.  packed **</font>

packed 属性用于设置变量或结构体成员以最小的对齐方式对齐。

在下面的结构体中,由于 x 已经使用 packed 进行对齐,所以此时结构体以 a 的size来对齐:

	#include <stdio.h>
	#include <stdlib.h>
	
	struct foo{
	  char a;
	  int x[2] __attribute__ ((packed));
	};
	
	int main(){
	  int s0 = sizeof(struct foo);
	  printf("s0 = %d\n",s0);//print s0 = 9
	  return 0;
	}



#三,类型属性

<font color='#bd260d'>** 1. packed **</font> 

如果 packed 属性用在 struct 或 union 上,表示该结构的成员变量按照紧凑模式对齐,即以变量的实际占用字节对齐,不用编译器进行优化对齐。如果用在 enum 上,则表示使用最小的整数来存储枚举类型。
	
	#include <stdio.h>
	#include <stdlib.h>

	struct my_unpacked_struct{
	  char c;
	  int i;
	};
	
	struct __attribute__ ((__packed__)) my_packed_struct {
	   char c;
	   int  i;
	   struct my_unpacked_struct s;
	};
	
	int main(){
	  int s0 = sizeof(struct my_unpacked_struct);
	  int s1 = sizeof(struct my_packed_struct);
	  printf("s0 = %d,s1 = %d\n",s0,s1);//print s0 = 8,s1 = 13
	  return 0;
	}



#参见

- <https://gcc.gnu.org/onlinedocs/gcc/Attribute-Syntax.html>
- <https://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Function-Attributes.html>
- <https://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Variable-Attributes.html#Variable-Attributes>
- <https://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Type-Attributes.html#Type-Attributes>
- <http://blog.zhangjikai.com/2015/11/28/%E3%80%90C%E3%80%91alignment/>
- <http://www.jianshu.com/p/6153eccdbe62>
- <http://blog.wangruofeng007.com/blog/2016/01/13/attribute/>
- <http://unixwiz.net/techtips/gnu-c-attributes.html>