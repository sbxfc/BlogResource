---
layout: post
title: "Blinn-Phong 光照模型"
date: 2014-04-24 18:32:32 +0800
comments: true
categories: 
---

>#### Blinn-Phong 光照模型

Blinn-Phong 比 Phong 光照模型更加柔和、平滑和快速。它用更少的代码实现了与Phong几乎相同的效果。

与Phong不同，Blinn-Phong光照模型没有去计算反射光线向量R,而是取入射光线L和实现方向V的中间向量halfVector，通常也称之为半角向量。（半角向量蕴含着丰富的信息价值。）

<!--more-->


	float3 halfVector = normalize (lightDir + viewDir);

最后，我们简单的用顶点的法向量N去乘以新计算的半角向量，计算出一个镜面反射光的强度。

	float NdotH = max (0, dot (s.Normal, halfVector));


贴代码:

	Shader "custom/CustomBlinnPhong"
	{
		Properties 
		{
			_MainTint ("Diffuse Tint", Color) = (1,1,1,1)
			_MainTex ("Base (RGB)", 2D) = "white" {}
			_SpecularColor ("Specular Tint", Color) = (1,1,1,1)
			_SpecPower ("Specular Power", Range(0.1, 120)) = 3  
		}
		
		SubShader 
		{
			Tags { "RenderType"="Opaque" }
			LOD 200
			
			CGPROGRAM
			#pragma surface surf CustomBlinnPhong
	
			sampler2D _MainTex;
			sampler2D _SpecularMask;
			float4 _MainTint;
			float4 _SpecularColor;
			float _SpecPower;
			
					
			inline fixed4 LightingCustomBlinnPhong (SurfaceOutput s, fixed3 lightDir, half3 viewDir, fixed atten)
			{
				float3 halfVector = normalize (lightDir + viewDir);
				
				float diff = max (0, dot (s.Normal, lightDir));
				
				float NdotH = max (0, dot (s.Normal, halfVector));
				float spec = pow (NdotH, _SpecPower);
				
				float4 c;
				c.rgb = (s.Albedo * _LightColor0.rgb * diff) + (_LightColor0.rgb * _SpecularColor.rgb * spec) * (atten * 2);
				c.a = s.Alpha;
				return c;
			}
	
			struct Input 
			{
				float2 uv_MainTex;
				float2 uv_SpecularMask;
			};
	
			void surf (Input IN, inout SurfaceOutput o) 
			{
				float4 c = tex2D (_MainTex, IN.uv_MainTex) * _MainTint;
				o.Albedo = c.rgb;
				o.Alpha = c.a;
			}
			ENDCG
		} 
		FallBack "Diffuse"
	}

>#### 创建基于纹理贴图的镜面反射效果
> 在反射模型计算中，添加了纹理贴图的RGB值。

设置属性,新增加了名为`_SpecularMask`的贴图:

	_MainTint ("Diffuse Tint", Color) = (1,1,1,1)
	_MainTex ("Base (RGB)", 2D) = "white" {}
	_SpecularColor ("Specular Tint", Color) = (1,1,1,1)
	_SpecularMask ("Specular Texture", 2D) = "white" {}
	_SpecPower ("Specular Power", Range(0.1, 120)) = 3  

在CG代码块里声明属性:

	sampler2D _MainTex;
	sampler2D _SpecularMask;
	float4 _MainTint;
	float4 _SpecularColor;
	float _SpecPower;

在SubShader里添加结构体`SurfaceCustomOutput`,这个是surf函数传递给光照模型的物体表面属性:

	struct SurfaceCustomOutput 
	{
		fixed3 Albedo;
		fixed3 Normal;
		fixed3 Emission;
		fixed3 SpecularColor;
		half Specular;
		fixed Gloss;
		fixed Alpha;
	};

将自定义的结构体，添加到我们的光照模型函数里,使用的Phong光照模型:(注意 s.SpecularColor 这是我们在surf函数里通过取纹理贴图取得的颜色值，并在计算中去影响镜面反射效果。)

	inline fixed4 LightingCustomPhong (SurfaceCustomOutput s, fixed3 lightDir, half3 viewDir, fixed atten)
		{
			//Calculate diffuse and the reflection vector
			float diff = dot(s.Normal, lightDir);
			float3 reflectionVector = normalize(2.0 * s.Normal * diff - lightDir);
			
			//Calculate the Phong specular
			float spec = pow(max(0.0f,dot(reflectionVector, viewDir)), _SpecPower) * s.Specular;
			float3 finalSpec = s.SpecularColor * spec * _SpecularColor.rgb;
			
			//Create final color
			fixed4 c;
			c.rgb = (s.Albedo * _LightColor0.rgb * diff) + (_LightColor0.rgb * finalSpec);
			c.a = s.Alpha;
			return c;
		}

