---
layout: post
title: "iOS - 基本动画"
date: 2015-03-30 17:03:49 +0800
comments: true
categories: 
---

#基础

在移动设备上,绝大多数方便、友好的操作体验都是基于动画实现的。

iOS程序里,Core Animation用来控制和管理动画。Core Animation是一个架构层,位于高级的UIKit层和低级绘图层 Core Graphics / OpenGL ES 之间。Core Animation里的许多绘图任务是通过直接使用图形硬件来处理的。

Core Animation的核心是<font  color = '#bd260d'>**CALayer**</font>,CALayer是Core Animation生成动画的地方。在运行时,Core Animation同时维护着两个平行的layer层结构: model layer tree（模型层树）和 presentation layer tree（表现层树）。前者中的 layers 反映了我们能直接看到的 layers 的状态，而后者的 layers 则是动画正在表现的值的近似。

在一个移动动画中,如果想获取目标的实时位置,就需要得到其表现层的位置信息。<font  color = '#bd260d'>**[CALayer presentationLayer]**</font>


	CALayer *layer = [view.layer presentationLayer];
	CGRect frame = layer.frame;
	
代码部分:<br><https://github.com/sbxfc/iOSAnimation/tree/master/basicAnimation>
 
<!--more-->
    
---

#基本动画

动画本身是一些可视属性变化时,目标形态过渡的过程。CALayer上一些常见的可视属性frame、position、origin、size、opacity、transform都可以用来创建动画。

对于一个位移动画,我们可以通过起始位置<font  color = '#bd260d'>fromValue</font>和终点位置<font  color = '#bd260d'>toValue</font>来设置动画区间。或者不考虑目标起始位置,通过设置目标最终到达的位置<font  color = '#bd260d'>byValue</font>来创建动画。

键值<font  color = '#bd260d'>keyPath</font>指定用于动画的显示属性。

	CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"position.x";
    animation.fromValue = @10.0f;
    animation.toValue = @200.0f;
    animation.duration = 2.0f;
    [btn.layer addAnimation:animation forKey:@"basic"];

当一个动画被添加到layer上时,会立刻复制出一份给当前layer。所以,你可以继续使用这个动画,将其添加到其他layer身上。
	
	CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"position.x";
    animation.byValue = @500.0f;
    animation.duration = 0.5f;
    animation.fillMode = kCAFillModeForwards;
    animation.removedOnCompletion = NO;
    [yellowView.layer addAnimation:animation forKey:@"basic"];
    
    animation.beginTime = CACurrentMediaTime() + 0.5f;
    [blueView.layer addAnimation:animation forKey:@"basic"];
    
完整代码:<br><https://github.com/sbxfc/iOSAnimation/tree/master/basicMoveAnimation> 
  
---  
#关键帧动画

相比简单动画,关键帧动画可以设置多于两个指定显示属性的点,然后填充中间帧。

![](/images/2015/4/form.gif)


	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"position.x";
    animation.values = @[ @0, @10, @-10, @10, @0 ];
    animation.keyTimes = @[ @0, @(1 / 6.0), @(3 / 6.0), @(5 / 6.0), @1 ];
    animation.duration = 0.4f;
    animation.repeatCount = HUGE_VAL;
    animation.additive = YES;
    
    [form.layer addAnimation:animation forKey:@"shake"];
  

设置<font  color = '#bd260d'>additive = YES</font>属性,可以使 model layer在更新presentation layer之前设置model layer的值。简单来说,对于位移动画,运动目标在移动时实际上是移动的presentation layer。这个过程里,model layer的值不变,当动画结束,目标会回到model layer的显示状态。设置additive可以使model layer可以实时更新presentation layer值,并在任意时刻当动画被打断时,还可以保持当前状态。

---
#轨迹动画

通过设置运动轨迹,实现更为复杂的关键帧动画。

![](/images/2015/4/planets.gif)

	CGRect boundingRect = CGRectMake(-150, -150, 300, 300);
	
	CAKeyframeAnimation *orbit = [CAKeyframeAnimation animation];
	orbit.keyPath = @"position";
	orbit.path = CFAutorelease(CGPathCreateWithEllipseInRect(boundingRect, NULL));
	orbit.duration = 4;
	orbit.additive = YES;
	orbit.repeatCount = HUGE_VALF;
	orbit.calculationMode = kCAAnimationPaced;
	orbit.rotationMode = kCAAnimationRotateAuto;
	
	[satellite.layer addAnimation:orbit forKey:@"orbit"];


使用CAShapeLayer实现更为惊奇的效果:

<iframe height=498 width=510 src="http://oleb.net/media/AnimatedPathsHelloWorld.m4v" frameborder=0 allowfullscreen></iframe>

原文:<br><http://oleb.net/blog/2010/12/animating-drawing-of-cgpath-with-cashapelayer/>

CAShapeLayer:<br><http://www.cnblogs.com/YouXianMing/p/3678709.html>

代码:<br><https://github.com/sbxfc/iOSAnimation/tree/master/Animated-Paths-master>

---
#时间函数

设置时间函数,控制加速度。

	animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];


- kCAMediaTimingFunctionLinear, //线性
- kCAMediaTimingFunctionEaseIn, //加速
- kCAMediaTimingFunctionEaseOut, //减速
- kCAMediaTimingFunctionEaseInEaseOut, //先加速后减速
- kCAMediaTimingFunctionDefault.

---

参考:

Core Animation编程指南:<br><http://www.cnblogs.com/xdream86/p/3250782.html> 

\#12 动画:<br><http://objccn.io/issue-12/>