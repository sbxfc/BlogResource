---
layout: post
title: "Unity3D - 法线贴图 "
date: 2014-04-14 18:54:09 +0800
comments: true
categories: 
---
#  法线贴图（Normal mapping）

使用高分辨率模型的法线贴图表大幅度地提高 low poly 低面模型的显示效果。

![normal_texture](/images/2014/4/normal_texture.png)


	Shader "customShader/NormalMapping" 
	{
		Properties 
		{
			//Add these Properties
			_MainTint ("Diffuse Tint", Color) = (1,1,1,1)
			_NormalTex ("Normal Map", 2D) = "bump" {}
			_NormalIntensity ("Normal Map Intensity", Range(0,2)) = 1
		}
		
		SubShader 
		{
			Tags { "RenderType"="Opaque" }
			LOD 200
			
			CGPROGRAM
			#pragma surface surf Lambert
	
			//Link the property to the CG program
			sampler2D _NormalTex;
			float4 _MainTint;
			float _NormalIntensity;
	
			//Make sure you get the uvs for the texture in the Struct
			struct Input 
			{
				float2 uv_NormalTex;
			};
	
			void surf (Input IN, inout SurfaceOutput o) 
			{
				//Get teh normal Data out of the normal map textures
				//using the UnpackNormal() function.
				float3 normalMap = UnpackNormal(tex2D(_NormalTex, IN.uv_NormalTex));
				normalMap = float3(normalMap.x * _NormalIntensity, normalMap.y * _NormalIntensity, normalMap.z);
							
				//Apply the new normals to the lighting model
				o.Normal = normalMap.rgb;
				o.Albedo = _MainTint.rgb;
				o.Alpha = _MainTint.a;
			}
			ENDCG
		} 
		FallBack "Diffuse"
	}


使用`UnpackNormal()`  函数将贴图的法线信息`normalMap`取出，并设置模型表面法线值。
由于tex2D()取值区间是[0,1],而UnpackNormal()将颜色空间里的法线[0,1],转换至真正3D空间里的法线范围[-1,1]。

	normalMap = float3(normalMap.x * _NormalIntensity, normalMap.y * _NormalIntensity, normalMap.z);
	
修改发现向量的x,y值，增加贴图的亮度。

# 工作原理

假设有一个球体的多边形模型只能近似表示曲面形状。通过在模型上应用RGB位图纹理，就可以对更加细致的法线向量进行编码。位图中的每一个通道即红色、绿色、蓝色通道都对应于一个空间尺度 X、Y、Z，这些空间尺度与物体空间法线图的固定的坐标系统相关，或者与 tangenet 空间法线图场合中根据位置相对于纹理坐标平滑变化的坐标系统相关。这使得模型表面更加细致，尤其是与先进的光照技术一起使用的时候更是如此。 —— 抄的。

大致意思是，利用贴图保存一份模型表面单位向量的信息，然后在Shader计算时应用。

PS.可以设置一下 Tiling 值x=12 y=12.
