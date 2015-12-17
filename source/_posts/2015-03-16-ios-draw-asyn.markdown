---
layout: post
title: "iOS - 异步绘制"
date: 2015-03-16 20:11:48 +0800
comments: true
categories: 
---

设置CALayer的属性<font color="#bd260d">**drawsAsynchronously**</font>为YES时,<font color="#bd260d">**-drawRect:/-drawInContext:**</font>里的绘图指令会被延迟到后台线程里异步执行。

	self.layer.drawsAsynchronously = YES;


#参考

CALayer:<br>
<https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CALayer_class/index.html#//apple_ref/occ/instp/CALayer/drawsAsynchronously>
