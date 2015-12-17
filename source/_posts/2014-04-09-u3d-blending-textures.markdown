---
layout: post
title: "Unity3D - 混合贴图的地形Shader"
date: 2014-04-09 10:20:38 +0800
comments: true
categories: 
---

贴图上每个像素都由四个通道构成，包括三个原色R、G、B 和一个alpha值A。因此，每个像素点都可以携带四个值。<br>

利用贴图的这种结构,我们将分别存放R、G、B、A四个通道值的灰图合成一张新贴图，如图所示：<br>
![](/images/2014/4/texture_data.png)

灰图上的颜色值介于[0,1]之间,越亮的部分颜色越接近白色,色值也越接近1。越暗的部分,颜色越接近黑色,色值接近于0。<br>
第一张灰图的左上角存放的是R通道(red)值,且颜色值是白色，色值接近1，在组合后形成的示意图中呈红色。其他三块区域类同。<br>
(PS.这里讲到的四张图合成一张图，并非实际将四张图混合在一起，而是示意灰图的合成变化。)
<br><br>
<!-- more -->


# 地表贴图

![](/images/2014/4/terrain_texs.png)

大多数地表都是由不同层级的贴图复合而成，如草地、泥土、小石子、石头贴图。我们将这几种不同贴图分布规律的灰图合成一张地表贴图。如下图:

![](/images/2014/4/terrain_texture.png)


计算最终颜色代码:

	float4 finalColor;
	finalColor = lerp(rTexData, gTexData, blendData.g);
	finalColor = lerp(finalColor, bTexData, blendData.b);
	finalColor = lerp(finalColor, aTexData, blendData.a);
	finalColor.a = 1.0;
	
	float4 terrainLayers = lerp(_ColorA, _ColorB, blendData.r);
	finalColor *= terrainLayers;
	finalColor = saturate(finalColor);
	
下图演示了利用插值取得地表颜色的过程:

![](/images/2014/4/lerp.png)

上面的代码中，首先以`R贴图为基础`绘制G贴图在R贴图上的分布,利用地形贴图中G通道上的G贴图分布值。(有点拗口) 在计算出的最终颜色之后,又依次与B、A贴图进行插值计算。<br>其中地形贴图中的R通道值并非记录R贴图的颜色分布,而是记录了地表基色`_ColorA`和`_ColorB`的值。同样R的值越接近1，颜色就越接近`_ColorB`。(PS.这段话是我自己的理解。)

	
实例地址:<br>
<https://github.com/rungameme/terrain_blending_textures>
