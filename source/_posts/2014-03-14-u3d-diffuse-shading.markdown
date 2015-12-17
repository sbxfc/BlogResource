---
layout: post
title: "Unity3D - 漫反射着色（一）"
date: 2014-03-14 20:57:36 +0800
comments: true
categories: 
---

#编写表面着色器

一,新建一个Shader文件,取名为BasicDiffuse:

	Shader "Custom/BasicDiffuse"

二,在属性(Properties)里设置两个颜色属性**Color**和一个float类型的**Range(0,10)**属性,并设置初始值
	
	_EmissiveColor ("Emissive Color", Color) = (1,1,1,1)//发散光
	_AmbientColor ("Ambient Color", Color) = (1,1,1,1)  //环境色
	_MySliderValue ("This is a Slider", Range(0,10)) = 2.5

三，删除纹理内容(用不到纹理):

	sampler2D _MainTex;
	
	//sort方法内
	half4 c = tex2D (_MainTex, IN.uv_MainTex);
	
四，在SubShader里添加以下变量,这些变量和Properties里的变量名称要一致。<br>
这样做,可以使得CG代码里的变量和ShaderLab里Properties上设置的变量链接在一起。

	float4 _EmissiveColor;
	float4 _AmbientColor;
	float _MySliderValue;

五,修改表面函数surf: 
	
	void surf (Input IN, inout SurfaceOutput o)
	{
    	//pow(x,y) 即x的y次幂
    	float4 c =  pow((_EmissiveColor + _AmbientColor),  _MySliderValue);
        o.Albedo = c.rgb;
        o.Alpha = c.a;
        
	}
	
在表面着色器里,surf函数主要控制每个片元的颜色和纹理的设置。<br>

表面函数的传入值是一个Input结构。在这个结构里,Unity默认为我们声明了一个纹理相关的属性uv_MainTex,在未删除纹理内容前,在struct结构外,有一个声明好的默认纹理对象_MainTex。在纹理对象前面加uv_实际是获取纹理的**uv_**值。

在这里,我们没有用到纹理相关数据。只是把输出的图元颜色重新做了处理,将设置好的两个颜色变量混合,然后进行幂运算。

六,最后执行编译语句,编译这个表面着色器,并且让它接收Lambert光照模型处理

	#pragma surface surf Lambert

- `#pragma surface` 指明我们使用的是表面着色器
- `surf` 我们用到的表面函数   
- `Lambert` 用到了Lambert光照模型(即漫反射光照模型)  


编译语句格式:

	#pragma surface surfaceFunction lightModel [optionalparams]

#光照模型

在上面我们使用了Lambert光照模型,Lambert是一个内置的光照模型。在Unity里,光照模型实际上就是一个固定的格式的函数。

光照函数名被声明为`Lighting<Your Chosen Name>`这样的格式。比如Lambert光照模型,实际上它是一个名称为LightingLambert的函数。假如,我们要创建一个名为"BasicDiffuse"的光照模型,我们就要把光照函数名写为LightingBasicDiffuse。

除了函数名的写法特殊外,光照函数有几个固定的参数格式:

**1,不考虑视线方向**

	half4 LightingName (SurfaceOutput s, half3 lightDir, half atten){}

**2,考虑视线方向**

	half4 LightingName (SurfaceOutput s, half3 lightDir, half3 viewDir, half atten){}

**3,延迟渲染**

	half4 LightingName_PrePass (SurfaceOutput s, half4 light){}

**BasicDiffuse光照模型完整代码:**

	inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, fixed atten)
	{
		float difLight = max(0, dot (s.Normal, lightDir));
		float4 col;
		col.rgb = s.Albedo * _LightColor0.rgb * (difLight * atten * 2);
		col.a = s.Alpha;
		return col; 	
	}
	
属性:

1. `SurfaceOutput s` 对象是经过的surf函数处理的图元信息。 
- `fixed3 lightDir` 光线的方向  
- `fixed atten` 光的衰减系数  
- `_LightColor0.rgb` 光线的颜色 
- `max(arg1,arg2)` 取两者中的最大值，因为第一个参数是0.所以我们取得的值都是大于、等于0的。 
- `dot(arg1,arg2)` 乘积
	
使用上面创建的表面着色器,只需要重新修改编译命令即可:

	#pragma surface surf BasicDiffuse

---

内置光照模型的代码在Lighting.cginc文件中。	
	

