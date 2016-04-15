---
layout: post
title: "cocos2d-JS 初探"
date: 2015-10-09 18:21:41 +0800
comments: true
categories: 
---

#锚点和坐标

Cocos引擎使用的是右手[笛卡尔坐标系](https://zh.wikipedia.org/wiki/%E7%AC%9B%E5%8D%A1%E5%84%BF%E5%9D%90%E6%A0%87%E7%B3%BB),相对于我们熟悉的标准屏幕坐标系,其坐标原点在屏幕的左下角。
	
在Cocos里,显示对象以锚点(anchor)对齐。即一个sprite如果坐标设置为【0,0】并且锚点也是【0,0】的话,它的左下角就与屏幕的左下角对齐。但是默认情况下,非Layer对象的默认节点都是【0.5,0.5】。所以,如果没注意直接将坐标设置为【0,0】时,就会导致图片上半边和下半边出现在屏幕外。
	
	var sprite  = cc.Sprite.create(res.bkg_image);
	sprite.anchorX = 0;
	sprite.anchorY = 0;	
	sprite.x = 0;
	sprite.y = 0;
	this.addChild(sprite);
	
#定位

cc.winSize表示游戏窗口大小,而cc.visibleRect表示游戏的可视区域。在有些情况下,游戏窗口大小并不等于显示区域大小。如果在屏幕上方显示一个文本或精灵推荐使用cc.visibleRect

	var visibleRect = cc.visibleRect;
	var topLeft = visibleRect.topLeft;

	
#自定义事件

自定义事件的使用和JS、AS3类似,首先是建立一个事件监听器,并且设置相应的处理函数:

	var listener = cc.EventListener.create({
		event: cc.EventListener.CUSTOM,
		eventName: "game_custom_event",
		callback: function(event){
			// 可以通过getUserData来设置需要传输的用户自定义数据
			console.log("Custom event received : " + event.getUserData().toString());
		}
	});
	cc.eventManager.addListener(listener, 1);
	
在触发事件的地方使用dispatchEvent抛出事件:

	var event = new cc.EventCustom("game_custom_event");
	event.setUserData("Hello World");
	cc.eventManager.dispatchEvent(event);
	
其中,addListener函数的第二个参数可以填一个显示对象,也可以填一个数字。如果以显示对象为参数时,当该对象被回收则监听器也被移除。如果以数值为参数,该数值的大小决定了事件的优先级,数值越大则优先级越低。显示对象为参数的事件优先级为0 (ps.若一个以非显示对象为参数的addListener函数的第二个参数填0时,在一些情况下会出现问题)

这些可以在官方文档里的[事件分发机制](http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/eventManager/zh.md)中找到:

- http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/eventManager/zh.md


#异步

有时候,在不增加新线程的情况下,可以使用分时处理的方式解决一些游戏里常见的卡顿问题。

卡顿的原因在于一帧时间内处理的事情太多,需要很长时间才能切到下一帧,所以导致画面卡顿。如果我们可以将逻辑放在多帧里处理,这样就会好很多。

ccjs里支持的异步方法:
	
	cc.log("预备!");
    var NEXT_FRAME_TIME = 17;
    cc.async.parallel([
        function (cb) {
            setTimeout(function () {
                cc.log("A说:异步大法好! 当前帧:" + cc.director.getTotalFrames());
                cb();
            }, NEXT_FRAME_TIME);
        },
        function (cb) {
            cc.log("B说:异步大法好! 当前帧:" + cc.director.getTotalFrames());
            cb();
        }
    ], function () {
        cc.log("大家都说:异步大法好!");
    });


使用JSB在iOS设备上有如下输出:

*预备!<br>
B说:异步大法好! 当前帧:22<br>
A说:异步大法好! 当前帧:23<br>
大家都说:异步大法好!*


在JS语言里有两个函数可以实现异步操作 setInterval 和 setTimeout
	
- <http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-async/zh.md>

