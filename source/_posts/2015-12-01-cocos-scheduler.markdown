---
layout: post
title: "cocos2d - scheduler(调度器)"
date: 2015-12-01 12:12:18 +0800
comments: true
categories: 
---

在cocos2d里,场景的每帧刷新是由一个与屏幕帧频相等的定时器来驱动绘制的,在iOS环境里,这个定时器是CADisplayLink

每当屏幕上一帧刷新结束时,主线程里维持程序生命周期的NSRunLoop就会向CADisplayLink发送通知。接着,CADisplayLink会触发一个名为doCaller的回调函数,在该回调函数里cocos2d引擎进行了一系列的帧刷新操作。

在这个过程里,除了执行基本的场景刷新以外,cocos2d还定义了一个名为scheduler(调度器)的定时器,让开发者可以参与管理与每帧刷新息息相关的操作

	//在Cocos引擎里,当Director初始化时会启动该CADisplayLink定时器
	//doCaller函数会执行每帧的刷新操作
	-(void) startMainLoop
	{
	    // Director::setAnimationInterval() is called, we should invalidate it first
	    [self stopMainLoop];
	    
	    displayLink = [NSClassFromString(@"CADisplayLink") 
	    				displayLinkWithTarget:self selector:@selector(doCaller:)];
	    [displayLink setFrameInterval: self.interval];
	    [displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];
	}

<font color='#bd260d'>**scheduler的回调函数优先于场景刷新函数被执行,这意味着,如果我们在scheduler回调函数里做一些UI刷新操作,这些操作会在下一帧被及时更新。**</font>
	
	_scheduler->update(_deltaTime);	  //调用scheduler
	...
    _runningScene->render(_renderer);//渲染场景

#基本用法

任何Node对象以Node的子类都可以直接调用scheduleUpdate方法,然后通过重载update函数来执行回调操作:
	
	this->scheduleUpdate();
	
	//*.h
	void update(float dt) override;
	//*.cpp
	void HelloWorld::update(float delta){
    	//Do something!
	}
	
#自定义scheduler

除基础用法以外,还可以通过指定执行频率和回调函数来创建自定义调度器:

	this->schedule(schedule_selector(HelloWorld::updateCustom), 1.0f, kRepeatForever, 0);
	
	//*.h
	void updateCustom(float dt);
	//*.cpp
	void HelloWorld::updateCustom(float dt){
    	//Do something!
	}
	
自定义的scheduler在回调时使用一个Timer来计时,所以会花费更多的内存和计算时间,而且由于其内部实现机制,其间隔时间至少大于0.1秒。这种scheduler更适合作为定时器来使用。
	
#优先级

我们还可以为调度器指定优先级，来处理回调方法执行的先后顺序逻辑。

	void Node::scheduleUpdate(){
	    scheduleUpdateWithPriority(0);
	}
	
	void Node::scheduleUpdateWithPriority(int priority){
	    _scheduler->scheduleUpdate(this, priority, !_running);
	}
	
#移除

Scheduler会随Node对象移除而停止,当然你也可以显示地执行移除函数

	node->unschedulerUpdate();
	node->unschedule(SEL_SCHEDULE selector, float delay);

#参考

- <http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/native/v3/scheduler/zh.md>