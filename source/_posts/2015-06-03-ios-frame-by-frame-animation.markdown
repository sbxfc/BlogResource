---
layout: post
title: "iOS - 帧时长可控的逐帧动画"
date: 2015-06-03 10:12:57 +0800
comments: true
categories: 
---

UIImageView组件可以通过设置 animationImages、anmiamtionDurations 等参数来实现逐帧动画。 但是,如果我们想单独改变某一帧的播放时间,通过UIImageView现有的API却不能实现。那么,我们就会考虑自己定义一个这样的逐帧动画。

当你选择诸如 <font color="#bd260d">**CADisplayLink**</font> 、 <font color="#bd260d">**NSTimer**</font>等时间驱动来绘制这样的的动画时,很可能会遇到一个问题。当这样的动画在同一个界面被大量使用时,会带来不小的CPU消耗、甚至是帧阻塞问题。通常情况下 UIView动画 或者 CAAnimation 驱动的动画任务几乎不受系统影响。所以,我们可以选择用 UIView 或 CAAnimation 等来封装我们的自定义动画。

实际上,UIImageView动画本身就是一个封装了传统动画API的image view。其实现方式是通过修改image view在layer上的contents属性来完成动画播放的:

<!-- more -->

	<CAKeyframeAnimation:0x8e5b020; 
	    removedOnCompletion = 0; 
	    delegate = <_UIImageViewExtendedStorage: 0x8e49230>; 
	    duration = 2.5; 
	    repeatCount = 2.14748e+09; 
	    calculationMode = discrete; 
	    values = (
	        "<CGImage 0x8d6ce80>",
	        "<CGImage 0x8d6d2d0>",
	        "<CGImage 0x8d5cd30>"
	    ); 
	    keyPath = contents
	>
    


所以,我们可以通过修改contents来实现自定义动画。创建一个CAKeyframeAnimation动画,设置 keyTimes 来控制每帧时长。keyTimes里的元素表示每帧图片出现在动画总时长duration里的相对位置,其取值为区间[0,1]上的比例值。且 keyTimes 里元素个数要比values值多1个,使循环播放时首尾帧动画可以衔接起来。

	CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"contents"];
	animation.calculationMode = kCAAnimationDiscrete;
	animation.keyTimes = keyTimes;
	animation.values = images;
	animation.duration = 7.0f;
	animation.repeatCount = HUGE_VAL;
	[imageView.layer addAnimation:animation forKey:nil];   

    
#代码

<https://github.com/sbxfc/FrameByFrameAnimation>