修改我们shader的编译命令:

	#pragma surface surf CustomPhong

既然，我们要使用贴图来修改镜面反射时顶点颜色的计算，所以我们要存储这张贴图的UV值。将贴图变量前面加上uv字符并放入Input结构体内来获取贴图的UV值。

	struct Input 
	{
		//Get uv information from the Input Struct
		float2 uv_MainTex;
		float2 uv_SpecularMask;
	};

最后，我们修改我们最关键的surf函数，在surf函数里，我们将我们的贴图信息传入光照模型并最终修改模型的镜面反射效果。

	//模型表面贴图
	float4 c = tex2D (_MainTex, IN.uv_MainTex) * _MainTint;
	//带有镜面反射纹理的贴图
	float4 specMask = tex2D(_SpecularMask, IN.uv_SpecularMask) * _SpecularColor;
			
	//Set the parameters in the Output Struct
	o.Albedo = c.rgb;
	o.Specular = specMask.r;
	o.SpecularColor = specMask.rgb;
	o.Alpha = c.a;


附贴图:
模型表面贴图:

![](/images/2014/4/specular_tint.jpg)

反射模型计算的贴图:

![](/images/2014/4/specular_normal.jpg)

最终代码:

	Shader "custom/SpecularMask" 
	{
		Properties 
		{
			//Set properties here so we can feed our shader information
			//from the inspector in the editor
			_MainTint ("Diffuse Tint", Color) = (1,1,1,1)
			_MainTex ("Base (RGB)", 2D) = "white" {}
			_SpecularColor ("Specular Tint", Color) = (1,1,1,1)
			_SpecularMask ("Specular Texture", 2D) = "white" {}
			_SpecPower ("Specular Power", Range(0.1, 120)) = 3  
		}
		
		SubShader 
		{
			Tags { "RenderType"="Opaque" }
			LOD 200
			
			CGPROGRAM
			#pragma surface surf CustomPhong
			
			//get the data from our properties block
			sampler2D _MainTex;
			sampler2D _SpecularMask;
			float4 _MainTint;
			float4 _SpecularColor;
			float _SpecPower;
			
			//Create a custom Output Struct
			struct SurfaceCustomOutput 
			{
				fixed3 Albedo;
				fixed3 Normal;
				fixed3 Emission;
				fixed3 SpecularColor;
				half Specular;
				fixed Gloss;
				fixed Alpha;
			};
					
			inline fixed4 LightingCustomPhong (SurfaceCustomOutput s, fixed3 lightDir, half3 viewDir, fixed atten)
			{
				//Calculate diffuse and the reflection vector
				float diff = dot(s.Normal, lightDir);
				float3 reflectionVector = normalize(2.0 * s.Normal * diff - lightDir);
				
				//Calculate the Phong specular
				float spec = pow(max(0.0f,dot(reflectionVector, viewDir)), _SpecPower) * s.Specular;
				float3 finalSpec = s.SpecularColor * spec * _SpecularColor.rgb;
				
				//Create final color
				fixed4 c;
				c.rgb = (s.Albedo * _LightColor0.rgb * diff) + (_LightColor0.rgb * finalSpec);
				c.a = s.Alpha;
				return c;
			}
	
			struct Input 
			{
				//Get uv information from the Input Struct
				float2 uv_MainTex;
				float2 uv_SpecularMask;
			};
	
			void surf (Input IN, inout SurfaceCustomOutput o) 
			{
				//Get the color information from the textures
				float4 c = tex2D (_MainTex, IN.uv_MainTex) * _MainTint;
				float4 specMask = tex2D(_SpecularMask, IN.uv_SpecularMask) * _SpecularColor;
				
				//Set the parameters in the Output Struct
				o.Albedo = c.rgb;
				o.Specular = specMask.r;
				o.SpecularColor = specMask.rgb;
				o.Alpha = c.a;
			}
			ENDCG
		} 
		FallBack "Diffuse"
	}

