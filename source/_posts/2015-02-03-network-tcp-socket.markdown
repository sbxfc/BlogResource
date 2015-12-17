---
layout: post
title: "TCP套接字编程 网络通信(三)"
date: 2015-02-03 11:36:06 +0800
comments: true
categories: 
---

TCP传输控制协议(Transmission Control Protocol)
---
---
TCP提供客户端与服务器之间的连接(connection)。TCP客户端先与某个给定服务器建立一次连接，再跨该连接与那个服务器交换数据，然后终止这个连接。

其次，TCP还提供了可靠性（reliability）。当TCP向另一端发送数据时，它要求对端返回一个确认。如果没有收到确认，TCP就自动重传数据并等待更长时间。在数次重传失败后，TCP才放弃，如此在尝试发送数据上所花的总时间一般为4~10分钟（依赖于具体实现）。

注意,TCP并不保证数据一定会被对方端点接收，因为这是不可能做到的。如果有可能，TCP就把数据递送到对方端点，否则就（通过放弃重传并中断连接这一手段）通知用户。这么说来，TCP也不能被描述成是100%可靠的协议，它提供的时数据的可靠递送或故障的可靠通知。

<!--more-->

TCP含有用于动态估算客户端和服务器之间的往返时间（round-trip time,RTT）的算法,以便知道它等待一个确认需要花多少时间。举例来说,RTT在一个局域网大约是几毫秒，跨越一个广域网则可能是几秒钟。另外，因为RTT受网络流通各种变化因素影响，TCP还持续估算一个给定连接的RTT。

TCP通过给其中每个字节关联一个序列号对所发送的数据进行排序（sequencing）。举例来说，假设一个应用写2048字节到一个TCP套接字，导致TCP发送2个分节：第一个分节所含数据的序列号为1～1024，第二个分节所含数据的序列号为1025～2048。（分节是TCP传递给IP的数据单元。）如果这些分节非顺序到达，接收端TCP将先根据它们的序列号重新排序，再把结果数据传递给接收应用。如果接收端TCP接收到来自对端的重复数据（譬如说对端认为一个分节已丢失并因此重传，而这个分节并没有真正丢失，只是网络通信过于拥挤），它可以（根据序列号）判定数据是重复的，从而丢弃重复数据。

再次，TCP提供流量控制（flow control）。TCP总是告知对端在任何时刻它一次能够从对端接收多少字节的数据，这称为通告窗口（advertised window）。在任何时刻，该窗口指出接收缓冲区中当前可用的空间量，从而确保发送端发送的数据不会使接收缓冲区溢出。该窗口时刻动态变化：当接收到来自发送端的数据时，窗口大小就减小，但是当接收端应用从缓冲区中读取数据时，窗口大小就增大。通告窗口大小减小到0是有可能的：当TCP对应某个套接字的接收缓冲区已满，导致它必须等待应用从该缓冲区读取数据时，方能从对端再接收数据。

最后，TCP连接是全双工的（full-duplex）。这意味着在一个给定的连接上应用可以在任何时刻在进出两个方向上既发送数据又接收数据。因此，TCP必须为每个数据流方向跟踪诸如序列号和通告窗口大小等状态信息。建立一个全双工连接后，需要的话可以把它转换成一个单工连接

<!--more-->

TCP连接的建立
---
---

建立一个TCP连接时会发生下述情形。

1. 服务器必须准备好接受外来的连接。这通常通过调用socket、bind和listen这3个函数来完成，我们称之为被动打开（passive open）。

2. 客户通过调用connect发起主动打开（active open）。这导致客户TCP发送一个SYN（同步）分节，它告诉服务器客户将在（待建立的）连接中发送的数据的初始序列号。通常SYN分节不携带数据，其所在IP数据报只含有一个IP首部、一个TCP首部及可能有的TCP选项。

