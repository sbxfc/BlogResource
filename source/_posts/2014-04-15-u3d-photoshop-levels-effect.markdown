---
layout: post
title: "Unity3D - 实现PS效果的Shader"
date: 2014-04-15 19:10:23 +0800
comments: true
categories: 
---
在Photoshop里，所有编辑工具和混合模式全部是基于像素的数学运算，无论与其他任何值进行乘、除、加、减运算,  <br>

最终都会在编辑的图片上返回一个新像素。

贴图:

![photoshop_levels_effect.jpg](/images/2014/4/photoshop_levels_effect.jpg)

	Shader "customShader/PhotoshopLevels" 
	{
		Properties 
		{
			_MainTex ("Base (RGB)", 2D) = "white" {}
			
			//Add the Input Levels Values
			_inBlack ("Input Black", Range(0, 255)) = 0
			_inGamma ("Input Gamma", Range(0, 2)) = 1.61
			_inWhite ("Input White", Range(0, 255)) = 255
			
			//Add the Output Levels
			_outWhite ("Output White", Range(0, 255)) = 255
			_outBlack ("Output Black", Range(0, 255)) = 0
		}
		
		SubShader 
		{
			Tags { "RenderType"="Opaque" }
			LOD 200
			
			CGPROGRAM
			#pragma surface surf Lambert
	
			sampler2D _MainTex;
			
			//Add these variables
			//to the CGPROGRAM
			float _inBlack;
			float _inGamma;
			float _inWhite;
			float _outWhite;
			float _outBlack;
	
			struct Input 
			{
				float2 uv_MainTex;
			};
			
			//调整色阶
			float GetPixelLevel(float pixelColor)
			{
				float pixelResult;
				pixelResult = (pixelColor * 255.0);
				pixelResult = max(0, pixelResult - _inBlack);
				pixelResult = saturate(pow(pixelResult / (_inWhite - _inBlack), _inGamma));
				pixelResult = (pixelResult * (_outWhite - _outBlack) + _outBlack)/255.0;	
				return pixelResult;
			}
			
			
			void surf (Input IN, inout SurfaceOutput o) 
			{
				half4 c = tex2D (_MainTex, IN.uv_MainTex);
				
				float outRPixel  = GetPixelLevel(c.r);
				float outGPixel  = GetPixelLevel(c.g);
				float outBPixel  = GetPixelLevel(c.b);
					
				//调整RGB通道后的颜色值	
				o.Albedo = float3(outRPixel,outGPixel,outBPixel);
				o.Alpha = c.a;
			}
			ENDCG
		} 
		FallBack "Diffuse"
	}

设置一些可控的变量,我们可以在操作面板对其做出修改:

	_inBlack ("Input Black", Range(0, 255)) = 0
	_inGamma ("Input Gamma", Range(0, 2)) = 1.61
	_inWhite ("Input White", Range(0, 255)) = 255
			
	//Add the Output Levels
	_outWhite ("Output White", Range(0, 255)) = 255
	_outBlack ("Output Black", Range(0, 255)) = 0

tex2D函数取得贴图的UV值

					half4 c = tex2D (_MainTex, IN.uv_MainTex);

然后对每个像素的RGB通道的值做调整:

	float outRPixel  = GetPixelLevel(c.r);
	float outGPixel  = GetPixelLevel(c.g);
	float outBPixel  = GetPixelLevel(c.b);
	
关键是重新获取像素的函数GetPixelLevel,我们传入一个像素值，然后返回一个调整后的像素：

	float GetPixelLevel(float pixelColor)
				
其中，下面的代码将通道的值[0,1]，转化为32位的计算机二进制颜色取值区间[0,255]

	pixelResult = (pixelColor * 255.0);
	
减去_inBlack属性，让该通道像素变暗,并且确保该值不小于0

	pixelResult = max(0, pixelResult - _inBlack);

将修改后的像素值，除以一个计算后（_inWhite - _inBlack）的新_inWhite值，这样能够提升像素值，并让该像素更亮。<br>
然后，计算将计算结果进行_inGamma次幂运算，移除中间值(midpoit)。最后利用saturate函数，将其压缩到区间[0,1]

	pixelResult = saturate(pow(pixelResult / (_inWhite - _inBlack), _inGamma));

最后，再次修改像素值,让最终输出结果的最大像素，最小像素可控。然后除以 255 转化为区间[0,1]

	pixelResult = (pixelResult * (_outWhite - _outBlack) + _outBlack)/255.0;	