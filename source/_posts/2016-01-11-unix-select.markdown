---
layout: post
title: "POSIX - select"
date: 2016-01-11 11:01:41 +0800
comments: true
categories: 
---

#I/O 复用 和 select函数



在《UNIX网络编程》里描述了这样一个例子,当客户端通过TCP协议正常连接服务端之后,紧接着,客户端进入标准输入的阻塞状态。此时,服务器突然重启,因为正处于输入状态所以客户端无法通过read函数读取对端返回EOF状态,直到输入结束,客户端恢复read为止(可能已久过去了许多时间)。

这种情况不是我们预期的结果。我们想让程序有一种预先设置内核通知的能力,当内核一旦发现进程指定的一个或多个I/O条件已就绪(比如输入完成,或可以进行读取数据时),它就告知进程。这个能力称为I/O复用,是由select和poll函数支持的,还有前者较新的POSIX变种pselect。

![IO multiplexing](/images/2016/1/io_multiplexing.png)

上图显示了IO复用的一般流程,首先,在图示集流阀(select)的位置,我们设置了许多需要监听的事件,然后select进入睡眠状态,当这些事件的其中一个或多个事件条件满足时,唤醒select函数,接着来执行相应的操作。

关于IO复用有一个很形象比喻:

[一个酒吧服务员（一个线程），前面趴了一群醉汉，突然一个吼一声“倒酒”（事件），你小跑过去给他倒一杯，然后随他去吧，突然又一个要倒酒，你又过去倒上，就这样一个服务员服务好多人，有时没人喝酒，服务员处于空闲状态，可以干点别的玩玩手机。至于epoll与select，poll的区别在于后两者的场景中醉汉不说话，你要挨个问要不要酒，没时间玩手机了。io多路复用大概就是指这几个醉汉共用一个服务员。](http://www.zhihu.com/question/32163005/answer/55687802)

#select函数

select函数允许程序指示内核等待多个事件的任何一个发生,并且在一个或多个事件发生时,或是经历一段指定的时间后才唤醒它。

	#include <sys/select.h>
	#include <sys/time.h>
	
	struct timeval{
		long tv_sec;/*秒*/
		long tv_usec;/*毫秒*/
	};
	
	int select(int maxfd, fd_set* readfds, fd_set* writefds, fd_set* errorfds, struct timeval* timeout);

一),首先从最后一个参数说起。timeout告知内核等待指定描述符中的任何一个就绪所花费时间。该参数会有以下三种情况:

1. 永远等待下去 : 仅在有一个描述符准备好时I/O才返回。为此,我们把该参数设置为空指针(NULL)。
2. 等待一段固定时间 : 在有一个描述符准备好I/O时返回,但等待时间不超过由timeval结构中指定的时间。
3. 根本不等待 : 检查描述符后立刻返回,这称为轮询(polling)。为此,该参数必须指向一个timeval结构,而且其中的定时器值必须为0。

二),中间的三个参数readfds,writefds,errorfds用于指定我们让内核试读、写和异常条件的描述符集合。目前支持的异常条件只有两个:

1. 某个套接字的**带外数据**的到达。
2. 某个已置为分组模式的伪终端存在可从其他端读取的控制状态信息。

如果该三个参数都为NULL,那么我们就有了一个比Unix的sleep更精确的定时器(sleep以秒为最小单位)。

三),maxfd 参数指定待测试的描述符个数,它的值是待测试的最大描述符加1,描述符0,1,2...maxfd-1将都被测试。
	
#示例代码

<https://github.com/sbxfc/POSIX-select>

#运行
	
1,客户端

	//Mac终端 
	$ gcc client_select.c -o client
	$ ./client
	$ hi //客户端输入
	$ input is readable //输出~~
	$ socket is readable //输出~~
	$ hi //回射服务器返回
	
2,服务端(回射服务器)
	
	$ gcc server_select.c
	$ ./a.out 
	
#参见
---
- <UNIX网络编程>
- <https://zh.wikipedia.org/zh/%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8>
- <https://zh.wikipedia.org/wiki/Select_(Unix)>