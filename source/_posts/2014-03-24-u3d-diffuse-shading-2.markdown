---
layout: post
title: "Unity3D - 漫反射着色（二）"
date: 2014-03-24 10:30:58 +0800
comments: true
categories: 
---

#Lambert定律

当光照射在物体表面时,无论光的入射角度如何，都会向所有方向发生反射。反射光的亮度只和入射角度有关,与观察者角度无关。光线越平行于物体表面时，反射光越弱，表面越暗。光线越垂直于表面时，反射光越强，表面越亮。这种各向同性的反射叫漫反射。
		
漫反射的光强近似地服从于Lambert定律,即漫反射光的光强仅与入射光的方向和反射点处表面法向夹角的余弦成正比:
	
	Idiffuse =Id * Kd * cosθ 

- `Idiffuse`漫反射光强  
- `Id`为点光源
- `Kd(0<Kd<1)`表示物体表面该点对漫反射光的反射属性（金属材质比木材质的反射要好）
- `θ`是入射光线的方向与物体表面该点处法线N的夹角，或称为入射角`(0≤θ≤90°)`,
入射角为零时，说明光线垂直于物体表面，漫反射光强最大;90°时光线与物体表面平行，物体接收不到任何光线。 

![diffuse](/images/2014/3/diffuse.jpg)

若L表示入射点指向光源的单位向量,N表示入射点的单位法向量。由[向量的点积](http://rungame.me/blog/2014/04/23/vector-projection/ "")定律可知**cosθ = N * L**。
	
Lambert定律就可以写为:

	Idiffuse = Id * Kd *(N * L)


#Lambert光照模型

在Unity里,我们使用Lambert光照模型近似地反映漫反射。Lambert光照模型由环境光和漫反射光两部分组成。

Lambert光照模型被写为:

	I= IaKa (环境光) + Idiffuse= IaKa + Id * Kd *(N * L)

上一部分的最后的光照模型就是Lambert光照模型:

	inline float4 LightingLambert (SurfaceOutput s, fixed3 lightDir, fixed atten)
	{
		//计算cosθ
	    float difLight = max(0, dot (s.Normal, lightDir));
	    float4 col;
	    col.rgb = s.Albedo * _LightColor0.rgb * (difLight * atten * 2);
	    col.a = s.Alpha;
	    return col;     
	}

- **dot(arg1,arg2)** 乘积	
- **max(arg1,arg2)** 取两者中的最大值，因为第一个参数是0.所以我们取得的值都是大于、等于0的。

NOTE:在Lambert光照模型里, `difLight * atten * 2` 中的*2，是为了增加光照强度。

---

# Half Lambert 光照模型

Half Lamber是大名鼎鼎的Valve公司在制作半条命时，为了使阴暗区域的光亮得到补强，而设计的光照模型。

half-Lambert 光照模型代码实现:

		inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, fixed atten)
		{
			float difLight = dot (s.Normal, lightDir);
			float hLambert = difLight * 0.5 + 0.5; 
			
			float4 col;
			col.rgb = s.Albedo * _LightColor0.rgb * (hLambert * atten * 2);
			col.a = s.Alpha;
			return col;
		}


上面的half-Lambert光照模型是在Lambert的基础上得来的。其中只是用hLambert取代了漫反射中的difLight(**即cosθ值**)。hLambert取difLight的`*0.5+0.5`后的结果。

下图中,我们可以比较原始的difLight（**即cosθ**）的值和新的hLambert的区别。原本的difLight是一条穿越(0,0)点的余弦曲线。重新计算处理之后的hLambert,其取值全都大于0。
	
![half.png](/images/2014/3/half.png)

原先的计算公式里，在区间[-1,0]部分,我们利用`max`函数将这些角度上的值转化为0,随着光照的角度变化，在[-1,1]这个区间的颜色变化显得比较生硬。而half-Lambert光照模型有一个很好的渐变的过程，每个角度对应的值都会比Lambert更高,表现出来也就更亮。

---
# 使用渐变图(RAMP)控制漫反射光强变化

![ramp_texture.png](/images/2014/3/ramp_texture.png)

使用图片编辑器（比如PS),创建一个渐变图如上图。修改Properties,添加一段声明

	_RampTexture ("Ramp Texture", 2D) = "white"{}

在CG代码里声明贴图变量:

	sampler2D _RampTexture;
	
修改光照模型:

	inline float4 LightingBasicDiffuse (SurfaceOutput s, fixed3 lightDir, fixed atten)
		{
			float difLight = dot (s.Normal, lightDir);
			float hLambert = difLight * 0.5 + 0.5;
			float3 ramp = tex2D(_RampTexture, float2(hLambert)).rgb;
			
			float4 col;
			col.rgb = s.Albedo * _LightColor0.rgb * (ramp);
			col.a = s.Alpha;
			return col;
		}

我们使用了一张269*33的贴图，其中颜色的变化只在横向x轴方向,y轴方向颜色值无变化。

我们已知hLambert是一个介于[0，1]值,假如此刻hLambert取值为0.3，那么经float2函数强制转型之后就变成了(0.3,0)的形式。

因为y方向颜色值是固定的，那么`0.3*269=80.7`,那么它将去贴图上取(80.7,0)点的颜色值。

我们没有使用顶点的UV值，而是基于漫反射即光线方向在贴图上取值，最终模型表面的颜色也是与光线方向高度关联的。

---

参考:

着色器代码:<br><https://github.com/sbxUnity/u3dLambertShader>
