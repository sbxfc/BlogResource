---
layout: post
title: "计算反射光线"
date: 2014-04-18 17:50:18 +0800
comments: true
categories: 
---
给定入射光线向量I和平面法向量N，求反射向量R。

![](/images/2014/4/dot10.jpg)

将R平移，与N的延长线相交。<br>
由入射角等于反射角，且I,R的长度相等，所以ION是等腰三角形，故有:

	ON = 2S;
	
所以

	R = I + 2S;
	
而S是-I在N上的投影，所以:

![](/images/2014/4/dot11.gif)

由于N是单位向量,简化则有：

![](/images/2014/4/dot12.gif)

所以

![](/images/2014/4/dot13.gif)
