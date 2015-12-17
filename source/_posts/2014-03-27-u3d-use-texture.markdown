---
layout: post
title: "Unity3D - 使用贴图"
date: 2014-03-27 16:58:27 +0800
comments: true
categories: 
---

# 使用贴图
修改UV值来移动贴图,实现瀑布、河流、熔岩等效果。

设置Propertis

	Properties 
	{
		_MainTint("Diffuse Tint",Color)= (1,1,1,1)
		_MainTex ("Ramp Texture", 2D) = "white"{}
		_ScrollXSpeed("X Scroll Speed",Range(0,10)) = 2
		_ScrollYSpeed("Y Scroll Speed",Range(0,10)) = 2
	}

`_MainTex` 在这里是我们水流的贴图。  </br>`_ScrollXSpeed`和`_ScrollYSpeed`分别代表x、y轴的水流速度。

同样在CG代码里声明这些属性

	fixed4 _MainTint;
	fixed _ScrollXSpeed;
	fixed _ScrollYSpeed;
	sampler2D _MainTex;
	
修改surf函数

	struct Input 
	{
		float2 uv_MainTex;
	};

	void surf (Input IN, inout SurfaceOutput o) 
	{

			fixed2 scrolledUV = IN.uv_MainTex;

			fixed xScrollValue = _ScrollXSpeed * _Time;
			fixed yScrollValue = _ScrollYSpeed * _Time;

			scrolledUV += fixed2(xScrollValue,yScrollValue);

			half4 c = tex2D (_MainTex, scrolledUV);
			o.Albedo = c.rgb * _MainTint;
			o.Alpha = c.a;

	}

`uv_MainTex` 在一个贴图变量上加上uv两个字母，就代表取它的uv值。  </br>
`_Time` 是一个内部变量,它与游戏运行时间有关,我们利用它来改变我们的x、y轴方向上的uv值。

随着时间的推移，源UVs会一点点偏移。然后利用tex2D函数转化为贴图的新UVs。通过这种方式，我们可以得到模型表面的移动效果。

---
# sprite sheets 动画

	Shader "/customShader/animatingSpriteSheets" 
	{
		Properties 
		{
			_MainTex ("Ramp Texture", 2D) = "white"{}
			_TexWidth("Sheet Width",float) = 0.0
			_CellAmount("Cell Amount",float) = 0.0
			_Speed("Speed",Range(0.01,32)) = 12
		}
		
		SubShader 
		{
			Tags { "RenderType"="Opaque" }
			LOD 200
			
			CGPROGRAM
			#pragma surface surf Lambert
	
			sampler2D _MainTex;	//贴图	
			fixed _TexWidth;
			fixed _CellAmount;
			fixed _Speed;
			
			struct Input 
			{
				float2 uv_MainTex;
			};
	
			void surf (Input IN, inout SurfaceOutput o) 
			{
				float2 spriteUV = IN.uv_MainTex;
	
				float cellPixelWidth = _TexWidth / _CellAmount;
				float cellUVPercentage = cellPixelWidth / _TexWidth;
				
				float timeVal = fmod(_Time.y * _Speed,_CellAmount);
				timeVal = ceil(timeVal);
				
				float xValue = spriteUV.x;
				xValue += cellUVPercentage * timeVal * _CellAmount;
				xValue *= cellUVPercentage;
	
				spriteUV = float2(xValue,spriteUV.y);
	
				half4 c = tex2D (_MainTex, spriteUV);
				o.Albedo = c.rgb;
				o.Alpha = c.a;
	
			}
			
			ENDCG
		} 
		
		FallBack "Diffuse"
	}

![spritesheet1.png](/images/2014/4/spritesheet1.png)

如图，由14个不同动作的卡通人物卡片(sheet)拼成了一个图片(sprite)。  </br>我们知道，动画实际上就是一个基于时间轴的不断变换影片的影像。类似于我们熟知的翻书映画，我们在每页书纸上画上一个固定形象，随着页码的变化，这些图片基于一个有序的行为不断的改变。当我们用拇指快速翻动书页时，这些图像就会以不停变换的的形式展示出来，这就是动画。

其中,`_TexWidth`是贴图(sprite)的宽度。`_CellAmount`是片(sheet)的数量。`_Speed`代表翻阅速度。

我们将UV值存储在贴图spriteUV中,变量cellPixelWith是计算出的贴图像素宽度。cellUVPercentage是每个sheet所所占的比例。

	fmod(x,y)	//返回x/y的余数
	
利用CGFX的内置函数fmod()，让卡片(sheet)以0、1、2、3、4顺序的形式运动起来，直到最后一个卡片，依次反复:	
	

	float timeVal = fmod(_Time.y * _Speed,_CellAmount);
	timeVal = ceil(timeVal);//向上取整
	
我们设置y的值为_CellAmount，所以fmod会返回一个以sheet长度为最大值不断重复的变量。

剩余部分类似于上一部分的水流，只不过这次移动的距离是以每个sheet为宽度的值。这样，我们就会看到图片一个接一个的变化，达到动画的效果。

	xValue += cellUVPercentage * timeVal * _CellAmount;
	xValue *= cellUVPercentage;
	
timeVal是介于0-14之间的数值，为了保证每次移动只增加一个sheet，我们再乘以 `cellUVPercentage`。

---
# 给Shader设值
通过CPU来控制时间，驱动动画。

	void FixedUpdate(){
		timeValue = Mathf.Ceil(Time.time%14);
		transform.renderer.material.SetFloat(_TimeValue,timeValue);
	}