3. 服务器必须确认（ACK）客户的SYN，同时自己也得发送一个SYN分节，它含有服务器将在同一连接中发送的数据的初始序列号。服务器在单个分节中发送SYN和对客户SYN的ACK（确认）。

4. 客户必须确认服务器的SYN。

这种交换至少需要3个分组，因此称之为TCP的三路握手（three-way handshake）。

建立TCP连接就好比一个电话系统.socket函数等同于有电话可用，bind函数是在告诉别人你的电话号码，这样他们就可以呼叫你。listen函数是打开电话振铃，这样当有一个外来呼叫到达时，你就可以听到。connect函数要求我们知道对方的电话号码并拨打它。accept函数发生在被呼叫的人应答电话之时，由accept返回客户端标识（即客户端的IP地址和端口）。类似于让电话机的来电显示功能，唯一区别在于，只有你在接听电话之后才会显示相应信息。而来电显示可以在不接听电话的情况下显示相应信息。getaddrinfo类似于从电话薄中查询电话号码，getnameinfo则类似于有一本按照电话号码而不是用户名排列的电话薄。

TCP连接终止
---
---

TCP建立一个连接需3个分节，终止一个连接则需4个分节。

1. 某个应用进程首先调用close，我们称该端执行主动关闭（active close）。该端的TCP于是发送一个FIN分节，表示数据发送完毕。

2. 接收到这个FIN的对端执行被动关闭（passive close）。这个FIN由TCP确认。它的接收也作为一个文件结束符（end-of-file）传递给接收端应用进程（放在已排队等候该应用进程接收的任何其他数据之后），因为FIN的接收意味着接收端应用进程在相应连接上再无额外数据可接收。

3. 一段时间后，接收到这个文件结束符的应用进程将调用close关闭它的套接字。这导致它的TCP也发送一个FIN。

4. 接收这个最终FIN的原发送端TCP（即执行主动关闭的那一端）确认这个FIN。

既然每个方向都需要一个FIN和一个ACK，因此通常需要4个分节。我们使用限定词"通常"是因为：某些情形下步骤1的FIN随数据一起发送；另外，步骤2和步骤3发送的分节都出自执行被动关闭那一端，有可能被合并成一个分节。

套接字地址结构
---
---

大多数套接字函数都需要一个指向套接字地址结构的指针作为参数。每个协议都定义了它自己的套接字地址结构，这些结构均以sockaddr_开头,并对应每个协议族的唯一后缀结尾。

IPv4套接字地址结构通常也称为“网络套接字地址结构”,它以sockaddr_in命名,定义在<netinet/in.h>中。
	
	/*
	 * Internet address (a structure for historical reasons)
	 */
	struct in_addr {
		in_addr_t s_addr;
	};

	/*
	 * Socket address, internet style.
	 */
	struct sockaddr_in {
		__uint8_t	sin_len;
		sa_family_t	sin_family;
		in_port_t	sin_port;
		struct	in_addr sin_addr;
		char		sin_zero[8];
	};
	
1. 长度成员sin_len是为OSI协议添加的。
2. POSIX规范只需要这个结构中的3个成员：sin_family、sin_addr、sin_port。
3. in_addr_t数据类型必须是一个至少32位无符号数据类型，in_port_t必须是一个至少16位的无符号数据类型，而sa_family_t通常是一个8位的无符号整数,而在不支持长度字段的实现中,它则是一个16位的无符号整形。


当作为一个参数传递进任何套接字函数时,套接字地址结构总是以引用形式(也就是该结构的指针)来传递。然后以这样的指针作为参数之一的任何套接字函数必须处理来自所有支持的任何协议族的套接字结构地址。

在任何声明所传递指针的数据类型上存在一个问题。有了ANSIC后解决办法很简单；void* 是通用的指针类型。然后套接字函数是在ANSIC之前定义的，在1982年采取的办法是在<sys/socket.h>头文件中定义一个通用的套接字地址结构:

	/*
	 * [XSI] Structure used by kernel to store most addresses.
	 */
	struct sockaddr {
		__uint8_t	sa_len;		/* total length */
		sa_family_t	sa_family;	/* [XSI] address family */
		char		sa_data[14];	/* [XSI] addr value (actually larger) */
	};

