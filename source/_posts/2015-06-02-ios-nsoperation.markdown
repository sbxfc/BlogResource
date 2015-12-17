---
layout: post
title: "iOS - 操作队列"
date: 2015-06-02 11:27:11 +0800
comments: true
categories: 
---

操作队列(NSOperationQueue)是一个可以方便实现后台操作的工作队列,它是在GCD的基础上使用Cocoa抽象出来的一个队列模型。操作队列和GCD一样既可以在主线程运行,做一些UI刷新操作,也可以在子线程运行。

操作队列里的工作被封装在一个NSOperation对象里,NSOperation本身是抽象对象不能直接使用,需要创建它的子类或者使用系统提供的两个子类 NSBlockOperation 或 NSInvocationOperation。


#NSOperation


NSOperation 将工作单独封装成一个单元,并提供了一些线程安全的特性,比如状态(state),优先级(priority),依赖(dependencies)以及取消(cancellation)等。

1,状态(state):

状态(state)描述了一个 operation 的执行过程:

	isReady -> isExecuting -> isFinished

1）isReady：isReady属性与dependencies有关,当operation依赖的工作完成时,会获得一个isReady键值的KVO通知,然后isReady属性修改为YES,标识该工作已经准备好。

2）isExecuting：如果操作队列正在执行该operation,operation的isExecuting返回YES。

3）isFinished: 任务成功的完成了或者中途被Cancel掉,该值返回YES。NSOperationQueue只会把isFinished为YES的operation踢出队列,isFinished为NO的永远不会被移除,所以实现时一定要保证其正确性,避免死锁的情况发生。

2,取消(cancellation):

NSOperation里的工作是可以取消的,取消一个operation可以是显式的调用cancel方法,也可以是operation依赖的其他operation执行失败。

和state类似,当NSOperation被取消,是通过isCancelled键值的KVO来获得。当NSOperation的子类覆写cancel方法时,注意清理掉内部分配的资源。特别注意的是,这时isCancelled和isFinished的值都变为了YES,isExecuting为值变为NO

3，优先级(priority):

所有的operation在NSOperationQueue中未必都是一样的重要,设置queuePriority属性就可以提升和降低operation的优先级，queuePriority属性可选的值如下:

- NSOperationQueuePriorityVeryHigh
- NSOperationQueuePriorityHigh
- NSOperationQueuePriorityNormal
- NSOperationQueuePriorityLow
- NSOperationQueuePriorityVeryLow

4,依赖(Dependencies):

如果你从服务器上下载一张图片完成后进行压缩,你可能会想把下载图片作为一个operation,压缩作为另外一个。一个图片在从服务器上下载下来之前是没有办法压缩的,于是我们说压缩图片的operation依赖从服务器上下载图片的operation,后者必须先完成,前者才能开始执行。通过代码来说就是:

	[resizingOperation addDependency:networkingOperation];
	[operationQueue addOperation:networkingOperation];
	[operationQueue addOperation:resizingOperation];

除非一个操作的依赖的isFinished返回YES，不然这个操作不会开始。要记住添加到队列里的所有的operation的依赖关系,并避免循环依赖,比如A依赖B,B依赖A,这样会产生死锁。


#自定义NSOperation子类


对于非并发的工作,你只需要实现NSOperation子类里的main方法:

	@implementation YourOperation

	-(void)main
	{
		if(self.isCancelled == NO)
	    {
	        //执行相应的操作
	    }
	}
	
	@end

如果要支持并发工作，那么NSOperation子类需要至少重写这四个方法:
	
- start
- isConcurrent
- isExecuting
- isFinished

start方法是工作的入口，通常是你用来设置线程或者其他执行工作任务需要的运行环境的，注意不要调用[super start]；isConcurrent是标识这个Operation是否是并发执行的，这里曾经是个坑，如果你没有实现isConcurrent，默认是返回NO，那么你的NSOperation就不是并发执行而是串行执行的，不过在iOS5.0和OS X10.6之后，已经会默认忽略这个返回值，最终和Queue的maxConcurrentOperationCount最大并发操作值相关；isExecuting和isFinished是用来报告当前的工作执行状态情况的，注意必须是线程访问安全的。

注意你的实现要发出合适的KVO通知，因为如果你的NSOperation实现需要用到工作依赖从属特性，而你的实现里没有发出合适的“isFinished”KVO通知，依赖你的NSOperation就无法正常执行。NSOperation支持KVO的属性有：

- isCancelled
- isConcurrent
- isExecuting
- isFinished
- isReady
- dependencies
- queuePriority
- completionBlock

当然也不是说所有的KVO通知都需要自己去实现，例如通常你用不到addObserver到你工作的“isCancelled”属性，你只需要直接调用cancel方法就可以取消这个工作任务。

#NSInvocationOperation & NSBlockOperation


除此之外,对于一些不复杂的工作,可以由官方提供的NSOperation两个子类 NSInvocationOperation 和 NSBlockOperation 来实现。

