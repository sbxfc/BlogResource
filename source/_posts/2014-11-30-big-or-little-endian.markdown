---
layout: post
title: "网络字节序(一)*"
date: 2014-11-30 16:00:56 +0800
comments: true
categories: 
---

对于单一的字节（a byte），大部分处理器以相同的顺序处理位元（bit），因此单字节的存放方法和传输方式一般相同。对于多字节数据，如整数（32位机中一般占4字节），在不同的处理器的存放方式主要有两种方式:

#大端序

大端序（英：big-endian）

大端序是指高位在内存的低地址上。在十进制中,我们通常说高位在左,在其他进制也是如此。

因此,一个十六位的短整型 0x3132 在一个大端设备上,字节在内存上的排列应该如此:
	
	地址增长方向  →
	地址	| 0x1000 | 0x1001 
	--------------------------- 
	字节 | 0x31 | 0x32 


0x31 是高位上的字节,在低地址0x1000上,0x32是低位字节在地址相对较高的0x1001上。

#小端序

小端序 (英：little-endian)，与大端序正好相反。

	地址增长方向  →
	地址	| 0x1000 | 0x1001 
	--------------------------- 
	字节 | 0x32 | 0x31

0x32 低位字节放在低地址0x1000上,0x31高位字节在高地址0x1001上。
	
#不同CPU的操作系统对应字节序

处理器 | 操作系统 | 字节排序
------------ | ------------- | ------------
Alpha | 全部   | Little endian
HP-PA Cell | NT  | Little endian
HP-PA | UNIX  | Big endian
Intelx86 | 全部  | Little endian
	
我们也可以自己通过代码来验证一下自己的设备是大端序还是小端序:

	unsigned int val = 0x12345678;
    printf("[0]:0x%x\n", *((char *)&val + 0));
    printf("[1]:0x%x\n", *((char *)&val + 1));
    printf("[2]:0x%x\n", *((char *)&val + 2));
    printf("[3]:0x%x\n", *((char *)&val + 3));
	
如果和我一样输出了下面的内容,说明设备是小端设备反之是大端设备。

	[0]:0x78
	[1]:0x56
	[2]:0x34
	[3]:0x12
	
	
#网络字节序

由于一个数值在不同设备上的字节序可能不同,当把这个数值转化为字节进行网络传输并被另一台设备接收时,就会遇到不知按照什么顺序将多字节转化为数值的问题。

幸好,前人为我们统一了口径。“网络传输采用大端序!“，所以,大端序也被称之为网络字节序。
	
#大小端检测

一,利用联合体来检测大小端。由于联合体 union 的存放顺序是所有成员都从低地址开始存放，利用该特性就可以轻松地获得了CPU对内存采用 little-endian 还是 big-endian 模式读写。
	
	/**
	 * return 1 : 小端
	 * return 0 : 大端
	*/
	int isLittleEndian()
	{
	    union {
	        unsigned int a;
	        unsigned char b;
	    }c;
	    c.a = 1;
	    return (c.b == 1);
	}
	
二，使用类型的强制转换实现

	/**
	 * return 1 : 小端
	 * return 0 : 大端
	 */
	int isLittleEndian()
	{
	    unsigned short flag = 0x4321;
	    if(*(unsigned char*)&flag == 0x21){
	        return 1;
	    }else{
	        return 0;
	    }
	}

	
#四,BSD Socket API 里定义的大小端转化


伯克利socket API定义了一组转换函数，用于16和32bit整数在网络序和本机字节序之间的转换。htonl，htons用于本机序转换到网络序；ntohl，ntohs用于网络序转换到本机序。






