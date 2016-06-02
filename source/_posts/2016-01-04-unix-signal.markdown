---
layout: post
title: "signal"
date: 2016-01-04 14:24:14 +0800
comments: true
categories: 
---

在UNIX系统里,每个进程都有一个独立的工作环境并且专注于做自己的事。然而有些时候,比如发现一个进程在砸墙(发生硬件错误),这时候,内核就会通过信号来让进程意识到这种危险情况。

#常见的信号

- SIGINT   交互式操作产生的信号（如CTRL - C）。

- SIGQUIT  当键盘按下CTRL+\从shell中发出信号，信号被传递给shell中前台运行的进程，对应该信号的默认操作是退出 (QUIT) 该进程。

- SIGTSTP  当键盘按下CTRL+Z从shell中发出信号，信号被传递给shell中前台运行的进程，对应该信号的默认操作是暂停 (STOP) 该进程。

- SIGCONT  用于通知暂停的进程继续。

- SIGALRM  起到定时器的作用，通常是程序在一定的时间之后才生成该信号。

- SIGFPE	浮点错误（0作为除数产生的错误，非法的操作）。 

#信号产生

信号的产生有两种方式,一种是由内核(kernel)差生的,比如出现硬件错误(比如出现分母为0的除法运算)。还有一种情况是其他进程产生的,发送给内核,再由内核传递给目标进程。

1,通过 raise 发送信号给当前进程,该函数
	
	/**
	 * 成功返回0,否则返回 -1
	 * @param signo 信号类型
	 */
	if (raise(SIGINT) != 0) {
        fputs("Error raising the signal.\n", stderr);
        return EXIT_FAILURE;
    }

	
2,在权限允许的情况下通过 kill 发送信号给指定进程。

	#include <sys/types.h> 
	#include <signal.h> 
	
	/**
	 * @param pid 进程ID,
	 * 当pid > 0时,信号发送至ID为pid的进程,
	 * 当pid = 0时,信号发送至同一进程组的进程,
	 * 当pid < 0 && pid != -1时,信号发送至进程组ID为(-1*pid)的所有进程
	 * 当pid = -1时,信号发送至除自身进程以外的所有ID大于1的进程
	 * 
	 * @param signo 信号类型
	 * 当 signo = 0 时,实际不发送任何信号,但照常进行错误检查,
	 * 因此,可用于检查目标进程是否存在,以及当前进程是否具有向目标发送信号的权限
	 * (root权限的进程可以向任何进程发送信号，非root权限的进程
	 * 只能向属于同一个session或者同一个用户的进程发送信号)
	 */
	int kill(pid_t pid,int signo)
	
3,sigqueue(Linux) 相比于kill,sigqueue传递了更多的附加信息,但只能向一个进程发送信号。成功返回0,否则返回 -1。

在调用sigqueue时,sigval_t指定的信息会拷贝到3参数信号处理函数（3参数信号处理函数指的是信号处理函数由sigaction安装,并设定了sa_sigaction
指针）的siginfo_t结构中，这样信号处理函数就可以处理这些信息了。
sigqueue系统调用支持发送带参数信号,所以比kill()系统调用的功能要灵活和强大得多。(Note:sigqueue是Linux系统下信号发送函数,我的Mac机没有所以只摘抄了一些介绍。)


 
	#include <sys/types.h> 
	#include <signal.h> 
	
	typedef union sigval {
 		int  sival_int;
 		void *sival_ptr;
 	}sigval_t;
 	
	/**
	 * @param pid 进程ID
	 * @param signo 信号值 sigqueue 不能发送信号给一个进程组,如果 signo = 0,将会执行错误检,
	 * 但实际上不发送任何信号，0值信号可用于检查pid的有效性以及当前进程是否有权限向目标进程发送信号
	 */
	int sigqueue(pid_t pid, int signo, const union sigval val) 

4,alarm 在指定的时间seconds秒后,将向进程本身发送SIGALRM信号，又称为闹钟时间。进程调用alarm后,任何以前的alarm()调用都将无效。如果参数seconds为零,那么进程内将不再包含任何闹钟时间。返回值,如果调用alarm()前,进程中已经设置了闹钟时间，则返回上一个闹钟时间的剩余时间,否则返回0。

	#include <unistd.h> 
	unsigned int alarm(unsigned int seconds); 

5,setitimer setitimer比alarm功能强大,setitimer调用成功返回0,否则返回-1。支持3种类型的定时器:

- ITIMER_REAL:设定绝对时间;经过指定的时间后，内核将发送SIGALRM信号给本进程;
- ITIMER_VIRTUAL 设定程序执行时间;经过指定的时间后,内核将发送SIGVTALRM信号给本进程；
- ITIMER_PROF 设定进程执行以及内核因本进程而消耗的时间和,经过指定的时间后,内核将发送ITIMER_VIRTUAL信号给本进程;

<!--more-->

	#include <sys/time.h>
	/**
	* @param which 定时器类型
	* @param value 
	*/
	int setitimer(int which, const struct itimerval *value, struct itimerval *ovalue));
	
