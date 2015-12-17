---
layout: post
title: "iOS - Notification & Delegation"
date: 2014-02-14 10:07:43 +0800
comments: true
categories: 
---

Notification 和 Delegation 是两种间接通信模式,二者都是MVC模式中的重要通信机制。


#Notification

1）KVO : KVO是通知(Notification)的一种,主要用于MVC中Controller层和Model层之间的通信。我们知道,在MVC中Controller层可以直接访问Model里的数据,但是Model却不允许直接访问Controller。

当Model层发生改变时,需要通知Controller去处理一些界面操作,这时就可以使用KVO。KVO是key-value observing的缩写,即当某个属性发生变化时通知它的观察者对象（observer）。

	//设置观察标记
	@property (nonatomic, strong) id ageObserveToken;
	
	//设置观察者                                                  
	self.ageObserveToken = [KeyValueObserver  observeObject:person
                             keyPath:@"age"
                              target:self
                            selector:@selector(ageDidChange:)
                             options:NSKeyValueObservingOptionNew];
    
age是对象person的一个属性,当age被重新赋值时(NSKeyValueObservingOptionNew),函数ageDidChange就会触发。

如果不再需要监听某个属性改变:

	[self removeObserver:person forKeyPath:@"age"];

2）NSNotificationCenter 消息中心。除了KVO以外,还可以通过消息中心进行消息散发,使用方法很简单: 

在Controller上监听一个消息:

	/* 
	 * 注册一个名为loginComplete的消息
	*/
	[[NSNotificationCenter defaultCenter]  addObserver:self selector:@selector(showMenu:) name:@"loginComplete" object:nil];


在相应的条件处罚时手动发出一条消息:

	[[NSNotificationCenter defaultCenter] postNotificationName:@"loginComplete" object:nil userInfo:[NSDictionary dictionaryWithObject:@"paul" forKey:@"username"]];
	
	
可以取消一条订阅信息:
	
	[[NSNotificationCenter defaultCenter] removeObserver:self];  


#Notification && Block

	NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
	[[NSNotificationCenter defaultCenter] 
     addObserverForName:@"loginComplete" 
     object:nil
     queue:mainQueue
     usingBlock:^(NSNotification *notification)
     {
          NSLog(@"Notification received!");
          NSDictionary *userInfo = notification.userInfo;
     }];
     
#Delegation

除了Model层和Controller层通信模式以外,在Controller层和View层之间也有类似的通信方式。

在Controller 和 View层,Controller可以直接访问View,而View层如果想操作Controller,需要在View和Controller之间建立一种间接的通信方式 - Delegation(代理)。

Delegation描述了一个对象需要别人要帮它做的事情。View是那个需要被辅助的对象,而Controller为其实现相应的帮助。在这里,对象view在绘制时需要一些数据来完成,这时,他向Controller索取帮助,因为view不能直接访问Controller,所以为要为其设置一个delegate。

首先声明一个委托协议 MyViewDelegate,并在该协议里声明需要帮助函数getSourceData:

	//view.h
	@protocol MyViewDelegate
    - (NSArray*) getSourceData;  
	@end 

然后声明一个delegate对象(注意,这里delegate被设置成弱引用):
	
	//view.h
	@property (nonatomic, weak) id <MyViewDelegate> delegate; 

当view进行绘制时,调用相应的函数发出通知给那些代理对象:
	
	//view.m
	//调用委托方法
	- (void) action{
    	[self.delegate getSourceData];
	}

Controller 通过实现协议来完成相应的委托:

	//controller.h
	@interface MyController:UIViewController <MyViewDelegate> { 
		
	}
	
	//controller.m
	view.delegate = self; 
	
	- (NSArray*) getSourceData{
    	//...返回数据
	}	
		