于是套接字函数被定位以指向某个通用套接字地址结构的一个指针作为其参数之一,这正如bind函数的ANSIC函数原型所示:

	int bind(int,struct sockaddr *,socklen_t);
	
这就要求对这些函数的任何调用都必须要将指向特定于协议的套接字地址结构的指针进行类型强制转换,编程指向某个通用套接字地址结构的指针，例如:

	struct sockaddr_in serv;
	/*fill in ser()*/
	bind(sockfd,(struct sockaddr *)&serv,sizeof(serv));

如果我们省略了其中的类型强制转换部分"(struct sockaddr *)",并假设系统的头文件中有bind函数的一个ANSIC原型,那么C编译器就会产生这样的警告信息：“warning:passing arg2 of (bind) from incompatiable type”。（警告：把不兼容的指针类型传递给"bind"函数的第二个参数。）

TCP套接字客户端
---
---

为了执行网络I/O，一个进程必须做的第一件事就是调用socket函数，指定期望的通信协议类型(使用IPv4的TCP、使用TPv6的UDP、Unix域字节流协议等)。

	int	socket(int family, int type, int protocol);
	
其中family参数指明协议族。type参数指明套接字类型。protocol参数应设为某个协议的类型常量，或者设为0，以选择所给定协议族和套接字类型的系统默认值。

TCP是一个字节流协议,仅支持SOCK_STREAM套接字。

客户端机器与服务器建立连接，只需要创建socket、connect 建立连接即可。如果需要发送、接收数据，只要添加相应的方法。

一，建立socket客户端:
	
	int sockfd = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
    if (-1 == sockfd)
    {
        perror("创建套接字失败!");
        exit(EXIT_FAILURE);
    }
    
二,用connenct函数建立与TCP服务器的链接。

	int connect(int sockfd,const struct sockaddr * seraddr,socklen_t addrlen);//若成功返回为0 ，若出错则为 -1

sockfd是由socket函数返回的套接字描述符,第二、三个参数分别是一个指向套接字地址结构的指针和指针的字节大小。

客户端在调用connect前不必非得调用bind函数，因为需要的话,内核会确定源IP的地址，并选择一个临时端口作为源端口。

在connect函数调用时将激发TCP的三次握手，而且仅在链接成功或出错时才返回，其中出错返回可能有以下几种情况:

1. 若TCP客户没有收到SYN包的响应,则返回ETIMEDOUT错误.如调用该函数时,内核发送一个SYN,如果无响应则等待6s后再发一个,如果仍无响应,则等待24s再发一个,若总共等了75s后仍未收到响应消息则返回该错误.

2. 若客服的SYN响应是RST(表示复位),表明该服务器主机在我们指定的端口上没有进程等待.这是一种硬错误,客户收到RST包后马上返回ECONNREFUSED错误

3. 若客户发出的SYN在中间的路由器上引发了一个"目的地不可到达"的ICMP错误,则认为是一种软错误.客服主机内核保存该消息,并按第一种情形所述的时间间隔继续发送SYN.若在某个规定的时间内没有收到回应,则将ICMP错 误作为EHOSTUNREACH或ENETUNREACH错误返回给进程.以下两种情况也是有可能的:一是按照本地系统的转发表,根本没有达到远程系统的路径;而是connect调用根本不等待就返回.

产生RST的三个条件:①、目的地为某端口的SYN到达,然而该端口上没有正在监听的服务器.②、TCP想取消一个已有的连接③、TCP接收到一个根本不存在的连接上的分节

客户端机(MAC)建立基本连接的完整代码:<br>
<https://github.com/sbxNetwork/TCPSocket>
	