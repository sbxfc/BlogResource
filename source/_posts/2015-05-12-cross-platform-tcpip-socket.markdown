---
layout: post
title: "跨平台的TCP通信客户端"
date: 2015-05-12 14:05:45 +0800
comments: true
categories: 
---

TCP/IP 是 TCP/IP协议族的简称,是互联网的基础通讯架构,事实上也是计算机网络通信的国际标准。TCP/IP的前身是NCP(网络控制协议),是S.克罗克及其小组在加州大学制定的最初的主机间通信协议。NCP协议的缺点是,它没有为每台电脑设置一个唯一地址,导致在越来越庞大的网络中难以准确定位。另一方面,NCP缺乏纠错功能,导致数据传输不稳定。

TCP/IP将网络通信的可靠性改为由主机保证而不是像NCP那样由网络保证。TCP/IP使用一个成为网关（后来改为路由器以免与网关混淆）的计算机为每个网络提供一个接口并且在它们之间来回传输数据包。

在1985年时,因特网架构理事会举行一个三天有250家厂商代表参加的关于计算产业使用TCP/IP的工作会议，帮助推广并且引领它日渐增长的商业应用。

如今,绝大多数商业操作系统都实现了TCP/IP协议,这其中包括所有的商业Unix、Linux发布包，以及Mac OS X和微软的Windows操作系统。 

#TCP/IP模型

两个因特网主机(Host)通过两个路由器（Router）和对象的层连接。

![](/images/2015/5/tcp_ip_stack_connections.png)

<!-- more -->

在TCP/IP模型是一个抽象的分层模型，这个模型中,所有的TCP/IP系列网络协议都被归类为4个抽象层,应用层、运输层、网络层、链路层。每一层建立在低一层的服务上,并为高一层提供服务。

- 数据链路层:在TCP/IP协议族中,链路层又称为网络接口层。通常包括操作系统中的设备驱动和计算机中的对应网络接口卡。它们一起处理与电缆的物理接口细节。

- [网络层](http://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%B1%82):网络层提供路由和寻址的功能，使两终端系统能够互连且决定最佳路径，并具有一定的拥塞控制和流量控制的能力。TCP/IP协议体系中的网络层功能由IP协议规定和实现，故又称IP层。

- 运输层:运输层主要为两台计算机上的应用程序提供端到端的通信。在TCP/IP协议族中,有两个不相同的传输协议:TCP和UDP。TCP为两台主机提供高可靠性的数据通信。它所做的工作包括把应用程序交给它的数据分成合适的小块交给下面的网络层,确认接收到的分组,设置发送最后确认分组的超时时钟 等。由于运输层提供了高可靠性的端到端的通信,因此应用层可以忽略所有这些细节。而另一方面,UDP则为应用层提供一种非常简单的服务。它只是把称做数据报的分组从一台主机发送到另一台主机,但并不保证该数据报能到达另一端。任何必须的可靠性由应用层来提供。

- 应用层:应用层是指我们的应用程序。

![](/images/2015/5/ip_stack_connections.png)

#POSIX

可移植操作系统接口（英语：Portable Operating System Interface of UNIX，缩写为POSIX），是IEEE为要在各种UNIX操作系统上运行软件，而定义API的一系列互相关联的标准的总称。

在OS和Linux在内的所有Unix系统里,我们都可以使用Posix接口来创建socket。在Windows系统里,我们可以使用WinSock来操作socket,WinSock实际上是对POSIX socket的一个封装。

#WinSock和POSIX socket接口的异同

一,头文件

在windows中需要引入头文件 

	#include <winsock.h>

在其他遵从POSIX规则的系统里需要引入

	#include <sys/types.h>
	#include <sys/socket.h>
	#include <netinet/in.h>
	
二,链接库

在windows下面需要主动连接ws2_32.lib库

	#ifdef WIN32
	#include "StdAfx.h"
	#pragma comment(lib, "wsock32")
	#endif

三,初始化

在windows下面需要初始化,windows下面是通过动态链接的方式调用socket的,所以需要你告诉调用的链接库的版本,操作系统会帮你加载并初始化dll。

	WORD wVersionRequested;
    WSADATA wsaData;
    int err;
    wVersionRequested = MAKEWORD(2, 0);
    err = WSAStartup(wVersionRequested, &wsaData);
    if(0 != err) //检查Socket初始化是否成功
    {
        cout<<"Socket2.0初始化失败，Exit!";
        return -1;
    }
    
四,函数

在Windows和POSIX里,socket的函数是一致的。只是windows里面宏定义了一个socket描述符。

#创建TCP套接字

一,创建一个TCP套接字

该函数返回一个整数描述符,以后所有socket调用(如随后的connect和write)就用该描述符来标识这个套接字
	
	int sockfd = socket(AF_INET,	 //internetwork: UDP, TCP, etc
	 					SOCK_STREAM, //SOCK_STREAM说明是TCP类型
	 					0);			 //protocol
	
二,与TCP服务器建立连接

	    /**
	     * 设置发往的地址
	     */
	    struct sockaddr_in addrto;
	    addrto.sin_family = AF_INET;//地址类型为internetwork
	    addrto.sin_addr.s_addr = inet_addr(ip);
	    addrto.sin_port = htons(port);
	    
	    int connect(int sockfd,			 //sockfd是由socket函数返回的套接字描述符
	    const struct sockaddr* servaddr,//套接字地址
	    socklen_t addrlen);				 //套接字地址结构大小
    
三,发送信息:

使用<font color="#bd260d">**send**</font>函数向TCP连接的另一端(服务器端)发送数据:
	
	int send(SOCKET s,	   //指定接收端的套接字描述符
			  char *buf,   //指定一个存放应用程序要发送数据的缓冲区			  int len,	   //指明buf的长度
			  int flags);  //这个参数一般置为0

四,数据接收

使用<font color="#bd260d">**recv**</font>函数从TCP连接的另一端接收数据:

	int recv(SOCKET s,	//指定接收端的套接字描述符
			 char *buf, //指明一个缓冲区,该缓冲区用来存放recv接收到的数据
			 int len,	 //指明buf的长度
			 int flags); //这个参数一般设置为0

#完整代码

<https://github.com/sbxfc/CrossPlatformSocket>	

#参考

网络传输协议(Wiki):<br><http://zh.wikipedia.org/zh-cn/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE>

TCP/IP协议族(Wiki):<br><http://zh.wikipedia.org/wiki/TCP/IP%E5%8D%8F%E8%AE%AE%E6%97%8F>

TCP/IP 详解

Unix 网络编程

sockimp.pdf:<br><http://www.openss7.org/papers/strsock/sockimp.pdf>

windows/linux socket接口的异同:<br><http://blog.chinaunix.net/uid-28323465-id-3876368.html>