NSInvocationOperation:

	NSInvocationOperation* operation = [[NSInvocationOperation alloc]
                                            initWithTarget:self
                                            selector:@selector(doSomeWork:)
                                            object:nil];

NSBlockOperation:

	NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
            // Do some work.
        }];

#NSOperationQueue

创建一个子线程队列,并将operation添加至队列里:

	NSOperationQueue* operationQueue = [[NSOperationQueue alloc] init];
	[operationQueue addOperation:operation];

或者,直接使用主线程队列:

	NSOperationQueue* operationQueue = [NSOperationQueue mainQueue];

设置并发工作的数量:

	operationQueue.maxConcurrentOperationCount = 1;

取消所有队列工作:

	[operationQueue cancelAllOperations];

#参见

- <https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperation_class/index.html>

<!--对于非并发操作,你可以通过重写 main 方法来定义自己的operation。使用main方法非常简单,你不需要管理一些状态属性（例如 isExecuting 和 isFinished）,当 main 方法返回的时候,这个 operation 就结束了。

	@implementation YourOperation
	    - (void)main
	    {
	        // 进行处理 ...
	    }
	@end

如果要支持并发工作,那么NSOperation子类需要至少重写这四个方法:

- start
- isConcurrent
- isExecuting
- isFinished

..

	@implementation YourOperation
	    - (void)start
	    {
	        self.isExecuting = YES;
	        self.isFinished = NO;
	        // 开始处理,在结束时应该调用 finished ...
	    }
		
		- (BOOL)isConcurrent {
		    return YES;
		}
		
		- (BOOL)isExecuting {
		    return self.executing;
		}

	    - (void)finished
	    {
	        self.isExecuting = NO;
	        self.isFinished = YES;
	    }
	@end-->


<!--操作队列(Operation Queue)是基于GCD实现的用于异步操作任务的队列模型。GCD本身是通过C语言实现的,而Operation Queue在GCD基础上实现了一些方便的功能,并且通过Cocoa进行抽象封装。

通常来说Operation Queue是开发者最安全的选择。

#NSOperation

Operation Queue队列处理的任务是通过NSOperation子类来封装的。你可以自己来写子类实现,也可以使用Foundation框架里提供的两个子类:<font color="#bd260d">**NSInvocationOperation**</font> 或 <font color="#bd260d">**NSBlockOperation**</font>

一,NSInvocationOperation

	NSInvocationOperation* operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(runTask:) object:nil];
	[operation start];

二,NSBlockOperation
	
	NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        [self runTask:@"task1"];
    }];
    [operation addExecutionBlock:^{
        [self runTask:@"task2"];
    }];
    [operation addExecutionBlock:^{
        [self runTask:@"task3"];
    }];
    [operation start];
    
[NSBlockOperation](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSBlockOperation_class/index.html) 可以并行执行多个 NSOperation ,但只有当所有 block 都执行完成之后,NSBlockOperation的 <font color="#bd260d">**isFinished**</font> 属性才会被标记为已完成。 

三,自定义NSOperation子类
	
第一种方式是简单地重写main函数来操作operations,当 main 方法返回的时候,这个 operation 就结束了。



	-(void)main
	{
	    NSLog(@"Do SomeThing");
	}
	


另一种方式是重写start方法。在这种情况下,你必须手动管理操作的状态。 为了让操作队列能够捕获到操作的改变,需要将状态的属性以配合 KVO 的方式进行实现。如果你不使用它们默认的 setter 来进行设置的话,你就需要在合适的时候发送合适的 KVO 消息。


	@implementation MyOperation
	    - (void)start
	    {
	        [self willChangeValueForKey:@"isExecuting"];
    		_isExecuting = YES;
    		[self didChangeValueForKey:@"isExecuting"];
    		
    		// 开始处理,在结束时调用 finished ...
    		[self runTask];	        
	    }
	    
	
	    - (void)finish
		{
		    [self willChangeValueForKey:@"isExecuting"];
		    [self willChangeValueForKey:@"isFinished"];
		    
		    _isExecuting = NO;
		    _isFinished = YES;
		    
		    [self didChangeValueForKey:@"isFinished"];
		    [self didChangeValueForKey:@"isExecuting"];
		}
	@end


参见:[MyOperation类](https://github.com/sbxIOS/OperationQueuesExample)

#Operation Queue

operation创建好以后很容易加入operation queue中

	queue = [[NSOperationQueue alloc] init];
	NSInvocationOperation* operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(runTask:) object:nil];
	[queue  addOperation:operation];
	
或者直接使用Block添加:
	
	//想queue队列添加任务
	[queue addOperationWithBlock:^{
		//向主队列添加任务 
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        	    
        }];
    }];
    
#参考

使用Operation Queue实现批量下载:<br>
<https://github.com/sbxIOS/asyncDownloadQueque>

使用Operation Queue对ScrollView里的项进行异步绘制:<br>
<https://github.com/sbxIOS/lazyViewExample>-->