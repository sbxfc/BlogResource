---
layout: post
title: "C/C++ cstring头文件"
date: 2014-11-16 20:04:24 +0800
comments: true
categories: 
---

char\* strcpy (char \*s1, const char \*s2);
---

---
将字符串s2复制到字符数组s1中去。<br>字符数组s1的长度应不小于"字符串"s2的长度,"字符串"s2 可以为字符数组名,也可以是一个字符串常量。<br>

	char s1[15];
    char s2[50] = "John Lennon";//字符串长度为12
    strcpy(s1, s2);
    printf(s1);
    
输出:*John Lennon* 
	
	char s1[15];
    const char* s2 = "John Lennon";//字符串长度为12
    strcpy(s1, s2);
    printf(s1);
    
输出:*John Lennon* 

<br>
char\* strncpy (char \*s1, const char \*s2, size_t len);
---

---
将s2的前len个字符复制到s1中指定的地址, 不加 '\0'

	char s1[12] = "";
    char s2[50] = "John Lennon";
    strncpy(s1, s2,4);
    printf(s1);
    
输出:*John* 

<br>
三.void\* memcpy (void \*dest, const void \*src, size_t length);
---

---
将src的前length个字节复制到dest中指定的地址, 不加 '\0'。dest和src不能是同一块内存区域。

	char dest[12] = "";
    char src[50] = "Goodman";
    memcpy(dest, src);
    printf(s1);
    
输出:*Good* 

<br>
void\* memmove (void \*s1, const void \*s2, size_t len);
---

---
当源单元和目的单元缓冲区交迭时使用。源和目的可以是同一块内存区域(例如数组某个元素在数组存储器内部移动数据)

<br>
char\* strcat (char \*s1, const char \*s2);
---

---
把字符串 2 接到字符串 1 后面（字符串 1 要足够大）。连接前两个字符串都有 "/0" ,连接时将字符串1后 "/0" 丢弃,只在新字符串后保留 '/0'

	char s1[50] = "John";
    char s2[50] = "Lennon";
    strcat(s1," ");
    strcat(s1,s2);
    
    printf(s1);
    
输出:*John Lennon* 

<br>
int strcmp (const char \*s1, const char \*s2);
---

---
两个字符串自左至右逐个字符相比(按 ASCII 码值大小比较)直到出现不同的字符或者遇到 "/0" 为止,如果全部字符相同,则认为相等,若出现不同字符,则以第一个不相同的字符为准

	const char* s1 = "John Lennon";
    const char* s2 = "John Smith";
    int len = strcmp(s1, s2);
    printf("%d",len);//输出:-7
    
- 如果字符串 1=字符串 2，函数返回值为 0
- 如果字符串 1>字符串 2，函数返回值为正数
- 如果字符串1<字符串 2，函数返回值为负数

<br>
int strncmp (const char \*s1, const char \*s2, size_t len);
---

---
对 s1 和 s2 的前len个字符作比较

<br>
int memcmp (const void \*s1, const void \*s2, size_t len);
---

---
对 s1 和 s2 的前 len 个字节作比较

<br>
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