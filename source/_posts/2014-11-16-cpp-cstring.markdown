---
layout: post
title: "C/C++ 字符操作"
date: 2014-11-16 20:04:24 +0800
comments: true
categories: 
---
#strcpy

将字符串 source 复制到字符数组 destination 中去。<br>
destination的长度应不小于source的长度,source可以为字符数组名,也可以为字符串常量:
	
	/**
	 * strcpy example 
	 * char * strcpy ( char * destination, const char * source );
	*/
	#include <stdio.h>
	#include <string.h>
	
	int main ()
	{
	  char str1[]="Sample string";
	  char str2[40];
	  char str3[40];
	  strcpy (str2,str1);
	  strcpy (str3,"copy successful");
	  printf ("str1: %s\nstr2: %s\nstr3: %s\n",str1,str2,str3);
	  return 0;
	}

#strncpy

将 source 的前 num 个字符复制到destination中指定的地址, 不加 '\0'

	/* 
	 *	strncpy example 
	 *  char * strncpy ( char * destination, const char * source, size_t num );
	*/
	#include <stdio.h>
	#include <string.h>
	
	int main ()
	{
	  	char s1[12] = "";
    	char s2[50] = "John Lennon";
    	strncpy(s1,s2,4);
    	printf(s1);/*输出:John */
	
	  return 0;
	}
	
- <http://www.cplusplus.com/reference/cstring/strncpy/>

#memcpy

c和c++使用的内存拷贝函数，memcpy函数的功能是从源src所指的内存地址的起始位置开始拷贝n个字节到目标dest所指的内存地址的起始位置中。

	/**
	 *	memcpy example 
	 *	void * memcpy ( void * destination, const void * source, size_t num );
	*/
	
	#include <stdio.h>
	#include <string.h>
	
	struct {
	  char name[40];
	  int age;
	} person, person_copy;
	
	int main ()
	{
	  char myname[] = "Pierre de Fermat";
	
	  /* using memcpy to copy string: */
	  memcpy ( person.name, myname, strlen(myname)+1 );
	  person.age = 46;
	
	  /* using memcpy to copy structure: */
	  memcpy ( &person_copy, &person, sizeof(person) );
	
	  printf ("person_copy: %s, %d \n", person_copy.name, person_copy.age );
	
	  return 0;
	}
    
#memmove

当源单元和目的单元缓冲区交迭时使用。源和目的可以是同一块内存区域(例如数组某个元素在数组存储器内部移动数据)

	void* memmove (void *s1, const void *s2, size_t len);



#strcat

把字符串 2 接到字符串 1 后面（字符串 1 要足够大）。连接前两个字符串都有 "/0" ,连接时将字符串1后 "/0" 丢弃,只在新字符串后保留 '/0'

	/*
	char* strcat (char *s1, const char *s2);
	*/

	char s1[50] = "John";
    char s2[50] = "Lennon";
    strcat(s1," ");
    strcat(s1,s2);
    
    printf(s1);/*输出:John Lennon*/
    

#strcmp

两个字符串自左至右逐个字符相比(按 ASCII 码值大小比较)直到出现不同的字符或者遇到 "/0" 为止,如果全部字符相同,则认为相等,若出现不同字符,则以第一个不相同的字符为准

	/*
	int strcmp (const char *s1, const char *s2);
	*/

	const char* s1 = "John Lennon";
    const char* s2 = "John Smith";
    int len = strcmp(s1, s2);
    printf("%d",len);//输出:-7
    
- 如果字符串 1=字符串 2，函数返回值为 0
- 如果字符串 1>字符串 2，函数返回值为正数
- 如果字符串1<字符串 2，函数返回值为负数

#strncmp

对 s1 和 s2 的前len个字符作比较

	int strncmp (const char *s1, const char *s2, size_t len);

#memcpy

	void* memcpy (void *buf1, const void *buf2, size_t length);

该函数用来比较buf1和buf2的前length个字节是否相同,该函数是按字节来比较的,如果返回0则表示相同。需要注意的是,buf1和buf2不能是同一块内存区域。

	#include<string.h>
	#include<stdio.h>
	void main()
	{
		char *s1 = "Hello,Programmers!";
		char *s2 = "Hello,Programmers!";
		int r = memcmp(s1,s2,strlen(s1));
		if(!r)
		    printf("s1 and s2 are identical\n");/*s1等于s2*/
		elseif(r<0)
		    printf("s1 is less than s2\n");/*s1小于s2*/
		else
		    printf("s1 is greater than s2\n");/*s1大于s2*/
		return 0;
	}


输出结果:
	
	s1 and s2 are identical
	
size_t strlen (const char \*s);
---

---
它是测试字符串长度的函数，函数的值为字符串中的实际长度(不包括 "/0")

#memset

	void memset (void* s, int val, size_t len);

将从 s 开始len个字节置设置为val.

	char buffer[] = "Hello world\n";
    printf("Buffer before memset: %s\n", buffer);
    memset(buffer, '*', strlen(buffer) );
    printf("Buffer after memset: %s\n", buffer);
	
输出:*Buffer before memset: Hello world*<br>
*Buffer after memset: ************* 

<br>
char\* strerror (int errno);
---

---
返回指向错误信息字符串的指针

char \* fgets( char \*str, int count, FILE \*stream );
---
----

从给定的文件流中读取不超过(count - 1)个字符。<br>若读取的是本行最后一个字符时在该串字符后加上换行符，并把它们存储在str中。


	FILE* fp = fopen("1.txt","rt");
    if(fp == NULL){
        printf("文件不存在!");
        return false;
    }
    
    std::vector<std::string> fLines;
    char sz[255];
    
    while( fgets(sz,255,fp) ){
        fLines.push_back( sz );
    }
    
    const char** codes = new const char*[fLines.size()];
    
    for(int i=0;i<int(fLines.size());i++){
        codes[i] = fLines[i].c_str();
    }

    //delete[] codes;
    
#参见 

- <http://www.cplusplus.com/reference/cstring/memcpy/>