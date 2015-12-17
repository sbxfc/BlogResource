---
layout: post
title: "优化Shader"
date: 2014-06-27 14:26:01 +0800
comments: true
categories: 
---
>选择变量

首先是变量的选择,cg语言内置的有关数值的变量类型有以下几种:

	float，32位浮点数据；
	half，16为浮点数据；
	int，32位整形数据；
	fixed，12位定点数；
	
在处理数据时,根据实际情况尽可能选择内存占用更少的数据类型，提高处理速度。
</br>
例如:使用half2存储UV值,或者把颜色值(介于[0,1]之间)设置为fixed类型。
<!--more-->

>在`#pragma`声明中设置`noforwardadd`

通过设置设置参数`noforwardadd`参数,告诉unity渲染该Shader模型的像素时只接受单一方向的光，其他光线以球面调和光来计算。(球面调和光计算的是顶点，只花费很少的CPU计算时间。)

	#pragma surface surf SimpleLambert noforwardadd

>设置`exclude_path:prepass`

通过设置`exclude_path:prepass`,只接收正向渲染，通过`Player Settings`选择来设置该渲染路径，也可以在具体相机上重置该值。

	#pragma surface surf SimpleLambert exclude_path:prepass noforwardadd
	
> 用贴图的UV值去索引法线贴图

对于一个模型上同时存在表面贴图和发现贴图，我们一般做法是将两张贴图的UV值传入，并取得像素点上的UV值:

	struct Input 
	{
		half2 uv_MainTex;
		half2 uv_Normal;
	};
	
我们可以简化一下，利用贴图的UV值来索引法线贴图的UV值:

	struct Input 
	{
		half2 uv_MainTex;
	};
	
	void surf (Input IN, inout SurfaceOutput o) 
	{
		…
		o.Normal = UnpackNormal(tex2D(_NormalMap, IN.uv_MainTex));
	}


<https://github.com/rungameme/optimizedShaderInUnity>