6,abort 向进程发送SIGABRT信号,默认情况下进程会异常退出,当然可定义自己的信号处理函数。即使SIGABRT被进程设置为阻塞信号,调用abort()后，SIGABRT仍然能被进程接收。该函数无返回值。
 
	#include <stdlib.h>
	void abort(void);

#安装信号(信号监听)

如果进程想要处理某一信号,就需要在进程里安装该信号。

1,signal

	#include <signal.h> 
	
	/**
	* 第一个参数信号值。
	* 第二个参数是信号值的处理。第二个参数如果设为 SIG_IGN 表示可忽略该信号,
	* 如果设为 SIG_DFL 表示采用系统的默认处理方式,也可以指定一个处理函数。
	*/
	void (*signal(int, void (*)(int)))(int);

2,sigaction

	#include <signal.h> 
	
	/**
	 * @param union 联合数据结构中的两个元素_sa_handler以及*_sa_sigaction指定信号关联函数,
	 * 即用户指定的信号处理函数。除了可以是用户自定义的处理函数外,还可以为SIG_DFL(采用缺省的处理方式),
	 * 也可以为SIG_IGN（忽略信号）
	 * @param sa_restorer,已过时,POSIX不支持它,不应再被使用。
	 */
	struct sigaction {
		union{
			__sighandler_t _sa_handler;
			void (*_sa_sigaction)(int,struct siginfo *, void *)；
		}_u
		sigset_t sa_mask；
		unsigned long sa_flags； 
		void (*sa_restorer)(void)；
    }
                  
	/**
	* @param signum 信号值,可以为除SIGKILL及SIGSTOP外的任何一个特定有效的信号。
	* @param act 指向结构sigaction的一个实例的指针,在结构sigaction的实例中,
	* 指定了对特定信号的处理，可以为空，进程会以缺省方式对信号处理。
	* @param oldact oldact指向的对象用来保存原来对相应信号的处理,可指定oldact为NULL。
	* 如果把第二、第三个参数都设为NULL,那么该函数可用于检查信号的有效性。
	*/
	int sigaction(int signum,const struct sigaction *act,struct sigaction *oldact));

#阻塞信号/恢复信号

有时候我们不希望信号立刻执行,同时也不想信号被忽略掉(SIG_IGN)。这时,我们可以设置阻塞信号,并在信号恢复后接收阻塞的信号。

如果信号在被阻塞期间产生了一次或多次,那么该信号被解除阻塞之后通常只提交一次,也就是说Unix信号默认是不排队的。
	
	#include <signal.h>
	
	/**
	 * @func 1
	 * @param how 选择阻塞方式,how有以下三种值:
	 * SIG_BLOCK 添加set阻塞信号集合,第二个参数为本次设置之前的阻塞信号集合
	 * SIG_UNBLOCK 如果当前进程阻塞集中包含set阻塞信号集合,则解除该阻塞信号集合
	 * SIG_SETMASK 设置当前阻塞集为set指定的集合,第二个参数NULL
	 */
	int sigprocmask(int how,  const  sigset_t *set, sigset_t *oldset))；
	
	/**
	 * 获取当前已递送到进程,却被阻塞的所有信号,
     * 从set指向的信号集中获取返回结果。
	 */
	int sigpending(sigset_t *set));
	
	/**
	 * @func 2
	 * 获取当前已递送到进程,却被阻塞的所有信号,
     * 从set指向的信号集中获取返回结果。
	 */
	int sigsuspend(const sigset_t *mask));
	
	/**
	 * @func 3
	 * 用于在接收到某个信号之前, 临时用mask替换进程的信号掩码, 并暂停进程执行，直到收到信号为止。
	 * 也就是说,进程执行到sigsuspend时,sigsuspend并不会立刻返回,进程处于TASK_INTERRUPTIBLE状态
	 * 并立刻放弃CPU,等待UNBLOCK（mask之外的）信号的唤醒。进程在接收到UNBLOCK（mask之外）信号后,
	 * 调用处理函数，然后把现在的信号集还原为原来的,sigsuspend返回,进程恢复执行。
	 * 该系统调用始终返回-1，并将errno设置为EINTR。
	 */
	sigsuspend(const sigset_t *mask));

#完整示例

<https://github.com/sbxfc/unix/tree/master/signal>	
#参见

- [Unix信号](https://zh.wikipedia.org/wiki/Unix%E4%BF%A1%E5%8F%B7)
- <http://en.cppreference.com/w/c/program/raise>
- <http://www.ibm.com/developerworks/cn/linux/l-ipc/part2/index1.html>
- <https://zh.wikipedia.org/wiki/Kill_(%E5%91%BD%E4%BB%A4)>
- <kernal/signal.c>
- <http://pubs.opengroup.org/onlinepubs/009695399/functions/sigqueue.html>
- <http://www.ibm.com/developerworks/cn/linux/l-ipc/part2/index1.html>
- <http://www.ibm.com/developerworks/cn/linux/l-ipc/part2/index2.html>
